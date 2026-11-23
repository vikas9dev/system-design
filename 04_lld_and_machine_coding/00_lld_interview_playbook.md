# 🎯 LLD & Machine Coding — The Playbook

> 🧭 **Method file, not a problem** &nbsp;·&nbsp; 🔥🔥 Very High Priority &nbsp;·&nbsp; 💬 **12 Questions**

> 🥡 **One-liner:** *"The round is not won by knowing more patterns. It is won by asking two
> questions in the first two minutes, finding the one or two seams that matter, and finishing
> something that runs."*

---

📊 **Question Count**

| # | Section | Questions |
|---|---|---|
| 1 | Which round am I in? | — |
| 2 | The six phases | — |
| 3 | Clarifying with committed defaults | — |
| 4 | Finding the seams | — |
| 5 | The concurrency cookbook | — |
| 6 | The JUnit recipe | — |
| 7 | What loses the round | — |
| 8 | Interview Q&A | 12 |
| | **Total** | **12** |

> 🔥🔥 Very high priority — asked in almost every Tier-1 interview loop &nbsp;·&nbsp; 🔥 High priority — very frequent in Tier-1 interviews &nbsp;·&nbsp; ⚡ Medium priority — commonly asked &nbsp;·&nbsp; 💤 Low priority — good to know, rarely asked

---

> **How to use this file.** Read §1–§4 before your first timed rep (one timed practice attempt).
> Then **come back and rewrite §7 with your own failure modes**.
> The general version below comes from the problems in this folder.
> A list of the three things *you* actually ran out of time on is always better.
> Learn by heart the version you write from your own reps.

---

## 1. 🧭 Which round am I in?

The phrase "LLD interview" can mean two completely different rounds.
The single most useful thing you can do is **find out which one within the first two minutes.**

| | 🛠 Machine coding | 💬 LLD discussion |
|---|---|---|
| Length | 90 min – 2.5 h | 45–90 min |
| Deliverable | **Code that compiles and runs** | Class diagram + signatures + reasoning |
| Graded on | Does it work? Is it extensible? Did you finish? | Modelling judgement, trade-offs, vocabulary |
| Tests | Often the differentiator | Rarely written, sometimes discussed |
| Fatal mistake | Spending 40 minutes on a diagram | Diving into implementation detail |

**How to ask, without sounding like you are wasting time:**

> *"Before I start — is the deliverable running code, or a design I walk you through? And should I
> optimise for breadth across features or depth on the core flow?"*

That question is itself graded. It shows you have done this before.
And the answer changes everything about how you spend the next two hours.

**In this folder the file number tells you the answer.**

- `01–09` are machine-coding grade. You could really be asked to build these.
- `50+` are discussion grade.
- Chess is `50_` because you cannot build it in a machine-coding slot.
  If you practise it as if you could, you waste the session.

---

## 2. ⏱️ The six phases

The phases use percentages of the total time.
So the same plan works for a 90-minute screen and a 2.5-hour onsite.
The table shows the percentages first, then the two common round lengths.

| # | Phase | % | 90 min | 2.5 h | Done when |
|---|---|---|---|---|---|
| 1 | **Clarify** — questions with committed defaults | 8% | 7 min | 12 min | You have written down the scope you are building |
| 2 | **Model** — entities, cardinality, one diagram | 12% | 11 min | 18 min | Every noun in the requirements is a class or explicitly rejected |
| 3 | **Skeleton** — interfaces, enums, empty methods | 15% | 14 min | 22 min | It compiles. Nothing is implemented. |
| 4 | **Core flow** — the happy path, end to end | 35% | 31 min | 53 min | A `main` shows the primary use case working |
| 5 | **Concurrency + edges** — the race, the guards | 20% | 18 min | 30 min | The obvious race is closed and you can name it |
| 6 | **Tests + walkthrough** | 10% | 9 min | 15 min | 3+ tests pass; you have narrated the design |

**Two rules that matter more than the numbers:**

- **Phase 3 ends with something that compiles.** At about the one-third mark, an empty skeleton that builds
  is worth more than a rich design that does not.
  The reason: every later phase then only has to keep it building.
  If you have never compiled by minute 40 of a 2.5-hour round, you are in trouble.
  You just cannot see it yet.
