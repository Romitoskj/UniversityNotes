# Cloud Storage Introduction

## 1. Atomicity in Cloud Storage

A transaction is a multi-step operation that should complete without interruption. Achieving this requires hardware support, such as non-interruptible instruction set operations (e.g., test-and-set, compare-and-swap) and mechanisms to create critical sections. There are two main types of atomicity:

- **All-or-Nothing Atomicity:** The transaction is executed completely, or aborted entirely. It requires two phases: a pre-commit phase (to gather necessary resources) and a commit point. If the transaction fails before committing, it is aborted. Systems maintain a log of committed transactions to recover from failures and guarantee consistency.
- **Before-or-After Atomicity:** A weaker form of atomicity where the result of concurrent read and write operations is identical to the result if they were executed sequentially.

**2. Storage Models**

To implement these atomicity properties, different storage models are used:

- **Cell Storage Model:** Simulates the physical structure of a disk (sectors/blocks) by using cells of the same size. This model naturally guarantees **read/write coherence** and **before-or-after atomicity**. However, it **does not guarantee all-or-nothing atomicity** because there is no native way to reserve a cell and undo an action if a failure occurs.
- **Journal Storage Model:** Combines a cell storage area with a "manager" and a version history/log. Before altering a cell, the intended change is written to the log. This model **guarantees all-or-nothing atomicity** because the log allows the system to undo partial transactions and recover from failures.

**3. Eventual Consistency and the Paxos Consensus Protocol**

Unlike traditional relational databases that rigidly enforce ACID properties, NoSQL distributed systems often relax consistency to improve scalability, leading to **"eventual consistency"**. To manage this, they use **consensus protocols** to agree on a single proposed value across multiple replicas.

**The Paxos Algorithm:**

- **Assumptions:** Paxos assumes a system prone to non-Byzantine failures—meaning nodes can crash and restart, and the network can lose or delay messages, but messages are **never corrupted**.
- **Roles:** Entities in the system act as **Clients** (make requests), **Proposers/Leaders** (coordinate the agreement), **Acceptors** (act as the fault-tolerant memory), and **Learners** (distribute the agreed results).
- **Process:** A value is chosen if a simple majority (quorum) of Acceptors agree. In **Phase 1 (Prepare)**, the Leader sends a proposal number to Acceptors, who "promise" not to accept requests with a lower number. In **Phase 2 (Accept)**, the Leader asks the Acceptors to officially accept the value, which is then passed to the Learners if the majority complies.

**4. Google File System (GFS)**

GFS was designed around the specific workload characteristics of cloud applications: massive files (GBs to TBs), predominantly **append operations** (rather than random writes), sequential reads, and relaxed consistency.

- **Chunk Size:** Files are divided into massive **64 MB chunks**. This large size reduces the metadata the system must manage, decreases the number of network requests needed to locate data, allows persistent client-server connections, and reduces disk fragmentation.
- **Architecture:** It features a centralized **Master** node that stores all metadata and chunk locations completely in-memory for speed, paired with multiple **Chunkservers** that store the actual data on their local disks. To recover from crashes, the Master relies on an atomic operation log.
- **Write Protocol:** The Master grants a "lease" to a **primary Chunkserver**. When a client writes, it pushes data to the primary and all secondary Chunkservers, which store the data in a temporary buffer. The client then sends a formal write request to the primary, which applies the mutation and coordinates the secondaries to do the same before acknowledging the client.

**5. Hadoop Distributed File System (HDFS)**

Similar to GFS, HDFS is a master/slave system designed for big data, though it is written in Java and is not fully POSIX compliant.

- **Architecture:** It consists of a **NameNode** (the Master handling namespace, metadata, and logs) and **DataNodes** (the slaves handling read/write ops).
- **Block Size & Rack Awareness:** Files are split into 64-128 MB blocks with 3 replicas. HDFS implements **rack awareness**, ensuring that replicas are stored across at least two different physical network racks to guarantee availability if a rack fails.
- **Write Pipeline:** Unlike GFS, an HDFS client **sends data only to the primary DataNode**. The primary then pipelines the data to the second secondary, which passes it to the third in a chain-like fashion.

**6. Object Stores (e.g., Amazon S3)**

Object stores differ fundamentally from file systems like GFS or HDFS. In an object store, **you cannot append to or randomly write inside a file**. You interact with the storage via APIs to retrieve or store the entire object at once. If you need to modify an object, you must download it, alter it in your application, and overwrite the existing file entirely.