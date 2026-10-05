---
title: "Eval Harnesses for Systems Engineers"
date: 2026-10-05 23:30:00 +02:00
description: "Why eval harnesses feel alien to infrastructure engineers moving into AI, and how to map them onto the concepts you already use every day: SLOs, canary analysis and distributed traces."
tags:
- ai
- llm
- distributed-systems
- testing
showComments: true
---

If like me you've spent a career in infrastructure or distributed systems, you probably have strong instincts about correctness. A service is healthy or it isn't. A test passes or it fails. A dashboard shows p99 latency crossing a threshold and somebody gets paged. Most of those instincts transfer well when you move into AI engineering: you already understand retries, timeouts, idempotency, tracing and capacity planning better than most people building LLM applications. The one place your instincts actively mislead you is evaluation.

That's exactly what happened to me the first time I built an AI system. My first thought was: "OK, I know how to build systems. Let's add some telemetry here and there, throw in some tests and create a couple of dashboards, including one for token consumption to keep costs under control." I looked at those dashboards and was genuinely happy. Everything was green: latency was low, token consumption sat within the ranges I expected, every check passed. But an applied AI system like a remediation agent has more than one way of being healthy, and I was only measuring the one I already knew. My dashboards told me the agent was running. None of them told me whether it was right: whether it picked the correct action, whether the cluster actually ended up in a good state, or whether it would make the same decision if I replayed the same incident tomorrow. I had built solid observability for availability and none at all for behavior.

In this post we'll try to cover that gap: what an eval harness is, why it doesn't behave like a unit test suite or a Grafana dashboard, and how to build one for the two systems most of us are actually shipping, a RAG pipeline and an agent. It's not meant to be a detailed guide to eval harnesses; it's meant to be a starting point for senior systems engineers moving into AI engineering.

## The failure mode a dashboard can't see

The failure mode I walked straight into looks like this. The retriever pulls the runbook for the wrong service, the model writes a confident, well-formatted answer from it, the API returns a 200 in 900 ms, and nothing on your dashboard changes color. Your observability stack was designed to detect systems that stop working. But LLM systems mostly keep working, just wrongly.

That's the core reason eval harnesses exist. They're the only layer that checks whether the *content* of the response was correct, not whether the response arrived or the latency of that response is adequate.

Your second instinct, writing unit tests, breaks for a different reason. A unit test asserts that `f(x) == y`. With an LLM in the loop you hit two problems at once.

- Nondeterminism: the same input produces different outputs across runs because of sampling, and across weeks because model versions change underneath you.
- Open-endedness: there are many correct answers to "how do I drain this node safely?", and an exact string match will reject most of them while happily accepting a wrong answer that happens to match the format.

**So correctness stops being a boolean and becomes a rate**. Once you accept that, the rest of the design follows, and it looks a lot more like things you already run in production than like `pytest`.

## What an eval harness actually is

That's probably the question you are currently asking yourself. Anthropic's engineering post [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) has the cleanest definition I've found: the eval harness is the infrastructure that runs evaluations end to end. It feeds the system its instructions and tools, runs tasks concurrently, records every step, grades the results and aggregates them.

Remove all the AI vocabulary, and what you have left is a test runner with a statistics layer on top; something we, as systems engineers, can understand.

Here's the vocabulary mapped onto concepts we already own:

| Eval harness concept | What you already know |
| --- | --- |
| Dataset (golden set of cases) | Synthetic probes, replayed production traffic |
| System under test | The service behind the load balancer |
| Trial | One request against that service |
| Transcript | A distributed trace |
| Grader | A health check or an assertion with tolerance |
| Pass rate across the suite | An SLI measured against an SLO |
| Regression gate in CI | Automated canary analysis |

The last row is the one that matters most. If you've used something like [Kayenta](https://github.com/spinnaker/kayenta), Netflix and Google's automated canary service, you already think the right way. You don't fail a canary because one request was slow. You fail it because the *distribution* of a metric moved relative to the baseline by more than noise would explain. A good eval gate works exactly like that: it compares a candidate (new prompt, new model, new chunking strategy) against a baseline on the same cases and asks whether the difference is real.