- **Take phase 5's time from phase 4, not from phase 6.** The natural habit is to keep adding
  features and skip tests.
  But extra features are the cheapest thing to show, and they get the fewest marks.
  The race condition and the tests are what separate candidates.

> 💡 **Announce the plan at minute 2.** *"I'll clarify for ~10, model for ~15, get a compiling
> skeleton by ~45, then core flow, then concurrency, and leave 15 for tests."*
> If an interviewer knows your plan, they will correct you early when it is wrong.
> That correction is free information.
> You only get it if you say the plan to them.

---

## 3. ❓ Clarifying with committed defaults

The most common failure is this: you ask open questions, and the interviewer hands the decision back to you.

**❌️ The weak form:** *"Should we support multiple currencies?"*
Now the interviewer decides. You have spent time.
And you learned nothing about their priorities.

**✅ The strong form — state the assumption and invite correction:**

> *"I'll assume single currency and integer minor units, since multi-currency is an FX-at-what-time
> question that would take up too much time. Say if you want it in scope."*

Why this works:

- It is faster.
- It shows judgement.
- The useful information is the interviewer's *objection*, not their answer.

The newer machine-coding files here (01, 02, 03, 08, 09, 10) have a clarifying-questions table with committed defaults.
This is exactly the reason.
Read the table before the rep.
Then you remember choices you already made, instead of inventing them under time pressure.

**The four questions worth asking in almost every problem:**

1. **Scale** — one machine, in-memory? (Almost always yes. Ask it, so you can *stop* worrying about scale.)
2. **Persistence** — in-memory collections, or a real store? (Almost always in-memory. Say so.)
3. **Concurrency** — single-threaded, or must it be thread-safe? (**Assume it must.** See §5.)
4. **The one domain ambiguity** — the thing, specific to this problem, that changes the model.
   Examples:
   - Seat vs ShowSeat.
   - Does the payer take part in the split?
   - Does a lift serve calls above it while it goes down?

   Find it and ask it.

---

## 4. 🧩 Finding the seams

Patterns are not the point. **Seams** are the point.
A seam is a place where a future requirement should arrive as a *new class*, not as an *edit*.
The interviewer's follow-ups almost always test a seam:
*"now add surge pricing"*, *"now add a new vehicle type"*, *"now change how we pick a spot"*.

**The simple rule, in one question:** *what would the product manager change next quarter?*
That is where the interface goes.

| Smell in the requirements | Seam | Pattern |
|---|---|---|
| A list of *kinds* of the same thing (EQUAL / EXACT / PERCENT) | Behaviour varies by kind | **Strategy** |
| A word like *policy*, *rule*, *how we choose* | Selection algorithm | **Strategy** |
| An object whose legal operations depend on its status | Lifecycle | **State** |
| "…and then notify / update the board / send an email" | Fan-out on an event | **Observer** |
| Creating one of several subtypes from input | Construction | **Factory** |
| A sequence of steps where one step varies | Algorithm skeleton | **Template Method** |
| "Check A, then B, then C, stop at the first match" | Ordered handlers | **Chain of Responsibility** |

> ⚠️ **Two seams is usually the right number.** Parking Lot wants pluggable *assignment* and
> pluggable *pricing*. That is the whole design.
> Adding five patterns to a small problem is a well-known bad sign.
> It looks like you copy patterns from a textbook instead of reading the requirements.
> Snake & Ladder needs about one.
> If you cannot name the requirement a pattern protects against, remove the pattern.

**The test for whether a seam is real:** say the future requirement to the interviewer and follow it through the design.
*"Weekend surge pricing"* → new `PricingStrategy` implementation, zero edits elsewhere.
If the trace forces you to open `ParkingLot.java`, the seam is fake.
Then you should move the interface.

