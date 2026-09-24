# Scale-Out Paradigm & Distributed Storage

---

## 1. Administrative Overview & Course Logistics

- **Lecturer:** **Prof. Gabriele Tolomei** (Associate Professor at Sapienza University of Rome, Lead at HERCOLE Lab, Co-founder & Chief Science Officer at _tellmewAI_).
- **Relocation Notice:** Prof. Tolomei noted in class that he is relocating at the end of the semester. Early exam sessions (January/February) will run as normal, while arrangements for later sessions will be communicated.
- **Class Schedule:**
    - **Wednesdays:** 8:30–11:00 (Room 1L).
    - **Thursdays:** 10:00–12:00 (Room 2L).
- **Assessment & Grading:**
    - Determined through an **oral seminar** on a recent research paper from top-tier venues (e.g., SIGMOD, VLDB, OSDI, SOSP, NeurIPS, ICML, KDD) chosen by the student and approved by the instructor.
    - **Formats:**
        - **Individual:** ~12 min presentation + 3 min Q&A (~15 min total).
        - **Pair:** ~25 min presentation + 5 min Q&A (~30 min total, evenly split).
- **Recommended References:** _Mining of Massive Datasets_ (Leskovec, Rajaraman & Ullman), _Designing Data-Intensive Applications_ (Kleppmann), and _Foundations of Data Science_ (Blum, Hopcroft & Kannan). Textbooks are optional supplementary reading; lecture slides and referenced papers form the primary material.
- **Course Structure (5 Modules):** 1. Foundations & Big Data Infrastructure, 2. High-Dimensional Data Representation / Virtualization, 3. Non-Learning Computational Tasks / Concurrency, 4. Large-Scale Machine Learning / Persistence, 5. Advanced Topics.

---

## 2. Why Scale Out? Limits of Single-Machine Computing

### 2.1 The Data Growth Challenge

Modern workloads—such as web-scale crawls (petabytes of raw HTML), sensor/telemetry log streams (terabytes/day), and genomic/astronomical datasets—generate data faster than single disks or servers can store or write.

### 2.2 Vertical Scaling ("Scale Up") vs. Horizontal Scaling ("Scale Out")

- **Vertical Scaling (Scale Up):** Adding more CPU cores, RAM, drive bays, or higher-bandwidth NICs to a single node.
    - **Superlinear Cost Curve:** Upgrading to high-end hardware incurs steep price premiums. While a standard commodity server represents 1× cost for 1× capacity, a high-end server costs ~10× for 4× capacity, and a top-of-line mainframe costs ~100×+ for 16× capacity.
    - **Hard Physical Ceilings:** Motherboards hit hard physical limits on sockets, DIMM slots, thermal envelopes, and power consumption. As Prof. Tolomei emphasized, vertical scaling buys an organization time, but eventually saturates.
- **Horizontal Scaling (Scale Out):** Adding hundreds or thousands of modest, independent **commodity servers** to a cluster.
    - **Cost & Scalability:** Capacity grows roughly linearly by appending standard nodes.
    - **Complexity Shift:** The trade-off is software complexity; instead of relying on standard single-machine software, a software infrastructure layer must coordinate nodes and manage distributed state.

```
Aspect               Vertical Scaling (Scale Up)          Horizontal Scaling (Scale Out)
-----------------------------------------------------------------------------------------
Strategy             Bigger single machine                More commodity machines
Ceiling              Hard physical limit                  Effectively unbounded
Cost Growth          Superlinear at high end              Linear (commodity hardware)
Failure Model        Hardware-level reliability           Software-level fault tolerance
Complexity           Low (standard local software)        High (distributed coordination layer)
```

**

### 2.3 The "Failure Is Routine" Philosophy

At scale, hardware failure is a daily background event rather than an edge case. In a cluster of thousands of disks and nodes, components are failing continuously. Big data systems assume hardware is inherently unreliable and shift the responsibility for durability and availability to the **software layer**.

---

## 3. Foundations of Distributed File Systems (DFS)

### 3.1 DFS Abstraction & Core Design Assumptions

A **Distributed File System (DFS)** provides a unified logical file system interface over files physically split, replicated, and stored across many independent machines.

- **Throughput-Oriented, Latency-Tolerant:** Big data storage maximizes aggregate cluster bandwidth for large batch processing rather than optimizing for sub-millisecond single-request response times.
- **Huge Files & Sequential Access:** Targets gigabyte-to-terabyte files accessed sequentially start-to-finish. Random small reads/writes are rare.
- **Write-Once, Read-Many (WORM):** Files are written once (e.g., appended) and read repeatedly by analytics jobs. Eliminating arbitrary in-place modifications significantly relaxes consistency and locking overhead.

### 3.2 Block/Chunk Abstraction

Large files are split into contiguous, fixed-size **blocks** (or **chunks**), managed and replicated independently.