{{< mermaid >}}
flowchart LR
    D[Dataset of cases] --> R[Runner]
    R --> S[System under test]
    S --> T[Transcript and final state]
    T --> G1[Code graders]
    T --> G2[LLM judge]
    G1 --> A[Aggregator]
    G2 --> A
    A --> B{Compare to baseline}
    B -->|within noise| P[Pass]
    B -->|regression| F[Fail the build]
{{< /mermaid >}}

Graders come in three flavors, and the order matters.

- **Code-based graders** (string checks, schema validation, state checks, retrieval metrics) are cheap, fast and deterministic, so use them wherever you can.
- **Model-based graders**, an LLM acting as judge, handle the open-ended parts but are themselves nondeterministic and need calibration.
- **Human graders** are the ground truth you use to calibrate the other two, sparingly, because they're slow and expensive.

## Example 1: a RAG assistant over your runbooks

Picture an internal assistant that answers on-call questions from your runbook corpus. Someone asks "how do I rotate the TLS cert on the ingress controller?" and it retrieves a few chunks and writes an answer.

The dataset is the part people underestimate. You want 50 to 100 real questions, ideally written by the engineers who carry the pager, each with the IDs of the chunks that actually contain the answer and a short reference answer. One JSONL line per case:

```json
{"id": "rag-017", "question": "How do I rotate the TLS cert on the ingress controller?", "relevant_chunks": ["ingress-tls.md#3", "ingress-tls.md#4"], "reference": "Create the new secret, patch the ingress to reference it, then verify with openssl s_client before deleting the old secret."}
```

The key design decision is to grade retrieval and generation as separate layers, the same way you'd break down end-to-end latency per hop instead of staring at one number. Retrieval is fully deterministic to grade, needs no LLM, and runs in milliseconds.

```python
def recall_at_k(retrieved: list[str], relevant: set[str], k: int) -> float:
    if not relevant:
        return 0.0
    hits = sum(1 for chunk_id in retrieved[:k] if chunk_id in relevant)
    return hits / len(relevant)


def reciprocal_rank(retrieved: list[str], relevant: set[str]) -> float:
    for rank, chunk_id in enumerate(retrieved, start=1):
        if chunk_id in relevant:
            return 1.0 / rank
    return 0.0
```

Generation needs a judge. The question you most care about for an ops assistant is faithfulness: is every claim in the answer supported by the retrieved context? Make the judge return a binary verdict, not a 1-to-10 score. Binary labels are easier to calibrate against human judgments and introduce far less noise. This is something I picked up from Hamel Husain’s excellent article, [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/), which was also my first introduction to the idea of applying systematic evals to AI systems.

```python
import json

JUDGE_PROMPT = """You are grading an answer from an internal ops assistant.

Context retrieved by the system:
{context}

Question: {question}
Answer: {answer}

List every factual claim in the answer that is NOT supported by the context.
Then give a verdict: "pass" if there are no unsupported claims, "fail" otherwise.
If the context is insufficient to decide, return "unknown".
Respond with JSON only: {{"unsupported_claims": [...], "verdict": "pass|fail|unknown"}}"""


def judge_faithfulness(llm, question: str, context: str, answer: str) -> dict:
    raw = llm(JUDGE_PROMPT.format(context=context, question=question, answer=answer))
    result = json.loads(raw)
    if result.get("verdict") not in {"pass", "fail", "unknown"}:
        raise ValueError(f"judge returned invalid verdict: {raw!r}")
    return result
```

Splitting the layers gives you a diagnosis matrix rather than a single score. Good retrieval with unfaithful answers means a prompt or model problem. Bad retrieval with answers that fail on correctness means chunking, embeddings or the index. And bad retrieval with *faithful* answers is the silent failure from earlier: the model is accurately summarizing the wrong document. That case only shows up because you graded retrieval independently, and it's the one that will burn you during an incident at 3 AM.

One more thing: the judge is a dependency, so treat it like one. Pin its model version and calibrate it against human labels; more on how to do that below.

## Example 2: an agent that remediates incidents

Now the harder one. You're building an agent with tools like `get_pods`, `get_logs`, `scale_deployment`, `restart_deployment` and `rollback`, and you want it to handle the boring first ten minutes of common incidents.