Full pattern reference:
[`../../java/2_design_patterns/01_LLD_Interview_Patterns_Guide.md`](https://github.com/vikas9dev/java/blob/main/2_design_patterns/01_LLD_Interview_Patterns_Guide.md).

---

## 5. ⚡ The concurrency cookbook

> ⚠️ **A correction, because one file in this folder used to say the opposite:** concurrency **is**
> asked at SDE-2.
> At your target companies it is usually the deciding follow-up.
> Expect "two users, one resource" to come up every time.
> When the resource is scarce and shared, that race *is* the problem.
> The CRUD code around it is only support code.

Four races cover almost every problem in this folder.

### 5.1 The last free resource — atomic claim

Two threads read "spot 7 is free" and both take it.

```java
// ❌️ check-then-act — a race, even with a ConcurrentHashMap
if (spot.isFree()) { spot.occupy(vehicle); }

// ✅ the claim itself is the atomic operation
class ParkingSpot {
    private final AtomicReference<Vehicle> occupant = new AtomicReference<>();
    boolean tryClaim(Vehicle v) { return occupant.compareAndSet(null, v); }
    void release()               { occupant.set(null); }
}
```

> **The sentence that gets you the point:** *"a concurrent collection makes each individual operation
> atomic, not a compound check-then-act — so `ConcurrentHashMap` alone does not fix this."*

### 5.2 The held resource — lock with expiry

A seat held for payment must not stay stuck when the user closes the tab.
The lock needs an **owner** and an **expiry**.
Every read must treat an expired lock as absent.

```java
record SeatLock(String userId, Instant expiresAt) {
    boolean isLive(Instant now) { return now.isBefore(expiresAt); }
}
```

Lazy expiry (check on read) is usually enough.
It has no background thread to reason about.
A `ScheduledExecutorService` sweeper (a background task that clears expired locks) is the extra safety step. It frees inventory quickly.
The full worked version is in [`08_bookmyshow_seat_booking.md`](./08_bookmyshow_seat_booking.md) §8.

### 5.3 Multiple resources — deterministic lock ordering

One thread locks seats 4 and 5, while another thread locks 5 and 4. This deadlocks.
**Sort the ids and always acquire in the same order.**
This is the same fix as the ascending-account-id rule for transfers in
[`../01_insider_guide_Alex_Xu/vol_2/ch12_digital_wallet.md`](../01_insider_guide_Alex_Xu/vol_2/ch12_digital_wallet.md).
Naming that connection is a strong signal.

```java
List<Seat> ordered = seats.stream()
        .sorted(Comparator.comparing(Seat::getId))
        .toList();   // now every thread acquires in the same global order
```

If you get only some of the locks, you must **roll back**. Release what you took and fail the whole request. All or nothing.

### 5.4 Read-mostly state — the right lock granularity

Do not `synchronized` the whole service.
Lock the smallest unit that owns the invariant: the spot, the seat, the account.
Then unrelated operations run in parallel.
If everything goes through one monitor (one lock), your program is correct, but it runs like a single-threaded program with extra steps.
An interviewer will say so.

---

## 6. 🧪 The JUnit recipe

**No file in this folder had a single `@Test` before 2026-09-01.**
In a machine-coding round, two candidates may both finish.
Tests are often what separates them.
Three tests written in the last fifteen minutes are worth more than a fourth feature.

**The three tests to write, in priority order:**

```java
// 1. HAPPY PATH — the primary use case end to end
@Test
void parksAndChargesCorrectly() {
    Ticket t = lot.park(new Car("KA-01-1234"));
    assertNotNull(t.getSpot());
    assertEquals(new BigDecimal("40.00"), lot.unpark(t, t.getEntryTime().plusHours(2)));
}

// 2. THE RACE — this is the one that earns the round
@Test
void onlyOneThreadClaimsTheLastSpot() throws Exception {
    ParkingLot lot = lotWithExactlyOneFreeSpot();
    int threads = 16;
    CountDownLatch start = new CountDownLatch(1);
    AtomicInteger wins = new AtomicInteger();
    ExecutorService pool = Executors.newFixedThreadPool(threads);

    for (int i = 0; i < threads; i++) {
        int id = i;                              // a lambda needs an effectively final copy
        pool.submit(() -> {
            start.await();                       // release all threads together
            if (lot.park(new Car("X" + id)) != null) wins.incrementAndGet();
            return null;
        });
    }
    start.countDown();                            // fire
    pool.shutdown();
    assertTrue(pool.awaitTermination(5, TimeUnit.SECONDS));
    assertEquals(1, wins.get());                  // exactly one winner
}

// 3. THE INVARIANT — the property that must hold whatever the inputs
@Test
void balancesAlwaysSumToZero() {
    group.addExpense(...);
    assertEquals(0, group.netBalances().values().stream()
            .reduce(BigDecimal.ZERO, BigDecimal::add).compareTo(BigDecimal.ZERO));
}
```

**Why the `CountDownLatch`:** without it, sixteen threads submitted in a loop mostly run
*sequentially*.
The first one finishes before the last one is scheduled.
So the test passes on a broken implementation.
The latch makes the threads compete at the same moment.
Knowing that is the point of the test.
Say it to the interviewer while you write it.

**Time control.** Never `Thread.sleep()` to test an expiry. It makes the suite slow and flaky.
Inject a `Clock` (or a `Supplier<Instant>`), so a test can move time forward instantly.

---

## 7. 💀 What loses the round

*Rewrite this section after your first three timed reps, with what actually went wrong for you.
Until then, these are the failures you can see in the problems collected here.*

| Failure | Why it costs you | The fix |
|---|---|---|
| **Nothing runs at the end** | The one unambiguous negative in a machine-coding round | Compile at phase 3 and keep it compiling |
| **40 minutes on the diagram** | You spend the time on an ungraded artefact instead of the graded one | Time-box the diagram; it exists to guide *your* code |
| **Referencing classes you never wrote** | The interviewer runs it and it does not build | `throw new UnsupportedOperationException(...)` at a boundary you consciously skipped — that is a *positive* signal |
| **Pattern soup** | Looks like textbook recall, not requirement analysis | Two seams. Name the requirement each defends against |
| **Silent working** | They are grading your reasoning. Silence gives them nothing to grade | Narrate decisions and their alternatives |
| **Ignoring concurrency** | It is the senior follow-up, and it will come | §5. Assume thread-safety is required |
| **Anaemic model** | Data classes plus one god service is procedural code that only looks like OOP | Behaviour belongs on the object that owns the state |
| **`double` for money** | A balance sheet that must sum to zero, and does not | `BigDecimal` or integer minor units |
| **Not finishing the last 10 minutes** | No tests, no walkthrough, no summary | Keep them free and protect them |

---

## 8. 🗺️ The problems, and what each is for

| File | Band | The one thing it teaches |
|---|---|---|
| [`01_parking_lot.md`](./01_parking_lot.md) | 🛠 | Two Strategy seams + atomic claim on the last spot |
| [`02_vending_machine.md`](./02_vending_machine.md) | 🛠 | State machine done properly |
| [`03_elevator_system.md`](./03_elevator_system.md) | 🛠 | Scheduling/dispatch policy, and a livelock that is easy to write |
| [`04_inventory_locker_system.md`](./04_inventory_locker_system.md) | 🛠 | Strategy for allocation; size compatibility |
| [`05_rate_limiter.md`](./05_rate_limiter.md) | 🛠 | Sliding window + Factory; amortised O(1) |
| [`06_process_logger.md`](./06_process_logger.md) | 🛠 | Ordered-queue modelling; a timestamp is not a unique key |
| [`07_lru_cache.md`](./07_lru_cache.md) | 🛠 | HashMap + doubly linked list. Code fixed on 2026-09-01; it now compiles |
| [`08_bookmyshow_seat_booking.md`](./08_bookmyshow_seat_booking.md) | 🛠 | **Lock with expiry** — the only problem here that forces it |
| [`09_splitwise.md`](./09_splitwise.md) | 🛠 | **An algorithm**, not just modelling — debt simplification |
| [`10_snake_and_ladder.md`](./10_snake_and_ladder.md) | 🛠 | Restraint. The 90-minute warm-up |
| [`50_chess_modelling.md`](./50_chess_modelling.md) | 💬 | Inheritance vs composition. Code fixed on 2026-09-01; it now compiles |
| [`51_flight_booking_modelling.md`](./51_flight_booking_modelling.md) | 💬 | Instance modelling — Flight vs Aircraft, seat ownership |
| [`99_tic_tac_toe.md`](./99_tic_tac_toe.md) | 💬 | Win detection that checks only the lines the move touches — O(N) per move |

---

## ❓ Interview Q&A

1. **How do you decide whether a round is machine-coding or LLD discussion, and why does it matter? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

You ask, in the first two minutes: *"is the deliverable running code, or a design I walk you through?"*

It matters because the two rounds reward opposite behaviour:

- **Machine coding:** the diagram is only a tool. The running program is the thing they grade. So forty minutes of modelling is forty minutes not spent on the graded output.
- **Discussion round:** the opposite is true. If you go into implementation detail, you leave no time for the trade-off reasoning they are grading.

If you guess wrong, you lose the round, even if your work is good.
And the question itself shows experience, not uncertainty.

</details>

---

2. **Walk me through how you budget a 2.5-hour machine-coding round. 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Roughly:

- 12 minutes clarifying with committed defaults
- 18 modelling
- 22 to a **compiling skeleton**
- 53 on the core happy path end to end
- 30 on concurrency and edge cases
- 15 on tests and a walkthrough

Two rules matter more than the split:

1. **Something must compile by around the one-third mark.** An empty skeleton that builds beats a rich design that does not. The reason: everything after it stays buildable.
2. **The concurrency time comes out of feature work, not out of tests.** Extra features are the cheapest thing to show, and they get the fewest marks. The race condition and the tests are what really separate candidates.

I also tell the interviewer my plan at minute two.
If they disagree, they will correct me early.
That correction is free information.

</details>

---

3. **What is wrong with asking "should we support multiple currencies?" 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

It hands the decision back to the interviewer, and you learn nothing.
The stronger form states a position and asks them to correct it:
*"I'll assume single currency and integer minor units, since multi-currency turns into an FX-at-what-time question that would take up too much time — say if you want it in scope."*

Why this is better:

- It is faster.
- It shows judgement about what is worth building.
- The useful information is the interviewer's **objection**, not their answer.

Committed defaults also protect your time.
An open question can start a five-minute discussion you did not plan for.

</details>

---

4. **How do you decide where an interface belongs? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

I ask what a product manager would change next quarter.
That is exactly what the interviewer's follow-ups test: *"now add surge pricing"*, *"now change how we pick a spot"*.
Those are requests for a **seam**.
A seam is a place where a new requirement arrives as a new class, not as an edit.

To test whether a seam is real, I follow the future requirement through the design while I talk:

- If "weekend surge pricing" is a new `PricingStrategy` implementation and nothing else changes, the seam is right.
- If it forces me to open the `ParkingLot` class, the interface is in the wrong place.

</details>

---

5. **Why is two usually the right number of design patterns? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Because patterns have a cost as well as a benefit.
Each one adds indirection (one more layer between the caller and the code). A reader has to keep it in mind.
It is only worth that cost if it protects against a requirement that will actually change.

Parking Lot has exactly two things a product manager would likely change: assignment policy and pricing.
So it has two Strategy seams, and that is the whole design.

Using five patterns on a small problem is a well-known bad sign.
It looks like you remember a textbook, instead of reading the requirements.
My rule: if I cannot name the requirement a pattern protects against, I remove the pattern.

</details>

---

6. **Two threads take the last parking spot. Where exactly is the race, and how do you close it? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

The race is between the check and the act.
Both threads see `spot.isFree()` return true before either one takes the spot.
So both go ahead.

The fix: make the claim itself the atomic operation, instead of guarding a read followed by a write.
Put an `AtomicReference<Vehicle>` on the spot and use `compareAndSet(null, vehicle)`.
Exactly one caller can win.

Say this clearly: **a concurrent collection does not fix this**.
`ConcurrentHashMap` makes each individual operation atomic.
It does nothing for a compound check-then-act across two operations.
Candidates who use a concurrent map and stop there have usually not understood where the race is.

</details>

---

7. **Why does a seat lock need an expiry, and what does that change? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Because the common case is not a crash.
It is a user who opens the payment page and closes the tab.
Without an expiry, that seat stays stuck forever, and inventory slowly disappears.

So the lock carries an owner and an `expiresAt`.
Every read treats an expired lock as absent.

- Lazy expiry on read is usually enough. It adds no background thread that you must think about.
- A scheduled sweeper is the extra safety step. It frees inventory quickly, instead of on the next access.

The expiry changes the confirm path (the step that books the seat).
Before booking, you must check that the lock is still held **by this user** and is **still live**.
A simple `if (seat.isLocked())` check brings back a check-then-act race.
It also lets a *different* user's lock pass the check.

This is also the one problem in this set that maps directly to distributed systems.
The same lock in Redis is `SET key val NX PX 600000`.
There it is unsafe without a fencing token (a number that increases with each new lock holder). The reason: a paused client looks exactly like a crashed one.

</details>

---

8. **Two users each lock two of the same four seats. What happens, and what is the fix? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Deadlock.
Each user holds what the other needs, and neither can go ahead.

The fix is **deterministic lock ordering**.
Sort the seat ids and always acquire in the same global order.
Then a cycle cannot form.

If you get only some of the locks, you must roll back.
Release everything taken and fail the whole request, because a multi-seat booking is all-or-nothing.

Name the general rule too.
It is the same fix as locking two accounts in ascending id order before a wallet transfer.
Any time you take more than one lock, a consistent global ordering is what prevents the cycle.

</details>

---

9. **What would you test if you had fifteen minutes left? 🔥🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Three tests, in this order:

1. **The happy path end to end.** This shows the primary use case works.
2. **The race.** This is the one most likely to win the round. Use sixteen threads on an `ExecutorService`, released together by a `CountDownLatch`, and assert exactly one winner.
3. **An invariant** that must hold whatever the inputs. For example, net balances summing to zero.

The `CountDownLatch` detail matters. Explain it while you write it.
Without it, threads submitted in a loop mostly run sequentially.
The first one finishes before the last one is scheduled.
So the test passes against a broken implementation.
The latch is what makes the threads really compete.

I would also inject a `Clock` instead of using `Thread.sleep()` to test expiry.
Then the suite stays fast and non-flaky.

</details>

---

10. **What single thing most reliably loses a machine-coding round? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Handing over something that does not run.
For everything else, the interviewer can judge how bad it is.
But a build failure is clear to everyone.

The two usual causes:

- Spending too long on a diagram before writing any code.
- Referencing classes that were never written. For example, a factory that creates two types the file does not define.

The second one is easy to avoid. And the fix itself counts in your favour.
At a boundary you have consciously decided not to build, write `throw new UnsupportedOperationException(...)`.
That shows you know where the seam is, and that you manage your time on purpose.
A phantom class (a class you use but never wrote) shows neither.
It fails the moment the interviewer compiles.

</details>

---

11. **What is an anaemic domain model, and why is it penalised here? 🔥**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

It is a design where the classes are little more than fields with getters and setters.
All the behaviour lives in one large service that reads and changes their fields.
It is procedural code that only looks like OOP.

It loses marks because the round is there to test object modelling.
If every decision ends up in a god service (one class that does everything), no modelling has really happened.

The clear sign is where the invariant lives.
Say `ParkingSpot` exposes `setOccupant()`, and the service checks first whether the spot is free.
Then the invariant is outside the object that owns it.
Every new caller is a chance to break it.
Putting `tryClaim()` on the spot puts the rule where the state is.
That is also exactly what makes the concurrency fix in §5 possible.

</details>

---

12. **Why not use `double` for money? ⚡**

<!-- -->
<details>
<summary>💡 Show Answer</summary>

**Answer:**

Binary floating point cannot represent most decimal fractions exactly.
`0.1 + 0.2` is not `0.3`.
So errors add up across many operations.

In this system, correctness means balances sum to exactly zero.
So the error is not a small rounding problem. It is a broken invariant.
It only shows up after many transactions, which makes it hard to trace.

Use `BigDecimal` with an explicit scale and rounding mode, or integer minor units (paise).
And choose the rounding rule on purpose.
Splitting ₹100 three ways means someone takes the extra paisa.
So pick who gets it with a fixed rule (for example, the payer), and tell the interviewer.
It is a good sign when you raise that detail yourself.
It shows you have thought about the case where the arithmetic does not divide evenly.

</details>

---
