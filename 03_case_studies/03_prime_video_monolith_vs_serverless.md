# 📺 Prime Video — Serverless Back to a Monolith

> 📚 **Case study** — kept because it makes its point with *numbers*. Most of this folder teaches patterns. These three files are the evidence. You cite them when an interviewer asks "why would you *not* do it the standard way?"

> ⚠️ **Give the correct scope. If you do not, this example hurts your answer.** This was the **Video Quality Analysis** team's
> internal monitoring tool. It was *not* Prime Video's streaming architecture. People very often
> overstate it as "Amazon abandoned serverless". Being the candidate who gives the scope correctly is
> worth more than the example itself.
>
> **Why keep it:** the bottleneck was **Step Functions state transitions (billed per transition, with several
> transitions for every second of each stream), and S3 used as a hop between components**. It was not compute cost. These exact details make it an
> argument, not just a story. Serverless cost too much here because the *orchestration* was
> chatty (many small calls between steps). You can look for this property in your own designs.

## 1. 📌 Metadata
- **Playlist URL:** https://www.youtube.com/playlist?list=PLCRMIe5FDPsd0gVs500xeOewfySTsmEjf
- **Video URL:** https://www.youtube.com/watch?v=JTp0TY_2hXM
- **Difficulty Level:** 🟡 Intermediate

---

## 2. 📖 What the Speaker Covers

- Case Study: Amazon Prime Video moving from Serverless to Monolith.
- Results: 90% cost reduction.
- Technical Bottlenecks: Step Functions state transitions and S3 intermediate storage.
- The "Serverless First" philosophy vs "Serverless Only".

---

## 3. 🧾 Transcript-Based Notes

> Only include information explicitly stated or clearly implied in the transcript.

### 🔹 The Problem

- **Context**: Prime Video monitoring tool (Defect Detector) for thousands of live streams.
- **Initial Architecture**: Serverless (AWS Step Functions + Lambda).
- **Issues**:
    1.  **Cost**: Expensive state transitions in Step Functions (several for every second of each stream).
    2.  **S3 requests**: Passing video frames between components via S3 caused a very high number of costly S3 calls.
- **Scaling Bottlenecks**: Hit hard limits on account throughput.

---

### 🔹 The Solution

- **Action**: Refactored into a **Monolithic Architecture**.
- **Deployment**: Media Converter and Defect Detector deployed in the same process.
- **Benefits**:
    - Removed S3 intermediate storage (in-memory transfer).
    - Removed expensive Step Function transitions.
- **Result**: **90% Cost Savings**.

---

### 🔹 The Lesson

- **Serverless First**: It was the right choice for *prototyping* and moving fast at the start.
- **Evolution**: Architectures must change over time. "Monolith vs Microservices" should not be a fixed belief.
- **Quotes**:
    - Werner Vogels: "Building evolvable software systems is a strategy, not a religion."
    - Adrian Cockcroft: Advocated "Serverless First", not "Serverless Only".

---

## 4. 💡 Speaker’s Key Takeaways

* ⭐ Microservices/Serverless are not always the answer. High-throughput data transfer between loosely coupled components can be costly.
* ⭐ The change showed that *refactoring* can work well. It was not a judgment against serverless.
* ⭐ Use Serverless to build fast. Then optimize or move to a monolith if scale or cost requires it.

---

## 5. ❓ Questions and Answers

> List the key questions the video answers, followed by a short answer based on the transcript.

> 🔥🔥 Very high priority — asked in almost every Tier-1 interview loop &nbsp;·&nbsp; 🔥 High priority — very frequent in Tier-1 interviews &nbsp;·&nbsp; ⚡ Medium priority — commonly asked &nbsp;·&nbsp; 💤 Low priority — good to know, rarely asked

1. **Why did Prime Video move away from Serverless for this tool? 🔥**
   - High costs from AWS Step Function state transitions and S3 request calls, for their high-volume workload.

2. **What was the outcome of the migration to Monolith? 🔥**
   - A 90% reduction in infrastructure costs.

3. **Does this mean Serverless is bad? 🔥🔥**
   - No. It supports "Serverless First" for prototyping.
   - But high-scale workloads may later need optimized, combined architectures.

4. **What key bottleneck was removed by combining components? 🔥**
   - The need to use S3 buckets for intermediate storage between distributed components.
