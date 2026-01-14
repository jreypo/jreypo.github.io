---
title: "Azure DevOps Pipeline Triggers Explained"
date: 2025-05-23T09:30:00+01:00
description: "Automate your CI/CD pipelines with CI, PR, scheduled, and completion triggers"
tags:
- azure
- azure-devops
- devops
showComments: true
---

Azure DevOps **pipeline triggers** allow you to automate the execution of CI/CD pipelines in response to events like code changes, pull requests, schedules, or other pipeline completions. In this article of my series about Azure DevOps, we will explore how pipeline triggers work in **Azure DevOps Services** and **Azure DevOps Server**. We’ll cover the types of triggers available, provide YAML pipeline examples, and discuss best practices, advanced configurations, and troubleshooting.

## Pipeline Triggers in Azure DevOps

**Pipeline triggers** are rules that cause a pipeline to run automatically in response to an event. Instead of manually running pipelines, triggers enable continuous integration, automated testing, and seamless continuous delivery. Azure DevOps supports many trigger types (for both Classic and YAML pipelines), including: Continuous Integration (CI) triggers, Pull Request (PR) triggers, Scheduled triggers, and Pipeline (completion) triggers.

In YAML pipelines (the modern approach), you define triggers in the pipeline YAML file. Azure DevOps **Services** and **Server** (2019+ and especially 2020/2022 versions) both support YAML-defined triggers. Generally, the behavior of triggers is the same in Services and the corresponding version of Server. (Newer trigger features introduced in the cloud will usually appear in the next Azure DevOps Server release.)