The first instinct is to assert on the sequence of tool calls: it should call `get_logs`, then `rollback`. Resist it. Agents regularly find valid paths you didn't anticipate, and asserting on the path makes your evals brittle. Grade the *outcome* instead. This should feel natural if you've worked with Kubernetes: a controller doesn't care which sequence of API calls got the cluster there, it cares whether actual state converged to desired state. Your grader is a reconciliation check.

Each task seeds a simulated cluster into a broken state (an OOMKilled deployment, a bad rollout, a crash-looping pod with a missing config map) and defines three kinds of checks:

1. **Outcome**: the final state matches what you expect, for example the deployment is healthy and running the previous revision.
2. **Invariants**: things that must never happen regardless of outcome, such as touching resources in another namespace or deleting anything.
3. **Budgets**: steps, tokens and wall time, because an agent that fixes the problem in 40 tool calls is a cost and latency incident of its own.

Isolation matters as much here as in any integration test suite. Every trial gets a fresh environment. Shared state between trials causes correlated failures that look like agent regressions but are really infrastructure flakiness. The Anthropic post has a great example of this going the other way: an agent scored better than it should have on internal evals because it could read git history left over from earlier trials.

Then there's the part that has no analog in unit testing: you run each task multiple times. Two metrics capture different things. **pass@k** asks whether at least one of k trials succeeded, which tells you whether the agent *can* solve the task. **pass^k**, introduced in the [τ-bench paper](https://arxiv.org/abs/2406.12045), asks whether *all* k trials succeeded, which tells you whether it solves it *reliably*. They diverge fast. An agent with a 75% per-trial success rate has a pass^3 of about 42%. For anything with write access to production, pass^k is the number you should care about, and it's the one that will match your intuition as an SRE: "works most of the time" is not a reliability property.

Here's the trial loop in Go:

```go
type Task struct {
    ID       string
    Prompt   string
    Seed     func() *Cluster          // builds a fresh broken environment
    Expect   func(c *Cluster) error   // outcome check on final state
    MaxSteps int
}

type Trial struct {
    Passed bool
    Steps  int
    Err    error
}

func RunTask(ctx context.Context, agent Agent, task Task, k int) []Trial {
    trials := make([]Trial, k)
    var wg sync.WaitGroup
    for i := 0; i < k; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            cluster := task.Seed() // isolated state per trial, never shared
            steps, err := agent.Run(ctx, task.Prompt, cluster, task.MaxSteps)
            if err == nil {
                err = cluster.Invariants() // safety first: no forbidden actions
            }
            if err == nil {
                err = task.Expect(cluster) // then: did state converge?
            }
            trials[i] = Trial{Passed: err == nil, Steps: steps, Err: err}
        }(i)
    }
    wg.Wait()
    return trials
}

// PassAtK: at least one trial succeeded. "Can it do this at all?"
func PassAtK(trials []Trial) bool {
    for _, t := range trials {
        if t.Passed {
            return true
        }
    }
    return false
}

// PassHatK: every trial succeeded. "Can I trust it with prod?"
func PassHatK(trials []Trial) bool {
    for _, t := range trials {
        if !t.Passed {
            return false
        }
    }
    return len(trials) > 0
}
```

Write a reference solution for each task too, a scripted agent that follows the known fix. If the reference solution doesn't pass your graders, the bug is in the task or the grader, not the agent. And if a capable model scores 0% across many trials on a task, assume the task is broken before you assume the model is.

## Gates, noise and why your first threshold is wrong

The natural next step is a CI gate: fail the build if the pass rate drops below 85%. That's the equivalent of alerting on a single static threshold, and it has the same problem: it ignores variance.

Do the arithmetic once and it sticks. With 50 cases and a true pass rate of 80%, the standard error is about 5.7 points, so the 95% interval is roughly plus or minus 11 points. A drop from 82% to 76% on a 50-case suite is well within the noise. You have three levers, all familiar from capacity testing:

- **More cases**, to shrink the interval.
- **More trials per case**, to tell a flaky case from a broken one.
- **Paired comparisons**, to compare like with like.

