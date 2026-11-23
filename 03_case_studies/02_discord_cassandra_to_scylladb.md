# 🎮 Discord — Cassandra → ScyllaDB

> 📚 **Case study** — kept because it makes its case with real *numbers*. Most of this tree teaches patterns. Quote these three files as evidence when an interviewer asks "why would you *not* do it the standard way?"

> **Why keep it:** it is a datastore migration. *JVM garbage-collection pauses* were one
> stated cause. Hot partitions and compaction backlog were the others. The cluster went from 177 nodes down to 72 after the move to a C++ reimplementation
> of the same data model. For a Java candidate, explaining a migration partly caused by GC behaviour
> is a much better answer than repeating Cassandra's write path from memory.

## 1. 📌 Metadata
- **Playlist URL:** https://www.youtube.com/playlist?list=PLCRMIe5FDPsd0gVs500xeOewfySTsmEjf
- **Video URL:** https://www.youtube.com/watch?v=O3PwuzCvAjI
- **Difficulty Level:** 🔴 Advanced

---

## 2. 📖 What the Speaker Covers

- Discord's very large migration of trillions of messages from Cassandra to ScyllaDB.
- Problems with the existing Cassandra cluster (Latency, GC pauses).
- The middle "Data Services" layer, written in Rust.
- The "Super-Disk" solution (RAID0 NVMe + RAID1 Persistent Disk).
- How they ran the migration with zero downtime.

---

## 3. 🧾 Transcript-Based Notes

> Only include information explicitly stated or clearly implied in the transcript.

### 🔹 The Problem with Cassandra

- **Scale**: By 2022, Discord held trillions of messages across 177 nodes.
- **Issues**:
    - Unpredictable latency.
    - Frequent on-call incidents.
    - Performance issues caused by Garbage Collector (GC) pauses (Cassandra is Java-based).
- **Criticality**: If the message cluster is down, Discord is down.

### 🔹 The Solution: ScyllaDB

- **Why ScyllaDB?**:
    - Cassandra-compatible.
    - Written in C++ (No Garbage Collection pauses).
    - Promised better performance and faster repairs.

### 🔹 Migration Strategy

1.  **Test with Small Databases**: They migrated smaller services first. This helped them learn and fix issues early.
    - *Philosophy*: Move fast on decisions you can undo. Be careful on decisions you cannot undo ("one-way doors").
2.  **Data Services Layer**:
    - They built a middle layer between the API and the DB, using **Rust**.
    - **Request Coalescing**: If many users request the same data at the same time, the DB is queried only once. The other requests wait for that single in-flight result.
        - It cuts *read amplification* on hot partitions: many readers of the same data no longer each send their own query.
        - It does **not** prevent hot partitions. The data distribution decides which key is hot. Coalescing does not change that.
        - It only stops N concurrent readers from becoming N queries.
        - This is the same idea as cache-stampede protection / thundering-herd suppression.
3.  **The "Super-Disk"**:
    - **Problem**: Google Cloud Persistent Disks were reliable but had high latency. Local NVMe SSDs were fast but ephemeral (data can be lost if the host fails or the VM is stopped).
    - **Solution**: A software RAID setup.
        - **RAID 0** (Stripe) across Local NVMe SSDs for **Speed**.
        - **RAID 1** (Mirror) that combines the RAID 0 array with a Persistent Disk for **Durability**.
    - **Result**: Low latency reads (from SSDs) and durable writes (to Persistent Disk).

### 🔹 The Payoff

- **Speed**: Migrated trillions of messages in 9 days with **zero downtime**.
- **Efficiency**: Reduced cluster size from 177 Cassandra nodes to 72 ScyllaDB nodes.
- **Reliability**: Predictable (deterministic) latency and a happier on-call team.

---

## 4. 💡 Speaker’s Key Takeaways

* ⭐ Use middle layers (like Rust Data Services) to protect the database from traffic spikes (Request Coalescing).
* ⭐ Solve hardware limits with new software ideas (The Super-Disk RAID combination).
* ⭐ "Move fast on reversible decisions, slow on irreversible ones."

---

## 5. ❓ Questions and Answers

> List the key questions the video answers, followed by a concise answer based on the transcript.

1.  **Why did Discord migrate away from Cassandra?**
    - Because of unpredictable latency. The causes were hot partitions, compaction falling behind, and Java Garbage Collection pauses at their very large scale.

2.  **What is "Request Coalescing"?**
    - A technique that groups many requests for the same data, sent at the same time, into a single database query. This reduces load.

3.  **How did Discord solve the Disk Latency vs. Durability trade-off?**
    - They created a "Super-Disk". It uses RAID1 to mirror two things: a fast but ephemeral RAID0 NVMe array, and a slow but persistent network disk.

4.  **Why did Discord choose Rust for the middle layer?**
    - For its safety and high performance. Also, it does not have the GC issues of other languages.
