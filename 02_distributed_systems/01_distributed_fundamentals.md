# Distributed Systems — The Fundamentals

> 🧭 **Foundations layer** &nbsp;·&nbsp; 🔥🔥 Very High Priority &nbsp;·&nbsp; 💬 **15 Questions**

---

📊 **Question Count**

| # | Section | Questions |
|---|---|---|
| 1 | [What a Distributed System Is](#1--what-a-distributed-system-is) | 2 |
| 2 | [The Four Core Challenges](#2--the-four-core-challenges) | 2 |
| 3 | [CAP and PACELC](#3--cap-and-pacelc) | 3 |
| 4 | [Consistency Models](#4--consistency-models) | 3 |
| 5 | [Consensus](#5--consensus) | 1 |
| 6 | [Clocks and Ordering](#6--clocks-and-ordering) | 3 |
| 7 | [Lock Safety and Fencing](#7--lock-safety-and-fencing) | 1 |
| | **Total** | **15** |

> 🔥🔥 Very high priority — asked in almost every Tier-1 interview loop &nbsp;·&nbsp; 🔥 High priority — very frequent in Tier-1 interviews &nbsp;·&nbsp; ⚡ Medium priority — commonly asked &nbsp;·&nbsp; 💤 Low priority — good to know, rarely asked

---

---

## 1. 🌐 What a Distributed System Is

It is a group of independent machines. They talk to each other **only by messages over a network**.
They work together so that a user sees them as one system.

It has four properties. Every hard problem below comes from one of them:

- **No shared memory** — every interaction is a network call that can fail
- **No global clock** — each node's clock drifts on its own
- **Independent failures** — one node dies while the rest keep running
- **Concurrency** — operations overlap across nodes with no total order

### Concurrency — why there is no total order

A **total order** is one list of all events that **every node agrees on**.
On one machine you get it for free: one database log, one lock, one clock decide what came first.
Across machines all three are gone, and a message sent first can arrive second.
So each node sees events in the order *they arrived at that node*.

```
Client 1 → Replica A:  X = 1
Client 2 → Replica B:  X = 2      (at about the same time; each replica forwards to the other)

Replica A applies:  X = 1 (local)  then  X = 2 (from B)  →  final X = 2
Replica B applies:  X = 2 (local)  then  X = 1 (from A)  →  final X = 1
```

Both replicas did the right thing in their own order, and now they disagree forever.

**"Concurrent" does not mean "same instant".** Two events are concurrent when **neither knew about the other** —
no message connected them. If client 2 read X = 1 before writing X = 2, the writes are ordered (a **causal** order).
If it did not, they are concurrent, even if they were 3 seconds apart.
So a distributed system knows only a **partial order**: the events linked by messages (§6).

To get a total order you must pay for coordination:

| Option | How it creates an order | Cost |
|---|---|---|
| **Single leader** | All writes go through one node, which writes one log | A bottleneck, and a single point of failure until failover |
| **Consensus** (Raft, §5) | The nodes vote; the Raft log *is* the order | Each write needs a majority, so it is slower |
| **Kafka partition** | Total order **inside one partition** only | No order across partitions — use one key (e.g. `orderId`) to keep related events together |
| **No order + conflict resolution** | LWW, vector clocks (§6), CRDTs | LWW silently loses writes; merging adds complexity |

AP systems such as Cassandra and DynamoDB skip the coordination and resolve conflicts afterwards.

### Why bother

| Problem | Monolith limit | What distribution buys |
|---|---|---|
| Scale | Vertical only — a bigger machine | Horizontal — add machines |
| Fault tolerance | One process, one failure domain | One node dies, the rest serve |
| Geography | One region, distant users pay latency | Deploy near users |

**The cost:** you swap a scale problem for a *correctness* problem.
Partial failure, consistency, ordering, and more things to operate are all new.
None of them existed in the monolith.
Say this clearly in the interview.
It shows you thought about the choice, and did not just copy microservices because others use them.

### ❓ Interview Questions

1. **What actually changes when you split a program across machines, and why is partial failure harder than total failure? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Three things stop being free:

- shared memory — every interaction becomes a network call that can fail
- a common clock — you lose a total order over events
- a single failure domain

Partial failure is harder than total failure because nobody makes the decision for you.

- In a monolith, a crash takes everything down. Every part agrees on the state, because everything is down.
- In a distributed system, the surviving half must *choose*: reject, queue, or degrade.
- It must choose without knowing if the other side is dead or only slow.

You cannot tell crashed from slow. That is the main cause of most distributed-systems complexity, including timeouts, heartbeats, leases, and fencing.

</details>

---

2. **What is the real cost of moving from a monolith to a distributed system? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

You swap a scaling problem for a correctness problem.

- In-process function calls could not fail. Now they are network calls that can fail, hang, duplicate, and arrive out of order.
- A single transaction gave you atomicity for free. Now it is a saga with hand-written compensations.
- A consistent read becomes a consistency *model* that you have to choose.
- Debugging one stack trace becomes linking traces across services.
- You get more things to operate: service discovery, deployment ordering, and schema compatibility across services that release independently.

Interviewers want to hear that this is worth it only for *specific* reasons, not by default. For example: independent scaling of an uneven workload, team autonomy at a certain headcount, or failure isolation.
Being able to argue *against* microservices for a given system is a stronger signal than arguing for them.
[`03_case_studies/01_stack_overflow_architecture.md`](../03_case_studies/01_stack_overflow_architecture.md) gives you the evidence for that argument.

</details>

---

## 2. ⚠️ The Four Core Challenges

### 2.1 The network is unreliable
Messages can be:

- **lost** (packet drop, timeout)
- **delayed** (congestion, a GC pause on the receiver)
- **duplicated** (your own retry)
- **reordered**

The result fits in one sentence. *"I sent it" never proves "it was received"*.
And a timeout never proves that the work did not happen.
This uncertainty is the reason idempotency keys exist.

### 2.2 Partial failure
In a monolith, the system is either up or down.
In a distributed system, *part* of it is down while the rest works.
The working part has to decide what to do.
This is the really hard one.
The answer is always a choice you make on purpose, not a default:

- **Reject** the request — honest, bad UX
- **Accept and queue** it for later — good UX, but now you own an async pipeline and its failure modes
- **Degrade** — circuit-break and serve a cached or default response

### 2.3 No global clock
Two nodes cannot agree on "what time is it" exactly enough to order events.
So a wall-clock timestamp is **not** proof that A happened before B — see §6.

<details>
<summary><strong>🤔 Doubt: How does a server keep time, why does its clock drift, and why can NTP not make two clocks exactly equal?</strong></summary>

**1. A physical part keeps the time.**

- Every motherboard has a small **quartz crystal**, the same part as in a wrist watch.
- With electricity applied, quartz **vibrates** at a fixed rate. A clock crystal is cut to vibrate **32,768 times per second**.
- A circuit counts the vibrations: 32,768 counts = 1 second. (32,768 = 2¹⁵, so a binary counter fills up exactly once per second.)
- This clock is the **RTC** (Real-Time Clock). A coin battery keeps it running when the machine is off.
- At boot the OS reads the RTC once. After that it keeps time with a fast CPU counter, which is also crystal-driven.

**2. Drift — every clock is wrong in its own way.**

- No crystal is perfect. Heat and age also change its rate.
- So "1 second" is counted a little early or late, every second, and the error adds up.
- Drift is measured in **ppm** (parts per million):

| Drift | Error per day |
|---|---|
| 20 ppm (normal server) | 20 × 10⁻⁶ × 86,400 s ≈ **1.7 s** |
| 200 ppm (the worst case Google plans for, cited in DDIA) | ≈ **17 s** |

- Each server drifts in its own direction. One can be 1 s fast and another 1 s slow — a 2 s gap. **No shared clock exists.**

**3. NTP (Network Time Protocol) corrects the drift, but not exactly.**

- A program on the server (`chronyd` or `systemd-timesyncd` on Linux) asks a more accurate time server every few minutes.
- The correct time comes in levels (**strata**): stratum 0 = atomic clocks and GPS → stratum 1 = servers wired to them → stratum 2, 3 … → your server. On AWS EC2 the source is the Amazon Time Sync Service at `169.254.169.123`.
- NTP uses 4 timestamps to estimate the network delay:

```
t1 = my clock, request sent           10:00:00.000
t2 = server clock, request arrives    10:00:00.060
t3 = server clock, reply sent         10:00:00.061
t4 = my clock, reply arrives          10:00:00.041

round trip  = (t4 − t1) − (t3 − t2)        = 41 − 1      = 40 ms
clock error = ((t2 − t1) + (t3 − t4)) / 2  = (60 + 20)/2 = 40 ms → my clock is 40 ms behind
```

**4. Why NTP is never exact — the key point.**

- NTP can measure the **round trip** (40 ms), but it **cannot see the two one-way delays**.
- It **assumes they are equal** (20 + 20).
- If the real split is 30 ms there and 10 ms back, the correction is wrong by **half the difference**: (30 − 10) / 2 = **10 ms**.
- Between syncs, the crystal keeps drifting, so the error starts to grow again.
- Result: about **1 ms** error inside one datacentre, **tens of ms** over the internet.

**5. NTP can move time backward.**

- Small error → NTP slowly speeds up or slows down the clock. Time never goes back.
- Large error → NTP **jumps** the clock. If the clock was ahead, time on that server moves backward.
- This is why you measure durations with `System.nanoTime()` (monotonic, one machine only), not `System.currentTimeMillis()` (wall clock, can jump).

**Say it in the interview:**

> "Every server counts the vibrations of its own quartz crystal, and each crystal drifts in its own direction. NTP corrects this, but it can only measure the round trip, not the one-way delays — it assumes they are equal. When they are not, the correction is off by half the difference, and the clock keeps drifting between syncs. So servers stay within about a millisecond, never exactly equal. That is why I never order events across nodes by wall-clock timestamps."

---

</details>
<br/>

### 2.4 Split brain
A network partition cuts the cluster in two.
Each half decides the other half is dead and elects itself leader.
Both halves accept writes. The data on the two sides becomes different, and you cannot merge it back.

**Prevention is quorum:** a write commits only after a majority (**⌊N/2⌋ + 1**) acknowledges it.
A partition can hold at most one majority, so at most one side can make progress.
Any two majorities of the same cluster **always share at least one node**, so two leaders cannot both win.
This is also why cluster sizes are odd.
5 nodes tolerate 2 failures, and 6 nodes also tolerate only 2.
So the sixth node adds nothing, and it adds coordination cost.

**The arithmetic** — failures tolerated = N − majority:

| Nodes (N) | Majority | Failures tolerated |
|---|---|---|
| 3 | 2 | **1** |
| 4 | 3 | **1** |
| 5 | 3 | **2** |
| 6 | 4 | **2** |
| 7 | 4 | **3** |

Adding a node to an odd cluster raises the majority too, so it tolerates nothing extra. And it makes things worse:

- **Every write waits for one more ack** — 4 of 6 instead of 3 of 5. The write is as slow as the slowest node in that group, so p99 latency rises.
- **An even split stops everything.** 6 nodes split 3 | 3 → neither side has 4 → no leader anywhere → writes stop.
  5 nodes split 3 | 2 → the side with 3 keeps working.
- **One more machine to pay for**, with no extra safety.

**Place the nodes across 3 availability zones.**
3 nodes = 1 per AZ; lose an AZ and 2 of 3 remain. 5 nodes = 2 + 2 + 1; lose any AZ and at least 3 of 5 remain.
With only **2** AZs, the bigger one holds the majority. If it fails, the cluster stops whatever the node count.
That is why a third AZ, or a small tiebreaker node in a third place, matters.

**Choosing a size:** **3** is the default (tolerates 1, lowest write latency).
**5** when you must survive one node in maintenance *plus* one unexpected failure. **7+** is rare.
Need more read capacity? Add **non-voting** members — ZooKeeper **observers**, etcd **learners**.
They copy the data and serve reads, but do not vote, so the majority does not grow.

### ❓ Interview Questions

1. **Why is a timeout the hardest failure to handle? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Because it does not tell you the one thing that matters. The request may have:

- never arrived,
- arrived and failed, or
- arrived and **fully succeeded**, with only the response lost.

A clean error tells you the work did not happen. A timeout tells you nothing.
This is why blind retry is unsafe on any non-idempotent operation. Retrying a payment after a timeout is how you double-charge.

The careful way to handle it:

- Make the operation idempotent with a client-supplied key, so a retry is a no-op.
- Or query status by that key before retrying.
- Never treat a timeout as proof that the work did not happen — in your logic or in your metrics.

</details>

---

2. **What is split brain and how does quorum prevent it? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

A partition divides the cluster. Each side cannot reach the other, decides the other is dead, and promotes its own leader.
Both sides accept writes. The two histories diverge, and there is no correct way to merge them afterwards.
That is split brain. It is a data-loss event, not an availability event.

Quorum prevents it by simple arithmetic: require ⌊N/2⌋ + 1 acknowledgements to commit.
A partition can produce at most one side with a majority. So at most one side can make progress.
The minority side refuses writes and stays consistent.
It works because any two majorities share at least one node, so two leaders cannot both collect votes.

This is also why cluster sizes are odd. With 5 nodes you survive 2 failures. A 6th node still only survives 2, and it adds coordination cost.
Worse, 6 nodes can split 3 | 3, and then neither side has a majority, so the whole cluster stops taking writes.
5 nodes always leave one side with 3.

In practice I run 3 or 5 voting nodes spread across 3 availability zones, so losing one AZ still leaves a majority.
If I need more read capacity, I add non-voting members (ZooKeeper observers, etcd learners) instead of more voters.

</details>

---

## 3. ⚖️ CAP and PACELC

You can guarantee only two of these three:

- **Consistency** — every read sees the latest write
- **Availability** — every request gets a non-error response
- **Partition tolerance** — the system keeps working while the network splits

**What the letters mean exactly** (from the formal proof by Gilbert and Lynch, 2002):

- **C is linearizability** — the system behaves as if there were one copy of the data. A read sees every write that finished before it began. It is **not** the "C" in ACID (that one means "your invariants hold", like "balance ≥ 0"). See §4.
- **A** means every request that reaches a **live** node gets a non-error answer. A node that is up but returns "try later" is not available in the CAP sense.
- **CAP talks only about the time during a partition.** When the network is healthy you can have both C and A. The trade-off then is latency vs consistency — that is PACELC below.

A concrete partition: the link between two datacentres drops for 40 seconds.
A user writes to DC-1 and then reads from DC-2. DC-2 can either refuse (CP) or return the old value (AP).
There is no third option.

**P is not a choice.** Partitions happen whether you plan for them or not.
So the real decision is what to do *during* one:

| | Behaviour during a partition | Examples | Use when |
|---|---|---|---|
| **CP** | Refuse or block rather than serve stale data | ZooKeeper, etcd, HBase, MongoDB with majority write concern | Money, inventory, seat counts — wrong data costs more than an error |
| **AP** | Answer from whatever the node knows, possibly stale | Cassandra, CouchDB, DynamoDB (default reads) | Feeds, catalogs, analytics — stale is survivable |

**The mistake to avoid:** CAP is a property of an *operation*, not of a system.
The same service usually needs both:

```
Loan disbursement    → CP   (double-disbursing is unrecoverable)
Loan status display  → AP   (a few seconds stale is invisible)
Credit score fetch   → AP   (a cached bureau score beats an outage)
```

The same split in retail — a limited sneaker launch:

```
Checkout (stock reserve)  → CP   (selling stock you don't have means cancelling paid orders)
Product page and price    → AP   (a few seconds stale is invisible)
"Only 3 left" badge       → AP   (an approximate count is fine; checkout checks again)
```

### PACELC — the half people forget

[Pronunciation](https://www.google.com/search?q=PACELC+pronunciation): _pass · elk_ (Daniel Abadi, who proposed it, says it this way)

CAP only describes the partition case. PACELC adds one more question.
**Else (E)**, when the network is healthy, do you choose **latency (L)** or **consistency (C)**?
You face that choice 99.9% of the time.

**Reading the name.** It is a sentence written as letters:

```
if P (Partition)      → choose A (Availability) or C (Consistency)    ← this is CAP
E  (Else, no partition) → choose L (Latency)    or C (Consistency)    ← the new half
```

So **PA/EL** means: "during a partition I choose availability; else, I choose low latency."

**Why the "else" half matters more.**

- A partition is rare — a few minutes a year in a good datacentre. CAP only gives you an answer for that case.
- The system is healthy almost all the time. In that state there is still a trade-off.
- You pay for that trade-off on **every single request**. CAP says nothing about it; PACELC does.

**Why latency and consistency fight even when the network is fine.** The cause is **replication**.
Your data lives on several nodes. When a write arrives, you have two choices:

- **EC (consistency):** wait until the replicas confirm the write, then answer "OK".
  Any replica you read from then has the new value. Cost: every write waits for the round trip to the replicas.
- **EL (latency):** write to one node, answer "OK" at once, and copy to the replicas in the background.
  Cost: for a short time, a read from another replica returns the **old** value.

The cost of waiting depends on the distance:

| Replica location | Round trip (approx.) | Effect of EC |
|---|---|---|
| Same AZ | < 1 ms | Almost free |
| Another AZ, same region | ~1–2 ms | Small |
| Another region | ~50–150 ms (Mumbai ↔ Frankfurt ~120 ms; far pairs like Mumbai ↔ US East ~200 ms) | Every write gets that much slower |

So **inside one region EC is cheap; across regions it is expensive.**
Reads have the same trade-off: read from the **leader** for fresh data (EC), or from the nearest **replica or cache** for speed (EL).

**The common classifications:**

| System | Class | What it means in practice |
|---|---|---|
| **DynamoDB** | PA/EL | Default reads are eventually consistent and fast. A strongly consistent read is possible, but it costs twice the read capacity. Global Tables copy between regions asynchronously. |
| **Classic single-leader RDBMS** (e.g. Postgres with a synchronous replica) | PC/EC | A commit waits for the replica. Always consistent, and you pay for it in latency on every write. |
| **MongoDB** | PA/EC | Abadi's standard example: normal reads go to the primary, so they are consistent. |
| **Cassandra** | PA/EL | Tunable per query: consistency level `ONE` is fast (EL), `QUORUM` is consistent (EC). |
| **Cosmos DB** | tunable | People usually cite it as PA/EL or simply "configurable". |
| PC/EL | rare | Abadi's example is Yahoo's PNUTS. Few systems pick it. |

**Tunable systems decide per request.** Cassandra, DynamoDB and Cosmos DB let you pick EL or EC for each read or write.
That is the same lesson as CAP above: **the choice belongs to the operation, not the system.**

**Using it in a design** — the retail launch again:

| Operation | Healthy-network choice | Why |
|---|---|---|
| Product page, price | **EL** — nearest replica or cache | A second of staleness is invisible, and fast pages sell |
| "Only 3 left" badge | **EL** | Only a display; checkout checks again |
| Checkout stock reserve | **EC** — read and write on the leader | Must not sell stock you don't have |
| Order history right after "Pay" | **EC for that user** — read-your-writes (§4) | The user must see the order they just placed |

The question PACELC makes you ask is: **"how stale may this read be when nothing is broken?"**
The answer decides whether that read can use a read replica or a cache.

Name PACELC without being asked.
It is an easy way to show that you know more than the basic CAP triangle.

### ❓ Interview Questions

1. **Why is P not really optional in CAP, and what does that leave you choosing between? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Partitions are not a design decision. They are a fact about networks: cables fail, switches reboot, a datacentre link becomes fully loaded.
If you do not tolerate partitions, your system is simply broken during one. That is strictly worse than either alternative.
So the real choice is what to do *while* partitioned:

- refuse service to keep correctness (CP), or
- answer with possibly stale data to keep availability (AP).

Be exact about the words. C in CAP means linearizability — every read sees the latest completed write — not the C in ACID.
A means every request to a live node gets a non-error answer.
And CAP is only about the partition window. When the network is healthy you get both, and the real trade-off is latency vs consistency, which is PACELC.

An example: the link between two datacentres drops. A user writes in DC-1, then reads in DC-2.
DC-2 must either refuse (CP) or return the old value (AP). "CA" is not on the menu, because the partition already happened.

The senior answer: you choose this **per operation**. The same service is CP on a money movement and AP on a status read.

</details>

---

2. **Give an example where the same service needs both CP and AP behaviour. 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

A loan platform.

- **Disbursement must be CP.** If the ledger is partitioned, failing the request is far cheaper than double-disbursing. Double-disbursing is unrecoverable and a regulatory problem.
- **Displaying loan status must be AP.** The user cannot notice a status that is a few seconds stale. But returning an error during a partition turns a short network glitch into a visible outage.
- **Fetching a bureau credit score is AP as well.** A cached score is acceptable. If you hard-fail when the bureau is down, your uptime depends on theirs.

A retail platform during a limited sneaker launch:

- **Checkout (the stock reserve) must be CP.** Thousands of buyers want the last few units. If two partitioned nodes both say "in stock", you sell the same pair twice. Then you must cancel a paid order and refund it. It is better for checkout to fail with "try again".
- **The product page and price must be AP.** A price or description that is a few seconds stale is invisible. An error page during a launch loses sales.
- **The "Only 3 left" badge is AP as well.** It is only a display, so an approximate count is fine. Checkout (CP) checks the stock again before it takes payment.

The retail case shows the key design idea: **an AP read can be wrong because a CP write checks again.**

Stating the decision per operation is the whole answer. If you say "we're a CP system", a follow-up question will break that claim.

</details>

---

3. **What is PACELC and why is the "else" half the one that matters day to day? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

CAP describes only the partition case, which is rare.
PACELC adds: **else**, when the network is fine, do you favour latency or consistency?
You face that choice almost all the time — on every request, not a few minutes a year.

The trade-off comes from **replication**:

- To be consistent (EC), a write waits for the replicas to confirm it. Inside one region that costs ~1–2 ms. Across regions it costs ~50–150 ms on every write.
- To be fast (EL), you answer at once and replicate in the background. Then a read from another replica may be stale for a short time.

Examples:

- DynamoDB is PA/EL (available under partition, low latency otherwise). A strongly consistent read is an opt-in, and it costs twice the read capacity.
- A single-leader RDBMS is PC/EC (always consistent, latency paid on every write).
- MongoDB is Abadi's PA/EC example.
- Cassandra is tunable per query: `ONE` is EL, `QUORUM` is EC.

The practical value: it forces the question "how stale may a read be under *normal* conditions".
That question decides whether you can use a read replica or a cache.
In a retail design, I serve the product page from a cache or nearby replica (EL), but run checkout on the leader (EC).

</details>

---

## 4. 🧊 Consistency Models

The table goes from strongest and most expensive to weakest and cheapest.

| Model | Guarantee | Mechanism | Reach for it when |
|---|---|---|---|
| **Strong** | Every read returns the latest write | Synchronous replication to all or a majority | Balances, seat bookings, inventory |
| **Causal** | If A caused B, everyone sees A before B; unrelated events may differ | Vector clocks / causal metadata | Chat — a reply must never appear before its parent |
| **Read-your-writes** | *You* always see your own write; others may lag | Route your reads to the replica that took your write, or pin the session | Profile edits — you change your name and must see it |
| **Monotonic reads** | Once you have seen a value, you never see an older one | Pin the session to a replica, or track a version | Dashboards — status must not flip approved → pending on refresh |
| **Eventual** | With no new writes, replicas converge — eventually | Asynchronous replication | Likes, view counts, activity feeds |

**The two in the middle are the interesting answer.**

- Most candidates know strong and eventual, and treat the choice as a binary.
- Read-your-writes and monotonic reads are *session* guarantees.
- They cost almost nothing (sticky routing).
- They remove the two anomalies that users really notice and report as bugs.
- Offering them changes your answer. You move from "eventual consistency is fine here" to
  "eventual across users, read-your-writes within a session."

**The anomaly, step by step** — why read-your-writes exists:

```
t=0 ms   User saves name "Alex K" → leader (writes succeed only here)
t=1 ms   Leader answers "OK"; replication to the follower starts (async)
t=5 ms   User's page reloads → load balancer sends the read to the follower
t=5 ms   Follower has not applied the write yet → returns the OLD name
t=40 ms  Follower catches up — too late, the user already saw the bug
```

**How read-your-writes is built in practice** (pick one):

- **Read from the leader for a short time after a write** — e.g. for 1 minute after this user's last update, send their reads to the leader. Simple, and only that user pays.
- **Send a version token.** The write returns its log position (an LSN or version). The client sends it with the next read. A replica answers only if it has applied at least that far. Otherwise it waits, or forwards the read to the leader. MongoDB's causally consistent sessions work this way.
- **Sticky routing** — pin the session to one replica. Cheapest, but it breaks when that replica fails.

Monotonic reads use the same tools: a sticky replica, or "never read from a replica older than the version you last saw".

**Quorums are the other way to tune this.** In leaderless stores (Cassandra, Dynamo-style), N replicas hold the data.
A write waits for W of them and a read asks R of them.
If **W + R > N**, the read set and the write set share at least one replica. So the read sees the latest write.
With N = 3, W = 2, R = 2 → 2 + 2 > 3 ✅. W = 1, R = 1 is fast but may read stale data.
Full treatment: [`vol_1/ch06`](../01_insider_guide_Alex_Xu/vol_1/ch06_design_a_key_value_store.md).

**Linearizability vs serializability — the names people mix up:**

| | Linearizability | Serializability |
|---|---|---|
| About | **One object** (one key, one register) | **Transactions** over many objects |
| Promise | A read sees the latest completed write — real-time recency | The result equals *some* one-at-a-time order of the transactions |
| Real time? | **Yes** — respects the order operations actually happened | **No** — that order may differ from the real-time order |
| Where you meet it | CAP's "C"; etcd, ZooKeeper writes | Database isolation level `SERIALIZABLE` |

Both together is **strict serializability**. Spanner offers it and calls it *external consistency*.

### ❓ Interview Questions

1. **Explain read-your-writes and monotonic-read consistency, and why they are worth asking for. 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Read-your-writes guarantees that after *you* write, *you* see your write. Other users may still see the old value.
Monotonic reads guarantee that once you have seen a value, you never later see an older one.
Both are **session** guarantees. Both are usually implemented in one of two ways:

- pin the session's reads to the replica that accepted its write, or
- carry a version that the read must meet.

They matter because they remove exactly the two anomalies that users see as bugs and file tickets about:

- "I saved my profile and it still shows the old name"
- "the status said approved, I refreshed, now it says pending"

They cost sticky routing, not synchronous replication.
So they are the cheapest large improvement you can make on an eventually consistent store.
If you know them, you do not have to pick only between strong and eventual.

</details>

---

2. **Which consistency model would you choose for a wallet balance, and what would you do about the read path? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Strong consistency on the write path, with no exceptions.
The system exists to get the balance right. An eventually consistent balance allows double-spend.
That means:

- a single leader per account (or a partition-level consensus group),
- synchronous commit,
- no reading the balance from an async replica for an authorisation decision.

The read path needs more thought.
Showing a balance in the app does not need the same guarantee as authorising against it. So a replica read is acceptable.
But it must be at least **read-your-writes and monotonic** for the session. If not, the user sees their own transfer missing, or sees the balance move backwards on refresh.

So: strong for authorisation, session-consistent for display. The two are explicitly different code paths.
Clearly stating that difference is the answer. Saying "strong consistency because it's money" and stopping is only half of it.

</details>

---

3. **What is the difference between linearizability and serializability? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

They answer different questions.

- **Linearizability** is about **one object**. Every read sees the latest completed write, as if there were one copy of the data. It respects **real time**: if write A finished before read B started, B must see A. This is the "C" in CAP.
- **Serializability** is about **transactions** that touch many objects. The result must equal *some* order in which the transactions ran one at a time. That order does **not** have to match real time.

An example of the gap: transaction T1 commits, then T2 starts a moment later.
A serializable database may still order T2 *before* T1 and return T2 a value from before T1 — that is legal.
A linearizable system could not do that.

Both together is **strict serializability**. Spanner provides it and calls it external consistency.

In a design: a lock service or leader election needs linearizability (etcd, ZooKeeper).
A money transfer across two accounts needs serializable transactions.
Mixing up the two words is a common mistake, so I name both and say which one the problem needs.

</details>

---

## 5. 🗳️ Consensus

[Pronunciation](https://www.google.com/search?q=consensus+pronunciation): _kuhn · sen · suhs_

**The problem:** several nodes must agree on one value.
For example: who the leader is, or what the committed log entry is.
They must agree while messages are lost and nodes crash.

### Raft — the one to know
It was designed to be easy to understand.
etcd (and so Kubernetes), Consul, and CockroachDB use it.

**Three roles.** Every node is in exactly one role:

| Role | What it does |
|---|---|
| **Leader** | Takes all writes and replicates them. Sends a **heartbeat** ("I'm alive") to every follower about every 50 ms. |
| **Follower** | Passive. Copies the leader's log. Each heartbeat resets its **election timer**. |
| **Candidate** | A follower whose election timer ran out with no heartbeat. It now asks the others for votes. |

**Leader election:** a follower that hears no heartbeat becomes a candidate and requests votes.

1. It increments the **term** (an election counter, e.g. 4 → 5) and votes for itself.
2. It sends **RequestVote** to every other node.
3. Each node gives **one vote per term**, to the first candidate that asks. It refuses a candidate whose log is
   less up to date than its own. So a node that missed recent writes cannot become leader.
4. The first candidate to win a **majority** becomes leader. Its heartbeats reset the other timers, so nobody
   else starts an election.

Randomised election timeouts stop every node from becoming a candidate at the same time.
Without them, the vote could split forever:

```
Same timeout (200 ms), 5 nodes, leader dies:
  all 4 followers time out together → each votes for itself → 1 vote each, nobody gets 3
  → all time out together again → repeats forever, no leader

Random timeout (150–300 ms):
  B times out at 162 ms, the others at 207, 241, 289 ms
  B asks for votes; C, D, E have not voted in this term → B gets 4 votes in ~1 ms → leader
```

If two candidates still tie, both pick a new random timeout and try again in the next term.

**Timing rule:** network round trip (~1 ms) ≪ heartbeat (~50 ms) < election timeout (150–300 ms).
The trade-off: a longer timeout means fewer false elections, but writes stop for longer after the leader dies.
If the old leader comes back, it sees a higher term and steps down. This is the same idea as a fencing token (§7).

**Log replication:** client → leader appends locally → replicates to followers → once a **majority**
acknowledges, the entry is committed → the leader tells followers to apply it.
The majority rule is the same quorum idea as §2.4.
That is why Raft's design makes split brain impossible.

### The other two, in one line each
- **Paxos** — older, correct, and well known for being hard to implement (Multi-Paxos, Fast Paxos). Google Chubby.
  It is enough to know it exists, and that Raft is the easier alternative.
- **ZAB** — ZooKeeper Atomic Broadcast. It is like Raft, but built around *ordered broadcast*.
  That is why ZooKeeper can offer leader election, config distribution, locks and service discovery on one primitive.

### ❓ Interview Questions

1. **Walk me through Raft leader election and log replication. 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Every follower runs an election timer. The leader's heartbeat resets it.
If the timer expires, the follower:

1. increments the term,
2. becomes a candidate,
3. votes for itself and requests votes from the rest.

The first candidate to collect a **majority** becomes leader and starts sending heartbeats.
Election timeouts are randomised per node. Without that, all followers time out together and split the vote. Then the cluster can fail to elect a leader forever.

For writes:

1. The client sends to the leader.
2. The leader appends the entry to its own log and replicates it to followers.
3. Once a majority has acknowledged, the leader marks it committed and applies it.
4. The leader tells followers to apply it too.

Commit requires a majority, and leadership requires a majority. So at most one leader wins each term. When a newer leader is elected, a majority has already moved to its newer term. So the old leader cannot commit anything more.
That is exactly the split-brain guarantee from §2 Q2.

</details>

---

## 6. ⏱️ Clocks and Ordering

Server A logs "order created 10:00:00.001". Server B logs "payment charged 10:00:00.000".
Did the payment come before the order? **You cannot tell.** B's clock may simply be 5 ms slow.
Wall-clock timestamps do not order events across machines.

**Lamport timestamps** — one counter per node.

- On send: increment the counter and attach it.
- On receive: `counter = max(local, received) + 1`.

It guarantees *if A caused B then ts(A) < ts(B)*.
The reverse does **not** hold: `ts(A) < ts(B)` may just mean the two were concurrent.
This one-way rule is the usual follow-up question.

```
Node A:  a1 (local event)      → counter 1
         a2 (send m to B)      → counter 2, attach 2
Node B:  b1 (local event)      → counter 1
         b2 (receive m)        → max(1, 2) + 1 = 3

a2 → b2 is causal (a message links them) and ts 2 < 3 ✅ — the guarantee holds.
b1 = 1 < a2 = 2, but b1 did NOT cause a2 — no message links them. They are concurrent.
So a smaller timestamp does not prove "happened before".
```

To build a **total order** (for tie-breaking), sort by `(counter, nodeId)`: equal counters are broken by node id.
The order is consistent with causality. For concurrent events, the order has no meaning — it only breaks the tie.

**Vector clocks** — a vector of counters, one slot per node.
Now you can tell *causally ordered* events apart from *concurrent* ones.
If neither vector dominates the other, the events are concurrent.
Then you have a real conflict to resolve.

```
Node A: [A:1, B:0, C:0]  → sends to B
Node B: [A:1, B:1, C:0]  → sends to C
Node C: [A:1, B:1, C:1]

Each vector ≤ the next ⇒ causally ordered (A → B → C)

Node A later: [A:2, B:0, C:0]  vs  Node C: [A:1, B:1, C:1]
Incomparable vectors (neither ≤ the other) ⇒ concurrent ⇒ conflict
```

> ⚠️ **Used by Riak and by the original Amazon *Dynamo paper*. DynamoDB the service does not use
> vector clocks.** It resolves conflicts with last-writer-wins plus conditional writes.
> So you design around LWW instead of merging siblings.
> Mixing up the paper and the product is a common and costly mistake.

**Hybrid Logical Clocks (HLC)** — a physical timestamp with a logical counter added at the end.
It is close enough to wall time to be human-readable and to compare across nodes.
But it is also monotonic and causally correct.
CockroachDB uses it. This is the modern practical answer.

How it stays correct: each node keeps `(time, counter)`.
On every event or received message, `time` becomes the largest of the local clock and any timestamp it has seen.
If `time` did not move forward, `counter` goes up by 1.
So a receiver's timestamp is always above the sender's (causality), even when the receiver's clock is behind.
And it never runs far from real time, because it follows the physical clock whenever that clock is ahead.

**Google TrueTime** — atomic clocks plus GPS in every datacentre.
It exposes time as a bounded *interval* `[earliest, latest]`, not as a single point.
Spanner waits until the uncertainty window has passed before it commits.
That is how it offers externally consistent global transactions.
The honest way to say it: Spanner didn't solve the clock problem.
It *put a limit* on the error, and the cost is extra commit latency.

### ❓ Interview Questions

1. **Why can't you order events across services with timestamps? What do you use instead? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Because clocks drift.
NTP keeps servers within milliseconds to tens of milliseconds of each other.
That is much larger than the gap between events in a busy system.
So an event that really happened second can carry the earlier timestamp.
Wall-clock time is a *hint*, never proof of ordering.

Use these instead:

- **Lamport timestamps** give causal ordering with one counter per node (`counter = max(local, received) + 1` on receive).
- **Vector clocks** also let you detect *concurrency*. If neither vector dominates, the events are truly concurrent, and there is a real conflict to resolve.
- **Hybrid Logical Clocks** are what modern systems use in practice. They add a logical counter to a physical timestamp. So the value is both human-comparable and causally correct. CockroachDB is the best-known user.

</details>

---

2. **What does Lamport ordering guarantee, and what does it not? ⚡**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

It guarantees one direction only: if A causally comes before B, then `ts(A) < ts(B)`.
It does **not** guarantee the reverse.
Seeing `ts(A) < ts(B)` tells you nothing about whether A caused B.
The reason: two fully independent events on different nodes will also get ordered timestamps.

- Lamport timestamps are safe for *setting* an order, to break ties deterministically.
- They are unsafe for *inferring* causality.

An example: node A sends a message at counter 2, and node B receives it at counter 3. That pair is causal, and 2 < 3.
But an unrelated local event on B also got counter 1, and 1 < 2 — yet it did not cause A's send.
The timestamps alone cannot tell these two cases apart.

To make a total order, sort by `(counter, nodeId)`. That is consistent with causality. For concurrent events, the order only breaks the tie and means nothing.

To detect real concurrency you need vector clocks. With vector clocks, two vectors that cannot be compared show concurrency.
The cost: a vector has one slot per node, so it grows with the cluster. A Lamport timestamp is a single number.

</details>

---

3. **How does Google Spanner get globally consistent transactions when clocks drift? ⚡**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

TrueTime.
Google puts atomic clocks and GPS receivers in every datacentre.
It exposes time as a bounded *interval* `[earliest, latest]`, not a point. The interval is typically a few milliseconds wide.
A transaction that commits at timestamp T then **waits until the uncertainty window has passed** before acknowledging.
So when the client hears that it committed, T is clearly in the past for every other node.

The honest description: Spanner did not solve clock drift.
It put a hard limit on the error, and the cost of that limit is extra commit latency.
It is worth naming because it is the best-known production system that makes wall-clock time trustworthy. And it shows what that costs.

</details>

---

## 7. 🔒 Lock Safety and Fencing

A distributed lock does **not** make a critical section safe on its own. This is the trap.

1. Client A acquires the lock, then stalls (a long GC pause, a hypervisor freeze).
2. The lease expires.
3. Client B correctly acquires the lock and starts work.
4. Client A wakes up. It still believes it holds the lock, and it writes.

**Two writers, and no error raised anywhere.**

The fix is a **fencing token**:

- The lock service returns a monotonically increasing number with every grant.
- The client passes this number to the storage layer on every write.
- Storage **rejects any write carrying a token lower than the highest it has already seen**.

```
A acquires lock → token 33 → (stalls)
B acquires lock → token 34 → writes with 34 → storage accepts, now at 34
A wakes         → writes with 33 → storage REJECTS (33 < 34)
```

The point worth stating: fencing moves the safety guarantee from the *lock service* to the
*resource*.
Without it, no lock protocol (Redlock included) can protect against a paused client.
The reason: from the outside, a pause looks exactly like a crash.

**Where the token comes from.** It must be a number that only goes up and that the lock service owns:

- **ZooKeeper** — the `zxid` (transaction id) or the znode version of the lock node.
- **etcd** — the revision of the lock key. Every change in etcd gets a higher revision.
- **A database** — a counter column that the lock grant increments in the same transaction.

**How storage checks it** — a conditional write. The resource stores the highest token it has seen:

```sql
UPDATE account
SET    balance = ?, fence_token = 34
WHERE  id = ? AND fence_token < 34;
-- 0 rows updated → a newer holder already wrote → reject this write
```

This only works if the storage **can** run such a check. Some storage cannot do it, such as a plain file store or an external API
with no version field. Then the lock alone gives you *efficiency* (less duplicate work), not *correctness*.

**The Redlock debate, in one line.** Martin Kleppmann argued (2016) that Redlock is unsafe for correctness:
it depends on timing assumptions and produces no fencing token. Redis's author (antirez) replied that the
timing assumptions are acceptable in practice. The safe position for an interview:
use a Redis lock for efficiency. For correctness, use a consensus-based lock (etcd / ZooKeeper) plus fencing tokens.

**A real-world scenario — the nightly invoice job.**
An e-commerce service runs on 2 pods. At 2 AM a job generates invoices and emails them, and it must run on **one pod only**.
Each pod tries to take a Redis lock with a 30-second lease: `SET lock:invoice-job pod-A NX PX 30000`.

```
02:00:00  Pod A takes the lock (lease 30 s) and starts the job
02:00:05  Pod A hits a long GC pause (or the node is CPU-throttled) — it freezes for 40 s
02:00:30  The lease expires. Pod B takes the lock and starts the same job
02:00:45  Pod A wakes up. It still thinks it holds the lock and continues
          → both pods generate invoices → customers get two emails, two invoice numbers
```

No error appears anywhere. Each pod followed the lock rules correctly.

The fix: the lock grant returns a token that only goes up — Pod A gets **33**, Pod B gets **34**.
Every write carries the token, and the table keeps the highest one it has seen:

```sql
UPDATE invoice_run SET status = 'DONE', fence_token = 34
WHERE  run_date = '2026-10-23' AND fence_token < 34;
-- Pod A later writes with 33 → 0 rows updated → rejected
```

**One more catch:** the email goes to an outside system, which cannot check a token.
So the send is also made **idempotent** — one email per `(run_date, invoice_id)`, recorded before it is sent.
The fencing token protects your database. The idempotency key protects the outside world.

**Real systems that use fencing tokens:**

| System | Its "fencing token" | What it protects against |
|---|---|---|
| **Google Chubby** (Google's lock service) | A **sequencer** — the lock name plus a lock generation number. Servers check it before they accept a request. | The paper describes this exact problem: a delayed client that still thinks it holds the lock |
| **HDFS NameNode HA** | An **epoch number** in the Quorum Journal Manager. A new active NameNode gets a higher epoch, and the journal nodes reject writes with a lower one. | Two NameNodes that both think they are active and both write the file-system log (split brain) |
| **Kafka controller** | The **controller epoch**. Brokers ignore commands that carry an older epoch. | A "zombie" controller that was paused, lost its role, and comes back with old commands |
| **Kafka transactional producer** | The **producer epoch**. A new instance with the same `transactional.id` raises the epoch, and the broker rejects the old instance's writes. | Two copies of the same producer writing after a restart or failover |
| **HBase** (a real incident, cited in Kleppmann's DDIA) | None at the time — that is the lesson | A region server paused in GC, lost its lease, and kept writing after another server took over. The data was corrupted. |

Kafka is the easiest one to name from your own stack: "Kafka fences a zombie controller with the controller epoch, and a zombie producer with the producer epoch."

### ❓ Interview Questions

1. **A client holds a distributed lock and its process pauses for 30 seconds. What goes wrong, and what fixes it? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

The lease expires while the client is paused. The cause can be a long GC pause, a VM freeze, or a stop-the-world in a container being throttled.

1. A second client correctly acquires the lock and starts work.
2. The first client wakes up. It still believes it holds the lock, and it writes.
3. Now two writers are in the critical section, and nothing anywhere raises an error.

No lock protocol fixes this on its own, Redlock included. From the outside, a paused process looks exactly like a crashed one.

The fix is a **fencing token**:

- The lock service returns a monotonically increasing number with each grant.
- The client passes it on every write.
- The *storage layer* rejects any write whose token is lower than the highest it has seen.

That moves the safety guarantee from the lock service to the resource. The resource is the only place where it can really be enforced.

In practice the token is the etcd revision or the ZooKeeper `zxid`, and the storage checks it with a conditional write — `UPDATE … WHERE fence_token < ?`.
If the storage cannot check a token, the lock only saves duplicate work; it does not make the result correct.

</details>

---

## 🔢 Numbers to Remember

- Quorum for N nodes: **⌊N/2⌋ + 1**. 3 nodes tolerate 1 failure, 5 tolerate 2. Even cluster sizes add nothing
- Typical NTP clock drift between servers: **single-digit to tens of milliseconds** — enough to reverse event order at high throughput
- Raft election timeout: **150–300 ms** in the Raft paper, randomised per node to avoid split votes. etcd's default is 1,000 ms (heartbeat 100 ms)
- Spanner's TrueTime uncertainty window: **~1–7 ms**, and commits wait it out
- A cross-region round trip is **~50–150 ms** — with synchronous cross-region replication, you pay that on every write

---

## 🗣️ Interview Cheat Sheet

| If they ask… | Hit these |
|---|---|
| "What is a distributed system?" | Multiple nodes, messages only, no shared memory, no global clock, independent failure, appears as one system |
| "CAP theorem?" | 2 of 3, P is mandatory, so the real choice is CP vs AP — **per operation, not per system**. Then give a concrete pair from your own work |
| "Eventual consistency?" | Converges once writes stop; faster writes, temporarily stale reads. Then volunteer read-your-writes as the cheap session-level upgrade |
| "How do nodes agree on a leader?" | Quorum + Raft: heartbeat timeout → candidate → majority vote. Majority is also what prevents split brain |
| "Can you order events across services?" | Not by wall clock. Lamport for causal order, vector clocks to detect concurrency, HLC in practice |
| "Is a Redis lock enough?" | No — a paused client looks identical to a crashed one. Fencing tokens, checked at the resource |
| "Why not just use 2PC?" | Blocking, coordinator is a SPOF, holds locks across network calls. Saga instead — see `java/17` |

---