The paired comparison is the canary move. Run the baseline and the candidate on the same cases and look at the flips: which cases passed before and fail now. Ten cases flipping from pass to fail with none flipping the other way is a real signal, even when the aggregate barely moves, and the list of flipped cases is exactly what you need to start debugging.

With that in place, a gate that's strict without being flaky comes down to three rules:

- **Trend the mean, gate on the worst trial.** Dashboards show the mean pass rate, but each case is gated on its worst trial. The user sees one run, never the average, so a case that gives the wrong answer one time in five is broken. That's pass^k again, used as a release criterion.
- **Split suites by intent.** A regression suite holds things the system already does well and should sit near 100%, so any failure is worth investigating. A capability suite holds things you want it to get better at and is supposed to start low. Mixing them produces a number that means nothing.
- **Give the gate a documented way through.** If a change improves most of the suite but regresses a small group of cases, the gate should still fail. A human then consciously accepts the trade-off, overrides the gate and updates the baseline. The point of the gate isn't to stop you shipping; it's to stop you shipping silently.

Finally, tier the cost. It's the same layering you'd use for unit, integration and soak tests, except here every test case costs tokens:

- **Every commit:** retrieval metrics and code graders, which are cheap and deterministic.
- **Pull requests:** LLM-judged suites.
- **Nightly, or before a model upgrade:** multi-trial agent suites with simulated environments.

## What to watch once it's running

The two examples show you the shape of a harness. What they don't tell you is which numbers deserve your attention once it's running in CI and against production. These are the ones I'd start with. It's not an exhaustive list; treat each item as a direction to dig into rather than a complete answer.

**Slice everything.** A global pass rate is a weighted average, and weighted averages hide small populations. If questions about your database runbooks are 3% of what the assistant gets asked and their quality drops by 30 points, the global number moves by less than one. We'll have seen this before: a healthy global p99 while one region is on fire. Pick the dimensions where you expect behavior to differ or where a mistake costs more.

For the runbook assistant that might be the owning team, the type of question (how-to vs diagnosis) or how old the source document is; for the agent, the incident type or the criticality tier of the workload. Tag every case and every production request with them, and compute and gate on each slice independently.

**Separate cheap failures from expensive ones.** A vague answer costs the on-call engineer a minute before they open the runbook themselves. A confident answer with the wrong command gets pasted into a terminal. For the agent, a wasted tool call is a cost problem; scaling the wrong deployment is an incident. If both kinds count as a single "fail" in a blended score, a change that makes answers more detailed but slightly more likely to fabricate can raise your average while raising your risk. Track severe failures (wrong commands, unsafe actions, invariant violations) as a metric of their own, with a much stricter threshold.

