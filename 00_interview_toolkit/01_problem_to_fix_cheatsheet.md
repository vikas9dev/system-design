# 🧰 Problem → Fix — The Pre-Round Cheat Sheet

Every fix here is explained in depth somewhere else. The last column links to that note.
**If this file starts to grow explanations, move them into the linked note instead.**

---

## 1. How to say a line in the round

A fix on its own sounds junior. Say three things: **problem → fix → cost.**

> "Writes are spiking, so I put Kafka in front of the DB to absorb the bursts.
> The cost: the client no longer gets a synchronous result, so I add a status endpoint.
> And if the *steady* write rate is above consumer capacity, the queue only delays the failure."

Before any fix, say **how you found the bottleneck** (metrics, `EXPLAIN`, a trace). Every line
below assumes you already know where the problem is.

---

## 2. The 12 lines — with their cost

| # | Problem | First fix | Its cost | Do not use it when… | Deep note |
|---|---|---|---|---|---|
| 1 | Slow reads | Add a cache | Stale data; you must pick an invalidation strategy (TTL, cache-aside, write-through) | The data must always be fresh, or the real cause is a bad query (fix the query first) | [Vol 1 ch01 §4](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) · [Redis IQ](../iq/03_Redis_Interview_Questions.md) |
| 2 | Write spikes | Buffer with a queue | Client gets "accepted", not "done". A queue **smooths** load; it adds **no** capacity | The *average* write rate is above what consumers can handle — the queue just grows | [Vol 2 ch04](../01_insider_guide_Alex_Xu/vol_2/ch04_distributed_message_queue.md) · [Vol 1 ch01 §7](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) |
| 3 | Too much traffic | Load balancer | The LB alone fixes nothing. The real fix is **more stateless servers**; the LB spreads traffic across them. Sessions must move out (Redis, JWT) | Servers hold session state — fix that first | [Vol 1 ch01 §2, §6](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) · [Load balancing](../02_distributed_systems/02_load_balancing_strategies.md) |
| 4 | Read-heavy DB | Read replicas | Replication lag breaks read-your-own-writes | A read must see the write just made — send it to the primary | [Vol 1 ch01 §3](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) · [Consistency models](../02_distributed_systems/01_distributed_fundamentals.md) (§4) |
| 5 | Write-heavy DB | Shard the data | Cross-shard joins and transactions, hot keys, painful resharding | You have not yet tried: batching writes, a bigger machine, a queue, or a write-optimised store (Cassandra, DynamoDB). **Sharding is the last step** | [Vol 1 ch01 §8](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) · [DB internals §3](../iq/02_Database_Design_and_Internals_Interview_Questions.md) |
| 6 | Slow queries | Add indexes | Every index makes writes slower and uses storage | `EXPLAIN` shows another cause: N+1 queries, a function on an indexed column, a bad join | [DB internals §1](../iq/02_Database_Design_and_Internals_Interview_Questions.md) |
| 7 | Slow file loads | Object storage + CDN | CDN cost; cache invalidation when a file changes | The content is dynamic or per-user | [Vol 1 ch01 §5](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) · [Vol 2 ch09](../01_insider_guide_Alex_Xu/vol_2/ch09_s3_like_object_storage.md) |
| 8 | Temporary failures | Retry with backoff | Without **jitter** all clients retry at the same moment. Without a **retry limit** you get a retry storm | The operation is **not idempotent** — a retry can charge twice | [Vol 2 ch11 §8](../01_insider_guide_Alex_Xu/vol_2/ch11_payment_system.md) · [Microservices §5](https://github.com/vikas9dev/java/blob/main/1_interview_questions/17_Spring_Microservices_Interview_Questions.md) |
| 9 | Too many requests | Rate limiting | Real users can be blocked; the counter store (Redis) becomes a dependency | — say *where* (gateway or per service) and *which algorithm* (token bucket, sliding window) | [Vol 1 ch04](../01_insider_guide_Alex_Xu/vol_1/ch04_design_a_rate_limiter.md) · [LLD rate limiter](../04_lld_and_machine_coding/05_rate_limiter.md) |
| 10 | One service failing | Circuit breaker | Needs a fallback; thresholds need tuning | There is **no timeout** — threads hang before the breaker ever trips. Timeout first, then breaker + bulkhead | [Microservices §5](https://github.com/vikas9dev/java/blob/main/1_interview_questions/17_Spring_Microservices_Interview_Questions.md) |
| 11 | Duplicate requests | Idempotency keys | Extra storage; the key and the result must be saved **atomically** (unique constraint) | — on a repeat, return the **stored** result, do not run the work again | [Vol 2 ch11 §8](../01_insider_guide_Alex_Xu/vol_2/ch11_payment_system.md) · [Vol 2 ch12](../01_insider_guide_Alex_Xu/vol_2/ch12_digital_wallet.md) |
| 12 | Can't see what broke | Logs, metrics, tracing | Storage cost; high-cardinality labels can break a metrics store | — always pass a **trace ID** across every service | [Vol 2 ch05](../01_insider_guide_Alex_Xu/vol_2/ch05_metrics_monitoring_alerting.md) · [Microservices §7](https://github.com/vikas9dev/java/blob/main/1_interview_questions/17_Spring_Microservices_Interview_Questions.md) · [Prod debugging](https://github.com/vikas9dev/java/blob/main/1_interview_questions/19_Production_Debugging_Interview_Questions.md) |

---

## 3. The 5 lines the list leaves out

| # | Problem | First fix | Its cost | Deep note |
|---|---|---|---|---|
| 13 | Single point of failure | Redundancy — replicas, multi-AZ, automatic failover | More cost; failover needs health checks, and split-brain needs fencing | [Vol 1 ch01 §7](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) · [Fencing](../02_distributed_systems/01_distributed_fundamentals.md) (§7) |
| 14 | Slow dependency | **Timeouts** on every remote call | Too short = false failures; too long = threads used up | [Microservices §5](https://github.com/vikas9dev/java/blob/main/1_interview_questions/17_Spring_Microservices_Interview_Questions.md) · [Prod debugging §4](https://github.com/vikas9dev/java/blob/main/1_interview_questions/19_Production_Debugging_Interview_Questions.md) |
| 15 | Hot key / hot partition | Split the key (add a suffix), give it its own shard, or a local cache in front | Reads must merge the split parts | [Vol 1 ch01 §8](../01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) (celebrity problem) |
| 16 | Long task inside the request | Async: return `202 Accepted`, a worker does the job | Client must poll or get a callback; failures need a retry path and a DLQ | [Vol 2 ch04](../01_insider_guide_Alex_Xu/vol_2/ch04_distributed_message_queue.md) |
| 17 | You don't know the bottleneck | **Measure first** — metrics, `EXPLAIN`, a trace | Time | [Prod debugging §1](https://github.com/vikas9dev/java/blob/main/1_interview_questions/19_Production_Debugging_Interview_Questions.md) |

---

## 4. Order matters — the usual pairs

- **Timeout → retry (with jitter) → circuit breaker.** A breaker without a timeout never trips. A retry without idempotency is a bug.
- **Index → cache → read replica.** Fix the query before you hide it.
- **Batch / queue / bigger machine → shard.** Sharding is hard to undo.
- **Retry ↔ idempotency key.** Every retry you add needs one.