**Why use YAML for triggers?** Defining triggers as code (in YAML) brings consistency and versioning. It keeps pipeline configuration alongside your code. While Azure DevOps has UI settings to configure triggers, using YAML is now the recommended approach – it avoids confusion and ensures the pipeline behaves consistently across different environments. (If both UI and YAML triggers are set, the UI settings can override YAML triggers, so it's best to manage triggers in one place, as discussed later.)

## Continuous Integration (CI) Triggers

**Continuous Integration triggers** automatically run a pipeline when code is pushed to a repository branch. This allows immediate build/test feedback on each commit. In Azure DevOps, CI triggers are enabled by default for YAML pipelines – meaning if you don’t specify any `trigger` in your YAML, the pipeline will run on any push to any branch by default. (This default can be disabled by an admin setting, but normally it’s on)

To customize CI triggers, use the `trigger:` keyword in your YAML pipeline. You can include or exclude specific branches, and even filter by file paths or tags. Below is an example of a YAML pipeline with a CI trigger configuration:

```yaml
trigger:
  batch: true            # process one batch at a time (combine commits if pipeline is busy)
  branches:
    include:
      - main             # triggers on pushes to main
      - releases/*       # and any release/* branches
    exclude:
      - dev              # but exclude the dev branch
  paths:
    include:
      - src/*            # only run if changes are in the src folder
  tags:
    exclude:
      - skip-ci          # do not run if commit has tag 'skip-ci'
```

In this example, any code push to the **main** branch or any branch under **releases/** will trigger the pipeline, as long as the changes include files in the `src` directory. Pushes to the **dev** branch are ignored, and if a commit is tagged with “skip-ci”, the pipeline will not run (tag filters apply to Git tags on the commit). The `batch: true` setting is useful in high-frequency scenarios: if another push happens while a run is in progress, Azure Pipelines will wait and then queue a single run including all changes rather than starting multiple runs. By default, `batch` is false (each push creates a run).

**Note on repository types:** CI triggers work for various Git repositories (Azure Repos, GitHub, Bitbucket, etc.). YAML syntax is mostly the same, though some trigger settings (like path filters) apply to Git-based repos. If using TFVC (Team Foundation Version Control), CI triggers are configured differently (and YAML pipelines for TFVC are less common).

## Pull Request (PR) Triggers

**Pull Request triggers** cause a pipeline to run when a PR is opened or updated, typically to validate the proposed changes. YAML PR triggers are defined with the `pr:` keyword. You can filter by target branches (the branches into which the PR is merging) and by path filters similar to CI triggers. Here’s a YAML example for PR triggers:

```yaml
pr:
  branches:
    include:
      - main            # run pipeline for PRs targeting main
      - develop         # and PRs targeting develop
    exclude:
      - experimental/*  # skip PRs merging into any experimental/* branch
  paths:
    include:
      - "src/*"         # only run for changes in src/ (in the PR diff)
    exclude:
      - "docs/*"        # ignore PRs that only change docs
  autoCancel: true       # cancel earlier runs if a new commit is pushed to the PR
```

This configuration will trigger the pipeline when a PR is created or updated targeting the **main** or **develop** branches (except those to any `experimental/*` branch). It will only run if the PR changes files under `src/` (and not *only* docs). The `autoCancel: true` ensures that if the PR has multiple pushes in a short time, only the latest commit gets a running build (earlier pending builds are canceled). By default, `autoCancel` is true.

**Important:** YAML-defined PR triggers are *not supported for Azure Repos Git* repositories. If your code is in Azure Repos, you configure PR validation via **Branch Policies** in the repository settings (which link to a pipeline) rather than via YAML. The YAML `pr:` syntax is only honored for GitHub and Bitbucket Cloud repositories connected to Azure Pipelines. In other words, for GitHub/Bitbucket repos the `pr:` block as shown above will work, but for Azure Repos you should set up a branch policy for build validation (the pipeline still runs, just triggered by the server through the policy).

By default (if `pr:` is omitted in YAML), for GitHub/Bitbucket, all PRs to any branch trigger the pipeline. For Azure Repos, PRs won’t trigger a YAML pipeline unless a branch policy is set, so it’s good practice to explicitly configure PR triggers via the appropriate method for your repo type.

## Scheduled Triggers

**Scheduled triggers** run pipelines on a defined schedule (for example, nightly builds). This is useful for scenarios like running long test suites overnight or doing periodic deployments or maintenance tasks. In YAML, scheduled triggers are configured with a `schedules:` block.

Example of a scheduled trigger in YAML:

```yaml
schedules:
- cron: "0 0 * * *"          # every day at 00:00 UTC
  displayName: "Daily Midnight Build"
  branches:
    include:
      - main                # only run schedule on main branch
  always: true              # always run even if no new changes since last run
```

In this example, the pipeline is scheduled to run **daily at midnight (UTC)** for the `main` branch. The cron syntax `"0 0 * * *"` follows the cron format (`minute hour day-of-month month day-of-week`). We give the schedule a friendly name **"Daily Midnight Build"**. The `always: true` flag means the pipeline will run every day regardless of whether there were changes since the last run. If `always` were false (default), Azure Pipelines would skip the scheduled run if nothing has changed in the source since the previous run, to save resources.

You can define multiple schedules. For example, you might have a nightly build and a separate weekly build on Sundays. You can also include or exclude specific branches in schedules, just like CI triggers (if no branches are specified, it defaults to all branches). Note that the time zone for `cron` is always UTC in Azure DevOps YAML schedules, so plan accordingly when setting the cron string.

**UI vs YAML scheduling:** If you also set up a schedule in the pipeline’s UI settings, it will override the YAML schedule. Azure DevOps gives precedence to triggers configured through the UI over YAML-defined ones for the same pipeline. To ensure your YAML schedule works, remove any UI-configured schedules (the system requires a commit after removal to start using the YAML definitions).

## Pipeline Completion Triggers (Triggering Pipelines from Pipelines)

Sometimes you want one pipeline to trigger another. For example, if you have separate pipelines for different components of a system, a change in a library might trigger a downstream application pipeline to rebuild with the new library. Azure DevOps supports **pipeline triggers** to handle this scenario.
In classic (GUI) builds, this was called build completion trigger. In YAML, you configure it via the `resources:` section. Specifically, you add a pipeline resource and specify `trigger: true` (or more refined filters) so that whenever the referenced pipeline completes, it triggers your pipeline.

Here’s a YAML example of a pipeline triggering another pipeline:

```yaml
# In the downstream pipeline's YAML (e.g., app-ci pipeline)
resources:
  pipelines:
  - pipeline: securitylib          # an identifier for the upstream pipeline resource
    source: security-lib-ci        # name of the upstream (triggering) pipeline
    project: Homelab       # project of the upstream pipeline (if different)
    trigger: true                  # trigger this pipeline when security-lib-ci completes
steps:
- bash: echo "Running after security-lib-ci completes"
```

In this snippet, the pipeline declares a resource pointing to an upstream pipeline named **“security-lib-ci”** (perhaps a CI for a security library). By setting `trigger: true`, the current pipeline (let's call it **“app-ci”**) will automatically run after **security-lib-ci** finishes successfully. If both pipelines are in the same project, the `project:` field isn’t needed, but if the upstream is in a different project, you must specify it. By default, `trigger: true` means trigger on *any* successful run of the upstream pipeline on any branch; you can also configure filters (for example, trigger only if upstream ran on the main branch, etc., using the `branches` or `tags` filter under `trigger`) – the docs describe these advanced filters for pipeline resources.

This mechanism is very powerful for orchestrating complex CI/CD workflows across multiple pipelines. It’s recommended to use YAML pipeline triggers rather than the old UI-based build completion triggers, as YAML triggers have fewer limitations (for instance, YAML can ensure the triggered pipeline uses the same commit of the code if they share a repository).

## Other Trigger Types and Advanced Scenarios

In addition to the main types above, Azure Pipelines supports triggers on other resources and advanced conditions:

- **Resource (Artifact) Triggers:** YAML pipelines can trigger on changes to external resources like container images or package feeds. For example, you can declare a container image resource from Azure Container Registry (ACR) with a `trigger` so that a pipeline runs whenever a new image tag is pushed. Below is a snippet illustrating a container trigger integration.
  
```yaml
resources:
    containers:
    - container: appdevcontaner
    type: ACR
    azureSubscription: starlabsdev  # service connection name
    resourceGroup: starlabs-dev-rg
    registry: starlabsACR
    repository: kubeappdev/frontend
    trigger:
        tags:
        include: 
            - release*    # trigger when a tag starting with "release" is pushed
```

   Here, the pipeline will start whenever a new container image tagged like "release..." is pushed to the `kubeappdev/frontend` repository in the ACR registry. You could similarly trigger on specific package versions in Azure Artifacts feeds (using the `resources: packages` syntax). This is useful for automating deployments when a new version of a dependency is available, decoupling the build processes.

- **Multiple Repositories:** If your pipeline pulls code from multiple repositories, you can set triggers on changes in those external repositories too. This is done by adding repository resources with a `trigger` section. For instance, if you have a `resources:` > `repositories:` entry for a secondary repo, you can specify which branches in that repo should trigger the pipeline to run.
- **Branch and Path Filters:** We touched on these in CI and PR triggers. They allow fine-grained control to avoid unnecessary runs. For example, you might only trigger a documentation pipeline when files under a `/docs` folder change, or exclude the `docs` folder from a build pipeline’s triggers. Using wildcards (like `features/*`) in branch names is supported to cover branch patterns. Always double-check that your include/exclude filters cover your scenario – a common mistake is accidentally excluding all branches by using an exclude without a matching include (which effectively means “exclude nothing”, thus triggering on everything).
- **Combining Trigger Types:** You can use multiple triggers in one pipeline. For example, a pipeline can have both CI triggers and a nightly scheduled trigger, or both CI and pipeline triggers. They will each fire the pipeline on their respective events. Just ensure they are defined at the root of the YAML (and remember UI triggers can override YAML if not careful).
- **Trigger Conditions within Pipeline (not same as triggers):** Note that within a pipeline’s jobs or stages, you can also set conditions to decide whether to run a job (e.g., only run certain steps on certain branches). This is different from pipeline triggers but is another method to handle multi-branch scenarios in one pipeline. We won't dive into those here, but it's good to know the distinction: triggers decide **whether the pipeline starts**, whereas job/stage conditions decide **what parts execute once the pipeline has started**.

## Disabling Automatic Triggers with `trigger: none`

If you remember in the previous [article about Variable Groups]({{< ref "posts/2025-05-14-managing-secrets-securely-in-azure-devOps-server-with-variable-groups.md" >}}) I had a pipeline example with the trigger `none`. A trigger can be set to `none` to **disable automatic runs** (e.g., no CI or PR triggers).

```yaml
trigger: none
pr: none
```

This is ideal when:

- You want to **run the pipeline manually** (e.g., for testing in a lab environment).
- You use the pipeline only as a **template or shared logic**.
- You want **full control** over when the pipeline runs.
  
---

### How to Trigger Manually

After setting `trigger: none` and `pr: none`, you can **manually trigger the pipeline** from the Azure DevOps or Azure DevOps Server portal:

1. Go to **Pipelines -> \[Your Pipeline]**.
2. Click the **Run pipeline** button.
3. (Optional) Provide parameters or branch overrides if defined.

In the case of Azure DevOps Services you can also trigger it via the **Azure CLI**:

```text
az pipelines run --name "<pipeline_name>" --branch "main"
```

Or use the **REST API** for automation.

In Azure DevOps Server you can trigger the pipeline via the **REST API** with `curl`.

```bash
curl -u :<PAT> \
  -X POST \
  -H "Content-Type: application/json" \
  https://<devops-server-url>/<collection>/<project>/_apis/build/builds?api-version=6.0 \
  -d '{
        "definition": {
          "id": <pipeline_id>
        },
        "sourceBranch": "refs/heads/main"
      }'
```

**Replace:**

- `<PAT>` – Your Personal Access Token (no username needed, just use `:`).
- `<devops-server-url>` – Your server's base URL (e.g., `devops.mycompany.local`).
- `<collection>` – The name of the collection (e.g., `DefaultCollection`).
- `<project>` – Your project name.
- `<pipeline_id>` – The numeric ID of the pipeline you want to trigger.

To get your pipeline ID go to your pipeline in Azure DevOps Server and check the URL. It looks like:

```html
.../_build?definitionId=42
```

The number after `definitionId=` is what you need.

And you can use more methods to interact with Azure DevOps API, here is quick python script I wrote to trigger any pipeline in my homelab Azure DevOps Server.

```python
#!/usr/bin/env python

import requests
from requests.auth import HTTPBasicAuth
import getpass
import json

# Prompt for inputs
devops_url = input("DevOps Server URL (e.g., https://devops.local/DefaultCollection): ").strip().rstrip('/')
project = input("Project name: ").strip()
pipeline_id = input("Pipeline (definition) ID: ").strip()
pat = getpass.getpass("Enter your Personal Access Token (PAT): ")
branch = input("Branch to run (default: main): ").strip() or "main"

# Optional parameters
params = {}
add_params = input("Add pipeline parameters? (y/N): ").strip().lower()
while add_params == 'y':
    key = input("Parameter name: ").strip()
    value = input("Parameter value: ").strip()
    params[key] = value
    add_params = input("Add another parameter? (y/N): ").strip().lower()

# API endpoint
url = f"{devops_url}/{project}/_apis/build/builds?api-version=6.0"

# Payload
payload = {
    "definition": {
        "id": int(pipeline_id)
    },
    "sourceBranch": f"refs/heads/{branch}"
}

if params:
    payload["parameters"] = json.dumps(params)  # Parameters must be a JSON string

# API request
response = requests.post(
    url,
    json=payload,
    auth=HTTPBasicAuth('', pat),
    headers={'Content-Type': 'application/json'}
)

# Output
if response.status_code == 201:
    print("\n Pipeline triggered successfully!")
    print(f"Build ID: {response.json().get('id')}")
else:
    print(f"\n Failed to trigger pipeline: {response.status_code}")
    print(response.text)

```

## Best Practices for Setting Up Pipeline Triggers

To get the most out of pipeline triggers and avoid pitfalls, consider the following best practices:

- **Specify Triggers Explicitly:** Don’t rely solely on the default “trigger on everything” behavior. In your YAML, explicitly define which branches (and paths) should trigger the pipeline. This avoids surprises (e.g., someone creating a new branch causing unexpected pipeline runs). Use `trigger: none` if you want to strictly control pipeline runs manually or via other triggers.
- **Use Branch Filters and Path Filters:** Limiting triggers to relevant branches and file paths can greatly reduce unnecessary pipeline runs. For example, if your pipeline only builds a Java service, you might exclude changes to a `docs/` directory or other unrelated paths so those don’t trigger a build. This makes CI faster and more efficient.
- **Avoid Trigger Loops:** Be careful with pipeline triggers to avoid circular dependencies where Pipeline A triggers Pipeline B, and B (perhaps indirectly) triggers A. This can create infinite trigger loops. If you have such dependencies, consider using conditions or manual approvals to break the cycle, or consolidate into one pipeline if appropriate.
- **Control Batch vs Parallel:** Use the `batch` setting (for CI triggers) if your team pushes very frequently and you want to avoid queueing multiple redundant runs. However, if quick feedback is critical, you might leave `batch` false so that each commit is built independently (parallel builds). Adjust based on repo activity.
- **Manage Scheduled Triggers Thoughtfully:** For scheduled triggers, choose times that won’t conflict with peak usage or deployments. Also, set `always: false` (the default) if there’s no need to run when nothing changed, to conserve build resources. If you *do* need a run regardless of changes (for example, refreshing data or doing cleanup), then use `always: true` as shown before.
- **Use Pipeline Resources for Multi-Stage Workflows:** Instead of one gigantic pipeline for everything, consider splitting logically distinct processes into separate pipelines and use pipeline triggers (resources) to connect them. This can make your CI/CD process more modular and easier to maintain. For instance, have a build pipeline that triggers multiple deploy pipelines for different environments.
- **Leverage Branch Policies (for Azure Repos):** If you use Azure Repos, set up branch policies for PR validations. This ensures that even if someone modifies the YAML to remove PR triggers, the policy still requires the pipeline to run before merging. It’s an extra guardrail for quality and security.
- **Documentation and Naming:** Clearly name your pipelines and use `displayName` for triggers (especially schedules) so that it’s obvious in the Azure DevOps UI *why* a pipeline ran. For example, runs triggered by another pipeline will indicate the source pipeline. A good naming convention (e.g., “[Project] CI”, “[Project] Nightly”, etc.) helps team members understand the pipeline triggers at a glance.
- **Test Trigger Configurations:** If you set up a complex trigger (like multiple branch filters, or cross-project triggers), test it out. For example, push a dummy commit to a branch that should trigger the pipeline and verify it works, and push to one that should not trigger and ensure it stays quiet. This helps validate that your YAML is correct.

## Security Considerations for Pipeline Triggers

Even if you are running Azure DevOps Server in your homelab for automation or learning purposes, you should be mindful of security and access control with triggers. All these recommendations are valid again for Azure DevOps on-premises and in the cloud.

- **Protect Sensitive Pipelines:** If a pipeline deploys to production or performs critical actions, ensure that only trusted code can trigger it. For example, restrict the pipeline’s triggers to the `main` branch which is protected by approvals or limited contributors. This prevents un-reviewed changes from triggering sensitive workflows.
- **Fork PRs and External Contributions:** If you run pipelines on code from forked repositories (common with open-source on GitHub), be cautious. Azure Pipelines by default does **not** expose secrets to builds of forks, and you should keep it that way. It’s recommended to require manual approval or use comment triggers for running CI on forks, so a maintainer can review the forked code before any sensitive steps run. In short, don’t automatically run a pipeline with secrets on code you haven’t vetted.
- **Scoped Service Connections:** If using pipeline triggers across projects or to external resources, ensure the service connections have minimal necessary permissions[. For example, if Pipeline A triggers Pipeline B in another project, Pipeline B might need a service connection to access Pipeline A’s artifacts – scope that connection to only what’s needed (perhaps a specific artifact feed or storage). Azure DevOps lets you scope service connections to resource groups or specific repositories, which can limit damage if credentials are misused.
- **Use Branch Policies:** As noted earlier, branch policies in Azure Repos can enforce that certain pipelines run successfully before allowing a PR to be completed. This ensures that the trigger mechanism is part of a controlled quality gate. Policies can also prevent certain branches from having direct pushes (requiring PRs instead), adding another layer of control to what triggers a pipeline.
- **Pipeline Permissions:** Azure DevOps has a feature to set pipeline-level permissions. For example, you can restrict who can manually run a pipeline or even for resources pipelines, who can trigger changes. Make sure critical pipelines are not accidentally run by unauthorized users (though triggers are automatic, someone could manually run a pipeline with different parameters if they have access).
- **Audit and Monitor:** All pipeline runs are logged. It’s good practice to monitor your pipeline runs for any unusual triggers. Azure DevOps can send notifications on pipeline events – set up alerts for runs of certain pipelines, especially if they should only trigger rarely (e.g., production deployments). If one triggers unexpectedly, you can investigate immediately.
- **Secret Variables and Triggers:** A pipeline trigger will carry over the pipeline’s variable settings. Ensure that secret variables are marked secret and that your pipeline doesn’t accidentally expose them. When using triggers like PR triggers, remember that with forks Azure DevOps will not automatically allow secrets (as mentioned). If you manually trigger or override this, it could expose secure info – so avoid that.

In summary, maintain tight control on what events can start your pipelines, especially those that perform deployments or handle sensitive data. Use the principle of least privilege at every level – from Git branch protections to pipeline permissions – to secure your CI/CD workflows.

## Troubleshooting Pipeline Trigger Issues

Finally some troubleshooting tips and recommendations, based on my experience working every day with Azure DevOps at Microsoft and with Azure DevOps Server in my homelab.

- **Pipeline not triggering at all:** First, check if there is a trigger override in the UI. In Azure DevOps, if someone edited the pipeline’s settings in the UI, they might have enabled “Override the YAML trigger” which can disable your YAML-defined triggers. Go to the pipeline’s settings (… menu -> Triggers) to ensure no override is in effect for CI or PR triggers.
- **PR trigger not firing (Azure Repos):** Remember that YAML PR triggers don’t work with Azure Repos. If you expected a PR to trigger the pipeline but nothing happened, for Azure Repos the solution is to use a branch policy for build validation. Probably the system is behaving correctly by ignoring `pr:` in YAML for Azure Repos – the pipeline must be linked via branch policy instead.
- **Missing `trigger:` section and no runs:** If you omitted the `trigger:` in YAML, Azure DevOps usually triggers on all branches by default. However, this implicit trigger can be turned off by an admin setting (introduced in Server 2022.1) called **“Disable implied YAML CI trigger”**. If that setting is enabled, and you have no `trigger:` in the YAML, then pushes won’t start the pipeline. The fix is either to explicitly add a `trigger:` section to YAML or ask an admin to enable the implied triggers setting.
- **Branch or path filters not working as expected:** Double-check your include/exclude patterns. If you specify an `exclude` without any `include`, it implicitly includes all branches and then excludes the ones listed. For example, if you wrote `branches: exclude: [main]` (with no include), it means “run on every branch except main.” Make sure that’s what you intended. Also ensure wildcards (`*`) are used correctly (they should be quoted in YAML, e.g. `'*.md'`, to avoid YAML parsing issues).
- **Scheduled pipeline timing issues:** All YAML cron schedules are in UTC. If your pipeline didn’t run at the expected local time, it might be due to time zone confusion. Convert your desired time to UTC for the cron expression. Also note the day-of-week in cron is also evaluated in UTC (so a “Monday 3AM” UTC trigger actually occurs Monday 3 AM UTC, which might be a different local day depending on your timezone).
- **Pipeline triggers triggering wrong branch/version:** Because each branch can have its own YAML, it’s possible that the triggers defined in one branch’s YAML differ from another’s. Azure DevOps will use the YAML *in the branch where the event occurred* to decide triggers. For example, if you have more restrictive triggers in the `develop` branch’s YAML, pushing to `develop` will use that YAML’s rules, whereas pushing to `main` uses `main`’s YAML. Make sure your branches have updated YAML if you’ve made trigger changes – otherwise behavior may differ by branch.
- **Pipeline completion trigger not firing:** For pipeline triggers (resources), ensure the names and project references are correct and that the triggering pipeline is actually completing successfully. Also, if the pipelines are in different Azure DevOps projects or organizations, verify that the connection (service connection for cross-project triggers or pipeline resources) is set and authorized. If a pipeline resource trigger isn’t working across projects, it could be due to missing permissions or the `project:` name being wrong.
- **UI vs YAML confusion:** This is avery common mistake if you are using both YAML and the UI to manage your pipelines. Remember that if you have set up triggers both via YAML and via the pipeline UI, the UI may override YAML for CI and PR triggers, and will override YAML for schedules. The resolution is to pick one approach to manage triggers (preferably YAML) and remove the conflicting settings.

Azure DevOps provides a **"Run pipeline"** history that shows the trigger for each pipeline execution (e.g., "CI trigger on commit X," "PR trigger," "Scheduled trigger," or "Manually run by ..."). Checking the details of a specific pipeline run can provide valuable clues for troubleshooting. Also, if you're not an administrator of your Azure DevOps organization (the most common scenario), remember to consult with your admin team regarding the various policies and settings in place. In your homelab, however, you are the administrator, so remember to properly document your Azure DevOps Server configuration. I personally use the built-in Azure DevOps Wiki and host a [DocFX](https://dotnet.github.io/docfx/index.html) instance in my lab to record and document all my homelab configurations and guidance, which is helpful if I ever need to recreate a component from scratch.

---

And with that, we're done. I hope this was useful and helped you understand what Azure Pipelines triggers are and how they work. My intention was to provide a quick overview, but I got a bit carried away! In the next article, I will finally explain Self-hosted Agents. I had planned to cover them here, but I decided to first write an overview of pipeline triggers since they are a key part of Azure DevOps.

--Juanma
