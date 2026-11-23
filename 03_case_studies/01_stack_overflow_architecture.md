# 🏛️ Stack Overflow — The Monolith That Scaled

> 📚 **Case study** — kept because it makes its point with *numbers*. Most of this tree teaches patterns. These three files are the evidence you give when an interviewer asks "why would you *not* do it the standard way?"

> ⚠️ **Date check: these figures are from 2016–2017.** Most come from Nick Craver's 2016 architecture
> write-ups. The 1.5 TB RAM figure comes from the SQL Server upgrade of early 2017.
> The "a third of the database in RAM" figure comes from a 2022 interview (Hanselminutes 847).
> They were correct then. Since then, Stack Overflow has moved to the cloud. Teams moved to Azure (2021–2023).
> The public sites moved to Google Cloud, and the company left its own data center in July 2025.
>
> - Quote them as *"as of their published 2016 architecture"*. Saying it this way also makes a good impression on the interviewer.
> - Do not state a 2016 number as a 2026 fact. That is the one real risk of using this file.
>
> **Why keep it:** it is the strongest answer we have to *"should this be microservices?"*. The numbers:
>
> - a few servers running at 5–10% of capacity
> - a third of the database kept in RAM
> - a 20 ms render on the busiest page
>
> Together they show, with numbers, that a well-run monolith does better than a distributed system when the
> workload fits on one machine. See also
> [`../02_distributed_systems/01_distributed_fundamentals.md`](../02_distributed_systems/01_distributed_fundamentals.md) §1 on what distribution really costs.

## 1. 📌 Metadata
- **Playlist URL:** https://www.youtube.com/playlist?list=PLCRMIe5FDPsd0gVs500xeOewfySTsmEjf
- **Video URL:** https://www.youtube.com/watch?v=fKc050dvNIE
- **Difficulty Level:** 🟡 Intermediate

---

## 2. 📖 What the Speaker Covers

- Stack Overflow's architecture, which is a monolith. This is surprising.
- Hardware specs (1.5TB RAM SQL Servers).
- No "trendy" tech (No K8s, No Microservices, No Cloud).
- Performance metrics (2 billion page views/month).

---

## 3. 🧾 Transcript-Based Notes

> Only include facts that the transcript states or clearly means.

### 🔹 The Anomaly

- **Expectation**: A site with 2 billion monthly views would use Microservices, Kubernetes, Cloud, Sharding, and Event Sourcing.
- **Reality**: Stack Overflow uses a **Monolithic Application** running on **On-Premise Servers**.
    - Just **9 Web Servers**. (The 2016 write-up lists 11: 9 "primary" servers plus 2 dev/meta servers.)
    - No Cloud. No Microservices.

### 🔹 Why it works

1.  **Vertical Scaling (Scale Up)**:
    - Instead of many small servers, they use a few powerful ones.
    - **SQL Server Instances**: They have **1.5 Terabytes of RAM**.
    - **In-Memory Data**: They keep about 1/3rd of their entire database in RAM. This gives instant access.
2.  **Optimization**:
    - The system is optimized for low latency and memory allocation.
    - Servers run at low capacity (5-10%), leaving a lot of spare capacity for traffic spikes.
3.  **Caching Strategy (or the lack of it)**:
    - The "Question Show" page (most visited) renders in about **20ms** (2016: 22.71 ms average). The whole page is *not* cached. It is built fresh on each visit.
    - Careful: the data behind the page is still cached. Each web server has an in-memory L1 cache, and Redis is the L2 cache. So the speed comes from the optimized monolith, the high-RAM DBs *and* these data caches.

### 🔹 The Lesson

- **Fit > Trend**: They designed an architecture that fits *their* specific problem (high read volume, text-heavy data). They did not just follow popular trends in the industry.
- **Simplicity**: A monolith is easier to manage if you build it efficiently.
- **Efficiency**: Running a few powerful on-premise servers reduces complexity / cost compared to a distributed cloud microservices mesh.

---

## 4. 💡 Speaker’s Key Takeaways

* ⭐ High scalability doesn't always require Microservices or the Cloud.
* ⭐ Very large vertical scaling (1.5TB RAM) can sometimes fix performance bottlenecks in a simpler way than distributed caching.
* ⭐ "The best architecture is the one that fits your needs, not the one that mimics tech giants."

---

## 5. ❓ Questions and Answers

> List the key questions the video answers, followed by a short answer based on the transcript.

1.  **How many web servers does Stack Overflow use?**
    - About 9 on-premise web servers.

2.  **Does Stack Overflow use Microservices?**
    - No, they use a monolithic architecture.

3.  **How do they achieve fast DB performance?**
    - By using a very large amount of RAM (1.5TB). This keeps a large part of the database directly in memory.

4.  **What is the main takeaway from their architecture?**
    - Understand your specific constraints and optimize for them. That is better than following trends like K8s or Microservices without thinking.