**Calibrate the judge, and measure it properly.** Raw agreement between the judge and humans includes agreement by chance. If humans pass 90% of cases and your judge flips a coin, they'll still agree half the time. [Cohen's kappa](https://en.wikipedia.org/wiki/Cohen%27s_kappa) removes that chance component, and for the coin-flipping judge it comes out at exactly zero.

Measure kappa between two humans first. That's your ceiling, because no judge can be held to a higher standard than experts who disagree with each other. Use a judge from a different model family than the system under test, since a judge that shares the generator's blind spots will happily approve its mistakes. And let the people who own the outcome write the rubric. For an ops assistant, that's the engineers on call, which conveniently means you.

**Watch online signals, and watch them diverge from offline ones.** Production gives you implicit feedback that offline evals can't. For the assistant: does the engineer copy the suggested command, or rephrase the question, or open the runbook anyway? For the agent: how often does a human override it, or reopen an incident it marked as resolved? That's the closest thing you have to ground truth for usefulness, but it arrives late, it's noisy and it never tells you why something failed, so it complements the harness rather than replacing it. Its best use is as a cross-check.

If your benchmark score climbs month after month while the override rate stays flat, you're tuning prompts to pass the test, not to help anyone. That's [Goodhart's law](https://en.wikipedia.org/wiki/Goodhart%27s_law) in action. Keep a held-out set that nobody tunes against, and rotate fresh production cases into the suite regularly.

**Watch the inputs, not just the outputs.** Quality usually degrades after the traffic changes shape: a new service gets onboarded, a platform migration rewrites half the runbooks, a new class of incident starts showing up. Tracking the distribution of incoming requests across your slice dimensions gives you a leading indicator. A share-per-category comparison is enough to start, or [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) if you want a single number. The alert isn't "quality dropped". It's "you're now serving traffic your benchmark doesn't cover", which is your cue to add slices and cases before a silent regression settles in.

**Assume the ground moves under you.** In a normal service, if you don't merge any code, last month's test results still hold. Here, your provider can update or retire the model underneath an identical prompt. Run the regression suite on a schedule, not only on merge, so you catch the regressions you didn't cause. Pin the judge model to an explicit version, and keep a small control set with fixed human scores. When you're forced to migrate the judge, run the control set first, so you can tell whether your system regressed or your measuring instrument changed.

## The habits that keep an eval harness honest

A few things I'd treat as non-negotiable:

- **Read the transcripts.** Aggregates hide everything interesting. When a score moves, open twenty failing transcripts before you change a single prompt. Half the time you'll find a grader bug, not an agent bug. It's the same reason you look at the actual trace instead of trusting the p99 panel.
- **Feed production back in.** Every incident where the assistant gave a bad answer becomes a new case, exactly like a postmortem producing a regression test. An eval suite that doesn't grow with production failures rots within a quarter.
- **Version everything together.** Dataset, prompts, graders, judge model and system under test all get versioned, and every run records which versions it used, with a timestamp. Otherwise you can't tell whether the score moved because the system changed or because the measuring instrument did.
- **Roll back the pair, not the prompt.** The deployable unit is the prompt plus the pinned model version. If the provider moved the model yesterday and you shipped a bad prompt today, reverting only the prompt lands you on a combination that has never been tested. Revert both to the last known good pair.
- **Don't page on quality.** An API returning errors, or latency locking up the UI, is an outage. A few points lost on one slice is a decision for business hours with whoever owns the rubric. Page people at 3 AM for a nondeterministic wobble and they'll start silencing the pager, including on the night it matters.
- **Test the harness itself.** Your metric functions, parsers and state checks are deterministic code. They get ordinary unit tests. Ironically, this is the one part of the whole setup where your existing testing habits apply unchanged.

## You already know how to do this

The reframe that made this click for me is simple: an eval harness is SLO-based monitoring for behavior instead of availability. The dataset is your synthetic traffic, the graders are your health checks, the pass rate is your SLI, and the regression gate is canary analysis. The only genuinely new idea is that the thing you're measuring is a distribution of answers, not a status code, so every decision has to account for variance.

I've put together a [companion repo](https://github.com/jreypo/eval-harness-starter) with a working harness for both examples. It runs offline with a stubbed model, so it works in CI without API keys. And to answer the question you're probably asking: yes, I used Claude Code with Opus 5.5 to build the repo, and to help me review this post too.

## My personal take

I know some of the concepts in this article won't click straight away. I still struggle with many of them myself, especially the ones that lean on deeper ML knowledge. Moving into AI engineering means learning new things, but it also lets you put all those years of designing and running systems in production to work, and that makes the effort completely worth it.

And here's my very personal, and possibly controversial, take: I believe systems engineers are especially well prepared for applied AI engineering. Many of us have spent years, some of us decades, going from two-node ServiceGuard clusters to multi-AZ Kubernetes platforms and hyperscaler-class regional compute clusters. We know how systems fail, how to observe them and how to keep them running when they do. At the end of the day, an AI application is still a system that has to be deployed, operated and kept honest in production. The part that's genuinely new is knowing whether it's right, and that's what this whole post has been about. Bring that skepticism with you.

Don't get me wrong, I have nothing but respect for the math PhDs and ML engineers doing the research, writing the papers and regularly making me feel like the dumbest guy in the room. But I still think we have the upper hand here. Not that it's a competition or anything, and I might be completely wrong. In the end, I'm just a distributed systems nerd having fun and teasing my friends on the data side, exactly like I used to tease my DBA friends twenty years ago when I was a Unix sysadmin. Some things never change.

Anyway, I hope you found this useful. Let me know in the comments if you have any questions, or tell me how it went building your own eval harness.

--Juanma