- **Trade-Offs of Large Block Sizes (64 MB–128 MB):**
    - **Pros:** Minimizes metadata volume stored by the central coordinator, reduces client-coordinator lookup requests, and encourages sustained sequential disk I/O.
    - **Cons:** Internal fragmentation for small files.

### 3.3 Separation of Metadata and Data

- **Metadata Service (Master / NameNode):** Manages namespace hierarchies, file-to-block mappings, and block locations. To ensure fast lookups, metadata is held **in memory**.
- **Data Servers (Chunkservers / DataNodes):** Store raw block byte streams directly on their local Linux file systems and handle bulk data transfers.
- **Client Interaction Model:** Clients ask the metadata server _only_ for block locations; all bulk data payload streams pass **directly between the client and data servers**, preventing the coordinator from becoming an I/O bottleneck.

```
  Client                   Metadata Server (Master / NameNode)          Data Servers (Chunkservers / DataNodes)
    │                                    │                                              │
    ├── 1. "Where is file X, block i?" ─►│                                              │
    │◄── 2. Handle & Node Locations ─────┤                                              │
    │                                                                                   │
    ├── 3. Direct Bulk Data Read / Write Transfer ─────────────────────────────────────►│
```

**

---

## 4. Architectural Deep Dive: GFS vs. HDFS

### 4.1 Comparative Overview

The **Google File System (GFS)** (published in 2003) directly inspired the open-source **Hadoop Distributed File System (HDFS)** developed at Yahoo.

```
Concept                  GFS (Google File System)            HDFS (Hadoop DFS)
---------------------------------------------------------------------------------------------
Metadata Coordinator     Master                              NameNode
Data Storage Node        Chunkserver                         DataNode
Basic Storage Unit       Chunk (default: 64 MB)              Block (default: 128 MB)
Metadata Persistence     OpLog + Checkpoint                  EditLog + FsImage
Checkpoint Helper        Shadow Master                       Secondary NameNode
Write Mechanism          Primary Lease + Direct Client Push  DataNode Replication Pipeline
```

**

### 4.2 GFS Architecture & Write Path

1. **In-Memory Metadata:** The GFS Master holds directory trees and file-to-chunk mappings in RAM. Chunk locations are not persisted on the Master's disk; instead, they are reconstructed at startup by polling Chunkservers via heartbeats.
2. **Lease Mechanism & Write Order:**
    - The Master grants a time-bounded **lease** to one replica Chunkserver, designating it as the **Primary**.
    - The client pushes raw data buffers directly to **all** replica Chunkservers.
    - The client then sends the write request to the Primary, which assigns a consecutive serial sequence number to the operation and forwards the ordered write command to secondary replicas.

### 4.3 HDFS Architecture & Write Pipeline

1. **Metadata Persistence:** The NameNode persists namespace state via an **FsImage** (complete point-in-time snapshot) and an **EditLog** (append-only mutation log). On startup, the NameNode loads the latest `FsImage` and replays all log entries in `EditLog`.
2. **Clarification on Secondary NameNode:** The Secondary NameNode is **not** a hot standby or failover replica. Its sole function is to periodically download `FsImage` and `EditLog`, merge them into a fresh checkpoint, and return it to the primary NameNode, offloading CPU work.
3. **Pipelined Write Path:**
    - The client requests a new block allocation from the NameNode.
    - The NameNode returns a sequence of DataNodes.
    - The client streams data strictly to the **first** DataNode in the pipeline, which buffers and streams it directly to the second DataNode, which in turn forwards it to the third.

```
HDFS Write Pipeline:
  Client ──► DataNode 1 ──► DataNode 2 ──► DataNode 3
    │           │              │              │
    ◄── Ack ────┴─── Ack ──────┴─── Ack ──────┘
```

**

4. **High Availability (HA) & Federation:**
    - **HA:** Uses Active/Standby NameNode pairs with synchronized shared edit logs (e.g., via quorum journal managers) to enable failover.
    - **Federation:** Uses multiple independent NameNodes sharing a common pool of DataNodes to scale namespace capacity past single-node RAM constraints.

---

## 5. Data Locality & Replica Placement

### 5.1 The Principle of Data Locality

> _"Move computation to data, not data to computation."_

Local disk/RAM I/O speeds (hundreds of MB/s to GB/s) far exceed cross-rack network bandwidth (~1–10 Gbps shared across racks). Distributed storage exposes block location mappings to job schedulers (such as MapReduce or Spark), enabling tasks to be assigned directly on or adjacent to the physical nodes holding input blocks.

### 5.2 Rack-Aware Replica Placement Policy

Default 3× replication balances cross-rack fault tolerance against cross-rack network traffic during writes:

1. **Replica 1:** Placed on the local node (or nearby node within the same rack as the client writer).
2. **Replica 2:** Placed on a node in a **different rack**.
3. **Replica 3:** Placed on a **different node within the same rack** as Replica 2.

_Trade-Off Insight:_ During in-class Q&A, Prof. Tolomei clarified that putting two copies on one rack and a third on a second rack provides fast intra-rack write propagation while ensuring the data survives an entire rack power or top-of-rack switch failure.

```
         RACK A                            RACK B
┌───────────────────────┐         ┌───────────────────────┐
│ Node 1: Replica 1     │         │ Node 4: Replica 2     │
│ Node 2                │         │ Node 5: Replica 3     │
│ Node 3                │         │ Node 6                │
└───────────────────────┘         └───────────────────────┘
```

**

---

## 6. Failure Handling & Statistical Failure Modeling

### 6.1 Failure Categories & Self-Healing

- **Transient vs. Permanent vs. Correlated Failures:** Systems handle brief network hiccups, permanent disk/node crashes, and correlated rack/datacenter outages.
- **Heartbeat Timeout Tuning:** Short timeouts detect dead nodes quickly but risk false positives on heavily loaded nodes; long timeouts prevent false positives but leave under-replicated data exposed longer.
- **Checksums & Silent Corruption:** Each block stores checksums. On read, if a mismatch is detected, the client fetches the data from an alternate replica and reports the corruption, triggering automatic re-replication.
- **Transparent Read Failover:** If a Chunkserver crashes mid-read, the client captures the timeout, queries the coordinator for alternative replica locations, and retries the read transparently.

---

### 6.2 Mathematical Derivation: Disk Failure Rate in Large Clusters

Prof. Tolomei presented a step-by-step statistical model demonstrating why disk failures are routine events in large clusters.

#### Model Setup & Assumptions

- **Cluster Size (\(N\)):** \(10,000\) commodity disks.
- **Annual Failure Rate (\(\text{AFR}\)):** \(2%\) per year (\(0.02\)).
- **Time Horizon:** \(1\) week (\(52\) weeks/year).
- **Uniformity & Independence:** Failures are assumed to be uniformly distributed across 52 weeks and independent across disks.

#### Step 1: Single Disk Weekly Failure Probability (\(p\))

The probability \(p\) that a given disk fails in a specific week is: \[p = \frac{\text{AFR}}{52} = \frac{0.02}{52} \approx 0.0003846 \quad (3.846 \times 10^{-4})\] **

#### Step 2: Formulating the Cluster Random Variable (\(X\))

Let \(X_i \sim \text{Bernoulli}(p)\) be an indicator variable for disk \(i\), where \(X_i = 1\) if disk \(i\) fails during the week, and \(X_i = 0\) otherwise.

The total number of weekly disk failures across the cluster is: \[X = \sum_{i=1}^{N} X_i \sim \text{Binomial}(N = 10,000, , p \approx 0.0003846)\] **

#### Step 3: Expected Failures (\(E[X]\)) and Standard Deviation (\(\sigma\))

- **Expected Weekly Failures:** \[E[X] = N \times p = 10,000 \times \left(\frac{0.02}{52}\right) \approx 3.85 \text{ failures/week}\] **
- **Variance (\(\text{Var}(X)\)):** \[\text{Var}(X) = N \cdot p \cdot (1 - p) = 10,000 \times 0.0003846 \times (1 - 0.0003846) \approx 3.8445\] **
- **Standard Deviation (\(\sigma\)):** \[\sigma = \sqrt{\text{Var}(X)} \approx 1.96 \text{ failures/week}\] **

#### Step 4: Probability of Exactly \(k=4\) Failures in a Given Week

Using the Binomial probability mass function: \[P(X = k) = \binom{N}{k} p^k (1 - p)^{N - k}\] For \(k = 4\): \[P(X = 4) = \binom{10000}{4} (0.0003846)^4 (1 - 0.0003846)^{9996} \approx 0.194 \quad (19.4%)\] ** _There is nearly a **1 in 5 chance** of experiencing exactly 4 disk deaths in any given week._

#### Step 5: Probability of At Least One Disk Failure in a Week

Using the complement rule: \[P(X \ge 1) = 1 - P(X = 0) = 1 - \binom{N}{0} p^0 (1 - p)^N = 1 - (1 - p)^N\] \[P(X \ge 1) = 1 - (1 - 0.0003846)^{10000} = 1 - (0.9996154)^{10000} \approx 1 - 0.0213 = 0.9787 \quad (97.87%)\] **

#### Empirical Conclusion

In a cluster of 10,000 disks with a 2% annual failure rate, **at least one disk fails during roughly 98% of weeks**, averaging **~3.85 failures per week**. This statistical reality justifies the core scale-out philosophy: hardware failure is a daily operational reality, and storage software must be continuously self-healing.

---
