---
title: "YugabyteDB: Postgres for the Distributed Era"
date: 2026-09-03 18:18:00 +02:00
description: "A hands-on tutorial on YugabyteDB: its history, why the DocDB/Raft architecture makes it a distributed, PostgreSQL-compatible database rather than just another NewSQL rewrite, real production deployments, and working Python and Go examples."
tags: 
- yugabytedb
- distributed-systems
- databases
- devops
showComments: true
---

Back in July I wrote about [ClickHouse]({{< ref "posts/2026-07-12-clickhouse-what-all-the-fuss-is-about.md" >}}), a database built to answer "how many of X happened, grouped by Y" over billions of rows in milliseconds and that I currently use on a daily basis in my role at Lobera. That's an [OLAP](https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/online-analytical-processing) problem.

This post is about the other half of the workload every real system has: the [OLTP](https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/online-transaction-processing) half: "does this specific order exist, update its status, and don't you dare lose it or double-charge the customer while you're at it." That's a completely different set of guarantees, and for a long time, scaling it horizontally without giving up SQL and ACID transactions meant either paying a hyperscaler for Spanner-as-a-service or building something extremely painful yourself.

YugabyteDB is one of a small handful of databases (CockroachDB being the other name you'll hear most) that set out to close that gap: a database that speaks actual PostgreSQL on the wire, survives losing a node or an entire region without going down, and scales writes horizontally rather than just reads. And, again, it is something I work with regularly as a systems engineer rather than something I read about once.

This is a practical, hands-on tutorial following a similar pattern to the ClickHouse article: covering what it actually is, where it came from, why the architecture is built the way it is, some real production gotchas, and working code in Python and Go.

## What YugabyteDB actually is

[YugabyteDB](https://www.yugabyte.com/) is an open source, distributed SQL database built for OLTP (transactional) workloads, the kind of access pattern with lots of small, concurrent reads and writes that need to be strongly consistent, as opposed to the big analytical scans ClickHouse is built for. The pitch, in one sentence: take PostgreSQL, keep its SQL surface and wire protocol close to identical, and replace the single-node storage engine underneath with a distributed, horizontally-scalable, Raft-replicated one.

That distinction matters because it's the main thing that separates YugabyteDB from most other distributed SQL databases. Rather than writing a new SQL engine from scratch and aiming for rough PostgreSQL compatibility, YugabyteDB actually reuses the upstream PostgreSQL query layer, this is parser, planner, optimizer, a large chunk of the executor, as the top half of the stack. Below that, instead of PostgreSQL's original single-node storage engine, sits a completely different, purpose-built distributed storage layer called [DocDB](https://docs.yugabyte.com/stable/architecture/docdb/). The practical upshot is a database that supports an unusually large slice of real PostgreSQL: stored procedures, triggers, extensions (the ones that don't reach into the storage layer), and the full breadth of PostgreSQL's type system and SQL dialect, all while distributing data, replication, and transactions across a cluster of nodes automatically.

It also ships a second, less commonly used API called [YCQL](https://docs.yugabyte.com/stable/quick-start/explore/ycql/), a Cassandra-compatible query language, sitting on the exact same DocDB storage layer. Most people reading this will care about YSQL (the PostgreSQL-compatible one), so that's what this post focuses on.

One question that "PostgreSQL compatible" always begs is: compatible with *which* PostgreSQL? YSQL started life on PostgreSQL 11.2 and stayed there for years, with individual features and fixes cherry-picked from later releases. That changed with [v2.25 (preview) and v2025.1 (stable)](https://docs.yugabyte.com/stable/api/ysql/pg15-features/), where Yugabyte moved the fork up to PostgreSQL 15 wholesale, bringing along things like stored generated columns, foreign keys on partitioned tables and non-distinct NULLs in unique indexes. Worth knowing for two reasons: it tells you which PostgreSQL features you can actually count on, and it means moving an existing cluster from a PG11-based release (v2024.2 and earlier) to a PG15-based one is a [major version upgrade](https://docs.yugabyte.com/stable/manage/ysql-major-upgrade-yugabyted/) with its own procedure, not a routine rolling restart. If you are starting fresh today, you land on PG15 and this is somebody else's problem.

## A bit of history

YugabyteDB's origin story runs straight through Facebook's infrastructure team in the mid-2000s and 2010s. Kannan Muthukkaruppan, Karthik Ranganathan, and Mikhail Bautin all worked there on the team that built and operated [Apache Cassandra](https://cassandra.apache.org/_/index.html) and [HBase](https://hbase.apache.org/), hardening both to run the operational data stores behind Facebook Messenger at a scale few companies have ever had to deal with. Karthik in particular was one of the original database engineers on that team, an early contributor to Cassandra before Facebook open sourced it, and an Apache HBase committer. After Facebook, Kannan and Karthik spent time at Nutanix working on distributed, hybrid-cloud data infrastructure, which turned out to be exactly the "how do enterprises actually operate this stuff" experience the next step needed.

In February 2016, the three of them founded Yugabyte to build what became YugabyteDB. The stated goal was blunt: bring the operational properties they'd spent a decade building for NoSQL systems (horizontal scale, resilience to node and zone failure, geographic distribution) to applications that still wanted real SQL, real transactions, and a data model developers already knew, instead of asking every team to redesign their data model around a key-value or wide-column store. The name blends the Sanskrit word "yuga" (a long era or age) with "byte," which is a fairly on-the-nose way of saying "we want this to be the storage layer that outlasts whatever infrastructure decisions you made this year."

The company initially split its features between an open community edition and a commercial enterprise edition, a fairly standard model at the time. That changed in July 2019, when Yugabyte open sourced its previously commercial features entirely under Apache 2.0. By the company's own account this was closer to an experiment than a grand strategic bet, and it worked: adoption picked up substantially once the whole database, not just a subset, was available to run and inspect for free. YugabyteDB 2.0, released later that year, was the point where YSQL (the PostgreSQL-compatible API) matured from beta into the primary, production-recommended way to use the database, alongside the original YCQL API it had launched with.

Yugabyte raised a $188 million Series C in October 2021 (led by Sapphire Ventures, with Wells Fargo Strategic Capital, Meritech, Alkeon, and existing investors like Lightspeed and 8VC participating), pushing the company's valuation past $1.3 billion and its total funding to roughly $290 million. That's still the company's most recent disclosed funding round as of this writing. Also in 2021, Yugabyte launched YugabyteDB Aeon (originally Yugabyte Cloud), a fully managed DBaaS offering, the same open-core-plus-managed-cloud business model you'll recognize from ClickHouse, MongoDB, and most of the current generation of infrastructure companies.

## Why it works this way: the architecture

{{< mermaid >}}
graph TB
Master["YB-Master Raft group<br/>3 nodes<br/>catalog, sharding, load balancing"]
subgraph N1 ["YB-TServer node 1"]
T1L["Tablet A<br/>Leader"]
T2F1["Tablet B<br/>Follower"]
T3F1["Tablet C<br/>Follower"]
end
subgraph N2 ["YB-TServer node 2"]
T2L["Tablet B<br/>Leader"]
T1F2["Tablet A<br/>Follower"]
T3F2["Tablet C<br/>Follower"]
end
subgraph N3 ["YB-TServer node 3"]
T3L["Tablet C<br/>Leader"]
T1F3["Tablet A<br/>Follower"]
T2F3["Tablet B<br/>Follower"]
end
Master -.metadata.-> T1L
Master -.metadata.-> T2L
Master -.metadata.-> T3L
Client["YSQL Client<br/>smart driver, cluster-aware"]
Client --> T1L
Client --> T2L
Client --> T3L
{{< /mermaid >}}

Operations in a YugabyteDB cluster ("universe," in Yugabyte's terminology) are split into two logical layers, and understanding that split explains most of the architecture's behavior.

**The query layer (YSQL / YCQL)** is stateless and handles parsing, planning, and the API-specific parts of a request. For YSQL this is, quite literally, PostgreSQL's own query layer, which is why feature compatibility is so deep and why tools built for Postgres (drivers, ORMs, `psql`-alikes) tend to just work.

**The storage layer (DocDB)** is where the distribution actually happens, and it's completely agnostic to which API sent the request. Every table is automatically split into **tablets**, shards, in more familiar terminology, using either hash sharding (the primary key, or a chosen prefix of it, is hashed into a fixed keyspace and each tablet owns a contiguous slice of that hash range) or range sharding (tablets own contiguous ranges of the actual sorted key values, useful for range scans but prone to a "single hot tablet at start" problem discussed below). Each tablet is replicated onto multiple nodes (three, in the standard replication-factor-3 setup) and those replicas form an independent **Raft consensus group**: one leader handling reads and writes for that tablet, and followers replicating the Raft log. A cluster with, say, 16 tablets and RF=3 is running 16 completely independent Raft groups, each electing its own leader and tolerating the loss of a minority of its replicas without losing availability or data.

Underneath each tablet's Raft log sits **DocDB's local storage engine**, a heavily customized fork of RocksDB, the same LSM-tree key-value engine that powers things like Meta's MyRocks. YugabyteDB layers a flexible document data model on top of RocksDB's flat key-value interface (which is where the "Doc" in DocDB comes from), enabling things like updating a single field of a row without a full read-modify-write, and it disables RocksDB's own write-ahead log since the Raft log already provides durability; recording only enough sequence-number bookkeeping to know how much of the Raft log can be safely garbage collected once a RocksDB flush has persisted it.

Two other processes round out the picture, and they map directly onto Linux processes you'll actually see running:

- **YB-TServer**, one per node, hosts the tablets, serves the actual reads and writes, and does the heavy lifting: compactions, block cache management, load-based rebalancing across disks.
- **YB-Master**, a small (typically three-node) Raft group of its own, acts as the cluster's catalog and orchestrator, it tracks which tablet lives on which TServer, drives automatic load balancing when nodes are added or removed, and coordinates schema changes. Critically, YB-Master is never in the hot path of a read or write; if it's briefly unavailable, the cluster keeps serving traffic, it just can't do things like create a new table or rebalance until it recovers.

Distributed transactions spanning multiple tablets (and therefore potentially multiple nodes) use a provisional-records mechanism plus hybrid logical clocks for ordering, in the same conceptual family as Google Spanner's TrueTime-based approach and CockroachDB's own hybrid-clock design, the practical difference being YugabyteDB and CockroachDB both do this with commodity clocks and no dependency on atomic clocks or GPS receivers, at the cost of slightly different consistency/latency trade-offs than what TrueTime buys you.

## What it's good for, and what it isn't

YugabyteDB is a strong fit for:

- **Transactional microservices that have outgrown a single Postgres instance**, where you need horizontal write scaling, not just read replicas, and don't want to bolt on an external sharding or caching layer to get there.
- **Applications that need to survive losing a node, a zone, or an entire region** without a manual failover runbook, since the Raft-per-tablet design means failover is automatic and typically measured in seconds.
- **Global or multi-region applications** with real data residency or latency requirements; geo-partitioning table data to specific regions, combined with local reads via read replicas, is a first-class, documented pattern rather than an afterthought.
- **Consolidating multiple special-purpose databases** (a Cassandra cluster here, a Postgres instance there, maybe a graph database for one specific feature) into one operationally simpler system, when the actual access patterns don't strictly require the specialized engine.
- **Teams that already know PostgreSQL** and want to avoid retraining developers on a new query language or data model just to get distributed scale.

It's a weaker fit for:

- **Analytical or reporting workloads.** This is the flip side of the ClickHouse post: DocDB is built for point lookups and small-range scans with strong consistency, not for scanning billions of rows and aggregating. Running heavy analytics directly against your OLTP cluster is exactly the mistake both posts are, in their own way, arguing against. Feed a columnar store like ClickHouse via change data capture instead, and let each database do the job it's actually good at.
- **Small, single-node applications with no real scaling or resilience requirement.** A distributed system has an operational floor, three nodes minimum for a meaningful replication factor, that plain PostgreSQL simply doesn't. If you don't need that, you're paying complexity tax for nothing. Worth noting that YugabyteDB does have an answer for the related "I have two hundred tiny tables and don't want two hundred Raft groups" problem, which is [colocation](https://docs.yugabyte.com/stable/additional-features/colocation/): small tables in a colocated database share a single tablet, and you split out only the ones that actually grow. That solves the sharding overhead, not the three-node floor.
- **Extension-heavy Postgres deployments.** If your application leans on a specific set of PostgreSQL extensions, that list is the first thing to check, not the last. More on why below.
- **Microsecond-latency requirements**, the territory of in-memory stores like Redis. Raft consensus and cross-node coordination are not free, and no distributed SQL database competes with an in-memory cache on raw latency for the workloads caches are actually built for.

## Real production deployments

**Kroger.** One of the more interesting public case studies, and a nice illustration of the "consolidation" use case: Kroger's engineering team modernized its data layer by migrating off a mix of Cassandra, Neo4j, Microsoft SQL Server, and CockroachDB instances onto a single YugabyteDB cluster, running over 5,000 cores in production and powering high-value microservices including its shopping cart application, running both on-premises and across multiple public clouds. Kroger's own account of the migration emphasizes that YugabyteDB's PostgreSQL compatibility meant their teams didn't have to retrain developers or rewrite application schemas to move off the legacy systems, for a retailer with decades of legacy investment, that compatibility was as much the point as the horizontal scaling itself.

**Charles Schwab and Paramount+.** Both are publicly listed as production YugabyteDB users by Yugabyte, Schwab for workloads handling extremely high volumes of reads and writes in financial services, and Paramount+ for scaling and multi-region reliability behind its streaming service. Worth noting these figures come from vendor-published case studies rather than independent benchmarks, so treat exact numbers with the same healthy skepticism you'd apply to any vendor's own success stories.

**Narvar.** A smaller but technically specific data point: Narvar has publicly described using YugabyteDB to avoid cloud lock-in while staying GDPR compliant, and (in Yugabyte's own published testimonial, worth flagging as vendor-supplied rather than an independent comparison) reported better performance with fewer resources than they'd seen running CockroachDB for the same workload. Take that specific comparison as one customer's data point rather than a general verdict on the two databases, they're architecturally closer to each other than either is to anything else on the market, and workload shape matters enormously in these comparisons.

Beyond these, Yugabyte's published customer list includes GM, Wells Fargo, Netskope, ComplyAdvantage, and a large Japanese telecom running IoT device connectivity at scale; a fairly consistent pattern of financial services, retail, and telecom companies with genuine multi-region or high-availability requirements, rather than workloads that just wanted "a bigger database."

## Hands-on: get it running locally

The quickest path is Docker, using `yugabyted`, a single management binary that wraps starting both the YB-Master and YB-TServer processes for you:

```bash
docker run -d --name yugabyte \
  -p 7000:7000 -p 9000:9000 -p 15433:15433 -p 5433:5433 -p 9042:9042 \
  yugabytedb/yugabyte:latest \
  bin/yugabyted start --background=false
```

Port 5433 is the YSQL (PostgreSQL-compatible) endpoint, 9042 is YCQL, and 7000/9000/15433 serve the YB-Master, YB-TServer, and the newer unified web UIs respectively. All three are worth a look, they show you tablet placement and leader/follower status in real time. On macOS Monterey or later, AirPlay Receiver squats on port 7000 by default, so swap in `-p 7001:7000` if the container fails to start.

`latest` is fine for kicking the tires, but pin an explicit tag for anything you want to be reproducible. The PostgreSQL version underneath moved from 11 to 15 at v2025.1, so which image you happened to pull determines which SQL features you get, and that is not a fun thing to discover from a failing test six weeks later.

One thing to keep in mind for the rest of this post: this is a single node. It's enough to exercise every bit of SQL below, but a one-node "cluster" obviously can't demonstrate load balancing across nodes or leader distribution, so the driver features further down will technically work while having nothing to spread across. If you want to actually watch that happen, start three containers and join them with `yugabyted start --join`, then re-run the same code and compare what `yb_servers()` returns.

Since 5433 is exposed to the host and speaks the actual PostgreSQL wire protocol, you can connect with `psql` directly if you have it installed:

```bash
psql -h 127.0.0.1 -p 5433 -U yugabyte -d yugabyte
```

Or, without installing anything extra, use the `ysqlsh` shell bundled in the container itself:

```bash
docker exec -it yugabyte bash -c '/home/yugabyte/bin/ysqlsh --echo-queries --host $(hostname)'
```

Either way, you land in a prompt that looks and behaves like `psql` because, under the hood, most of it is:

```sql
CREATE TABLE orders (
    order_id     UUID NOT NULL DEFAULT gen_random_uuid(),
    customer_id  INT NOT NULL,
    status       TEXT NOT NULL DEFAULT 'pending',
    total_cents  INT NOT NULL,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (order_id HASH)
) SPLIT INTO 12 TABLETS;
```

Two things in that DDL are YugabyteDB-specific and worth pausing on.

`PRIMARY KEY (order_id HASH)` says to hash-shard on `order_id`, which is what you want for a random UUID: the hash spreads keys evenly across tablets and there are no range queries on a UUID worth preserving anyway. Note that this is [the default](https://docs.yugabyte.com/stable/explore/going-beyond-sql/data-sharding/) and not a behavioral change; hash sharding is what you get on the first primary key column unless you ask for `ASC` or `DESC`, and the default itself is configurable through `yb_use_hash_splitting_by_default`. I write it explicitly because sharding is the single most consequential decision in the schema and I'd rather it be visible in the DDL than inferred by whoever reads it next.

`SPLIT INTO 12 TABLETS` presplits the table into 12 tablets at creation time instead of letting the defaults pick a number based on cluster size. This is useful once you have a rough sense of how big the cluster is and how much write volume is coming, and it matters much more for range-sharded tables, which start life as a single tablet and only split as data arrives.

## Python example

YugabyteDB ships a "smart" psycopg2 driver, a thin layer over the standard one that adds cluster-aware connection load balancing, so your application doesn't need an external load balancer or a hardcoded node list in front of it.

```bash
pip install psycopg2-yugabytedb
```

```python
import psycopg2
import random
from datetime import datetime, timezone

conn = psycopg2.connect(
    dbname="yugabyte",
    host="127.0.0.1",
    port="5433",
    user="yugabyte",
    password="yugabyte",
    load_balance="true",  # discovers all nodes, spreads connections across them
)
conn.set_session(autocommit=True)
cur = conn.cursor()

cur.execute("""
    CREATE TABLE IF NOT EXISTS orders (
        order_id     UUID NOT NULL DEFAULT gen_random_uuid(),
        customer_id  INT NOT NULL,
        status       TEXT NOT NULL DEFAULT 'pending',
        total_cents  INT NOT NULL,
        created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
        PRIMARY KEY (order_id HASH)
    ) SPLIT INTO 12 TABLETS
""")

statuses = ["pending", "paid", "shipped", "delivered", "cancelled"]

rows = [
    (random.randint(1, 5_000), random.choice(statuses), random.randint(500, 50_000))
    for _ in range(20_000)
]
cur.executemany(
    "INSERT INTO orders (customer_id, status, total_cents) VALUES (%s, %s, %s)",
    rows,
)

cur.execute("""
    SELECT status, count(*) AS orders, round(avg(total_cents) / 100.0, 2) AS avg_order_usd
    FROM orders
    GROUP BY status
    ORDER BY orders DESC
""")
for row in cur.fetchall():
    print(row)

# a YugabyteDB-specific function: list the nodes the smart driver actually
# discovered and is load-balancing connections across
cur.execute("SELECT * FROM yb_servers()")
for row in cur.fetchall():
    print(row)
```

A couple of things worth knowing before you use this in anything real. `psycopg2-yugabytedb` and plain `psycopg2` can't coexist in the same environment, pick one. `executemany()` here is fine for a demo but issues one round trip per row under the hood; for genuinely large batch loads, look at `execute_values()` from `psycopg2.extras` or COPY, the same advice that applies to plain Postgres. And `yb_servers()` is a good one to remember, it's how the smart driver itself discovers cluster topology, and it's a handy sanity check when you want to confirm your application is actually spreading load across nodes rather than hammering one.

## Go example

The equivalent for Go is `yugabyte/pgx`, a fork of the popular `jackc/pgx` driver with the same cluster-aware load balancing bolted on.

```bash
go mod init yugabyte-example
go get github.com/yugabyte/pgx/v5
```

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/rand"

    "github.com/yugabyte/pgx/v5"
)

func main() {
    ctx := context.Background()
    url := "postgres://yugabyte:yugabyte@127.0.0.1:5433/yugabyte?load_balance=true"

    conn, err := pgx.Connect(ctx, url)
    if err != nil {
        log.Fatalf("unable to connect: %v", err)
    }
    defer conn.Close(ctx)

    _, err = conn.Exec(ctx, `
        CREATE TABLE IF NOT EXISTS orders (
            order_id    UUID NOT NULL DEFAULT gen_random_uuid(),
            customer_id INT NOT NULL,
            status      TEXT NOT NULL DEFAULT 'pending',
            total_cents INT NOT NULL,
            created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
            PRIMARY KEY (order_id HASH)
        ) SPLIT INTO 12 TABLETS
    `)
    if err != nil {
        log.Fatalf("create table failed: %v", err)
    }

    statuses := []string{"pending", "paid", "shipped", "delivered", "cancelled"}

    const total = 20_000
    const chunk = 1_000

    for sent := 0; sent < total; sent += chunk {
        batch := &pgx.Batch{}
        for i := 0; i < chunk; i++ {
            batch.Queue(
                "INSERT INTO orders (customer_id, status, total_cents) VALUES ($1, $2, $3)",
                rand.Intn(5_000)+1,
                statuses[rand.Intn(len(statuses))],
                rand.Intn(49_501)+500,
            )
        }
        br := conn.SendBatch(ctx, batch)
        if err := br.Close(); err != nil {
            log.Fatalf("batch insert failed: %v", err)
        }
    }

    rows, err := conn.Query(ctx, `
        SELECT status, count(*) AS orders, round(avg(total_cents) / 100.0, 2) AS avg_order_usd
        FROM orders
        GROUP BY status
        ORDER BY orders DESC
    `)
    if err != nil {
        log.Fatalf("query failed: %v", err)
    }
    defer rows.Close()

    for rows.Next() {
        var status string
        var count int
        var avgOrderUSD float64
        if err := rows.Scan(&status, &count, &avgOrderUSD); err != nil {
            log.Fatalf("scan failed: %v", err)
        }
        fmt.Printf("%-10s %6d orders, avg $%.2f\n", status, count, avgOrderUSD)
    }
    if err := rows.Err(); err != nil {
        log.Fatalf("row iteration failed: %v", err)
    }
}
```

`pgx.Batch` pipelines the inserts over one connection instead of round-tripping per statement, which is the Go-idiomatic equivalent of the `PrepareBatch`/`Append`/`Send` pattern from the ClickHouse post's Go example: same underlying idea, batch client-side, ship once. I'm sending in chunks of a thousand rather than queueing all twenty thousand statements into a single batch, because the whole batch is buffered in memory on both ends and one enormous batch buys you nothing over a few reasonably sized ones. The `load_balance=true` query parameter is doing the same job the Python driver's `load_balance="true"` argument does: on first connect, the driver calls `yb_servers()` itself, caches the node list, and spreads subsequent connections across the cluster rather than hammering whichever node you happened to point the connection string at.

## A system design example: a multi-region deployment

Global distribution with strong consistency is YugabyteDB's headline feature, so it's worth sketching what that actually looks like rather than just asserting it exists.

{{< mermaid >}}
graph TB
subgraph US ["us-east region"]
UST["YB-TServer(s)<br/>tablet leaders for US-local data"]
end
subgraph EU ["eu-west region"]
EUT["YB-TServer(s)<br/>tablet leaders for EU-local data"]
end
subgraph APAC ["ap-south region"]
APT["YB-TServer(s)<br/>follower replicas"]
end
Master["YB-Master Raft group<br/>one member per region"]
Master -.-> UST
Master -.-> EUT
Master -.-> APT
USClient["US clients"] -->|topology_keys=us-east| UST
EUClient["EU clients"] -->|topology_keys=eu-west| EUT
UST <-->|Raft replication| EUT
UST <-->|Raft replication| APT
EUT <-->|Raft replication| APT
{{< /mermaid >}}

Two deployment patterns cover most real cases inside a single universe, and they're not mutually exclusive:

- **Synchronous, multi-region tables.** A table's tablets replicate synchronously (via Raft, as always) across three regions instead of three racks in one region. Every write has to reach a Raft quorum before it's acknowledged, so you're paying real cross-region round-trip latency on writes in exchange for the guarantee that losing an entire region doesn't lose data or availability. Use this only for data that genuinely needs that guarantee, like global user accounts or financial ledgers, and not for everything indiscriminately.
- **Geo-partitioning plus read replicas.** Tables (or table partitions) get pinned to a specific region via tablespaces, so EU customer data physically lives in `eu-west` and US data in `us-east`, which is directly useful for GDPR-style data residency requirements, echoing the Narvar case study above. Read replicas in other regions serve local, slightly-stale reads for global reporting or search use cases without paying the synchronous write penalty everywhere.

There is a third option that doesn't fit the "one universe stretched across regions" model at all, and it's worth naming because it comes up constantly in multi-region conversations: [xCluster](https://docs.yugabyte.com/stable/architecture/docdb-replication/async-replication/), YugabyteDB's asynchronous replication between two separate universes. Writes commit locally and ship to the other side afterwards, so you're trading strong consistency across regions for write latency that isn't hostage to the speed of light. It's the right tool for disaster recovery and for regulatory setups that need a full standby in another jurisdiction, and the wrong tool if you were expecting the same correctness guarantees Raft gives you inside a universe.

On the client side, the smart drivers support a `topology_keys` parameter (seen in both the Python and Go docs) that tells the driver to prefer connecting to nodes in a specific region, so a service running in `us-east` naturally talks to `us-east` tablet leaders instead of round-tripping to Europe for every query, while still failing over to another region automatically if its local one goes down.

## Production gotchas worth knowing before you get burned

- **Range-sharded tables with sequential keys create a hot tablet at start.** If you ask for range sharding on a primary key (`PRIMARY KEY (id ASC)`, typically because you want range scans over that column) the table starts as a single tablet and only splits as data arrives. Feed that a `SERIAL` or any other monotonically increasing value and every insert lands on the same tablet, on the same node, until it grows big enough to split; your brand new "distributed" table is, for a while, exactly as distributed as a single Postgres instance. Hash sharding is the default and doesn't have this problem, so the fix is usually to not reach for `ASC`/`DESC` unless you genuinely need ordering, and to `SPLIT INTO N TABLETS` upfront when you do and you already know roughly what your write volume looks like.
- **Not every PostgreSQL extension works.** DocDB replaces PostgreSQL's storage engine outright, so extensions that reach into storage internals rather than staying at the SQL and functions layer aren't guaranteed to be compatible. Yugabyte publishes the [list of supported extensions](https://docs.yugabyte.com/stable/additional-features/pg-extensions/), and it's short enough to read in a couple of minutes. Do that before promising anyone a drop-in swap.
- **Tablet count is a real sizing decision, not a knob to max out.** Every tablet replica costs you something. Yugabyte's [deployment checklist](https://docs.yugabyte.com/stable/deploy/checklist/) puts the raw overhead of a thousand tablet replicas on a node at roughly 0.4 vCPUs of Raft heartbeat traffic and 800 MiB of memory, and recommends budgeting around 7000 MiB per thousand replicas once you account for the caches that come with actually using them. That works out to a fairly concrete ceiling per node: their own table has an 8 GiB node supporting about 530 tablet replicas, and a 64 GiB node about 5500. Oversplitting on a small cluster burns that budget for nothing; undersplitting limits write parallelism. The defaults are a reasonable starting point, `SPLIT INTO` is for when you actually know better.
- **Synchronous multi-region replication has a latency floor set by geography, not by YugabyteDB.** No amount of tuning removes the physical round-trip time between `us-east` and `eu-west`. If a workload can't tolerate that on the write path, that's a signal to use geo-partitioning or async replication for that specific data, not a YugabyteDB performance problem to chase.
- **Use the built-in connection manager or a pooler for high connection counts**, the same operational lesson that applies to plain PostgreSQL. A large number of idle client connections carries real per-connection overhead on the server side, distributed database or not, and it competes for the same memory budget as your tablets.

## Wrapping up

The thing that makes YugabyteDB worth a serious look isn't any single benchmark number, it's the combination: real PostgreSQL compatibility (because it's genuinely built on PostgreSQL's own query layer, not a from-scratch reimplementation chasing compatibility), horizontal write scaling that doesn't require your team to learn a new data model, and automatic survival of node and region failures that would otherwise mean a 3 AM page and a manual failover runbook. None of that is free, you're running a distributed system with everything that implies, but for the class of application that's outgrown a single Postgres instance and doesn't want to trade away SQL and transactions to get there, it's a genuinely different answer than "shard it yourself" or "rewrite it on a NoSQL store."

And if you paired it with the [previous post]({{< ref "posts/2026-07-12-clickhouse-what-all-the-fuss-is-about.md" >}}): this is exactly the kind of system-of-record database you'd stream out of via CDC into something like ClickHouse for the analytical side, rather than ever pointing a dashboard directly at your transactional cluster. Two databases, each doing the one job it's actually built for, is usually a better architecture than one database straining to do both.

As always, if you've run YugabyteDB in production, multi-region or otherwise, I'd genuinely like to hear how it held up, what tripped you up, and whether the PostgreSQL compatibility promise held for your specific extension list. Drop a comment below or find me on [Twitter/X](https://twitter.com/jreypo).

–Juanma
