# Cloud Storage Introduction

### Goals of Data Storage in the Cloud

- Massive scaling on demand
- High availability
- Simplified application development and deployment

## 1. Atomicity in Cloud Storage

A transaction is a multi-step operation that should complete without interruption. Achieving this requires hardware support, such as non-interruptible instruction set operations (e.g., test-and-set, compare-and-swap) and mechanisms to create critical sections (e.g., locks, semaphores, monitors). There are two main types of atomicity:

- **All-or-Nothing Atomicity:** The transaction is executed completely, or aborted entirely. It requires two phases: a pre-commit phase (to gather necessary resources) and a post-commit (irreversible actions), the transition is called commit point. If the transaction fails before committing, it is aborted. Systems maintain a log of committed transactions to recover from failures and guarantee consistency.
	
	![](Images/Pasted%20image%2020260511165724.png)
	
- **Before-or-After Atomicity:** A weaker form of atomicity where the result of concurrent read and write operations is identical to the result if they were executed sequentially.
	
	![558](Images/Pasted%20image%2020260511165840.png)

## 2. Storage Models

To implement these atomicity properties, different storage models are used:

- **Cell Storage Model:** Simulates the physical structure of a disk (sectors/blocks) by using cells of the same size. This model naturally guarantees **read/write coherence** and **before-or-after atomicity**. However, it **does not guarantee all-or-nothing atomicity** because there is no native way to reserve a cell and undo an action if a failure occurs.
- **Journal Storage Model:** Combines a cell storage area with a "manager" and a version history/log. Before altering a cell, the intended change is written to the log that maintains the history of all variables in the cell store. This model **guarantees all-or-nothing atomicity** because the log allows the system to undo partial transactions and recover from failures.
	![](Images/Pasted%20image%2020260511171242.png)

## 3. Eventual Consistency and the Paxos Consensus Protocol

Unlike traditional relational databases that rigidly enforce ACID properties, NoSQL distributed systems often relax consistency to improve availability and scalability, leading to **"eventual consistency"**. To manage this, they use **consensus protocols** to agree on a single proposed value across multiple replicas.

### The Paxos Algorithm:

- **Assumptions:** Paxos assumes a system prone to non-Byzantine failures—meaning nodes can crash and restart, and the network can lose or delay messages, but messages are **never corrupted**.
- **Roles:** Entities in the system act as **Clients** (make requests), **Proposers/Leaders** (coordinate the agreement), **Acceptors** (act as the fault-tolerant memory), and **Learners** (distribute the agreed results).
- **Process:** A value is chosen if a simple majority (quorum) of Acceptors agree. In **Phase 1 (Prepare)**, the Leader sends a proposal number to Acceptors, who "promise" not to accept requests with a lower number. In **Phase 2 (Accept)**, the Leader asks the Acceptors to officially accept the value, which is then passed to the Learners if the majority complies.

## 4. Google File System (GFS)

GFS was designed around the specific workload characteristics of cloud applications: massive files (GBs to TBs), predominantly **append operations** (rather than random writes), sequential reads, and relaxed consistency.

- **Chunk Size:** Files are divided into massive **64 MB chunks**. This large size reduces the metadata the system must manage, decreases the number of network requests needed to locate data, allows persistent client-server connections, and reduces disk fragmentation.
- **Architecture:** It features a centralized **Master** node that stores all metadata and chunk locations completely in-memory for speed, paired with multiple **Chunkservers** that store the actual data on their local disks. To recover from crashes, the Master relies on an atomic operation log.
- **Write Protocol:** The Master grants a "lease" to a **primary Chunkserver**. When a client writes, it pushes data to the primary and all secondary Chunkservers, which store the data in a temporary buffer. The client then sends a formal write request to the primary, which applies the mutation and coordinates the secondaries to do the same before acknowledging the client.

## 5. Hadoop Distributed File System (HDFS)

Similar to GFS, HDFS is a master/slave system designed for big data, though it is written in Java and is not fully POSIX compliant.

- **Architecture:** It consists of a **NameNode** (the Master handling namespace, metadata, and logs) and **DataNodes** (the slaves handling read/write ops).
- **Block Size & Rack Awareness:** Files are split into 64-128 MB blocks with 3 replicas. HDFS implements **rack awareness**, ensuring that replicas are stored across at least two different physical network racks to guarantee availability if a rack fails.
- **Write Pipeline:** Unlike GFS, an HDFS client **sends data only to the primary DataNode**. The primary then pipelines the data to the second secondary, which passes it to the third in a chain-like fashion.

## 6. Object Stores (e.g., Amazon S3)

Object stores differ fundamentally from file systems like GFS or HDFS. In an object store, **you cannot append to or randomly write inside a file**. You interact with the storage via APIs to retrieve or store the entire object at once. If you need to modify an object, you must download it, alter it in your application, and overwrite the existing file entirely.

- **Data Model (Buckets and Objects):** Data is stored as opaque "objects" grouped inside "buckets". Buckets must be explicitly created, have globally unique names, and use Access Control Lists (ACLs) to manage permissions. Objects are byte strings (up to 5 GB) identified by a unique key and a URI path, and they are stored or retrieved via HTTP `PUT` and `GET` requests.
- **Failure and Error Handling:** The system relies on the client application to handle read/write failures by retrying requests. To guarantee data integrity, S3 computes an MD5 checksum for every object, returning it in a field called the `ETag`. Clients compare this `ETag` with their own MD5 computation to verify the object was not corrupted during transmission.
- **Consistency Model:** S3 provides **strong read-after-write consistency** for `PUT` and `DELETE` requests of objects, meaning updates to a single key are atomic. However, if concurrent writers attempt to update the same key, S3 uses a **"last-writer-wins"** semantic; it does not natively support object locking. Furthermore, bucket configurations operate on an **eventual consistency** model (e.g., a deleted bucket might temporarily still appear in a list of buckets).

### **7. NoSQL Data Stores (Overview)**

For many cloud applications, traditional Relational Database Management Systems (RDBMS) are insufficient because they cannot scale efficiently to handle massive amounts of data under tight latency constraints.

- **Dropping ACID Properties:** To achieve high availability and scalability, NoSQL databases drop strict ACID (Atomicity, Consistency, Isolation, Durability) guarantees.
- **Eventual Consistency:** They use a "soft-state" approach where data is partitioned and replicated across multiple servers. Data may be temporarily inconsistent but becomes "eventually consistent" over time.

### **8. Google BigTable**

BigTable is a distributed storage system built to handle structured data across thousands of commodity servers with a highly flexible data model.

- **Data Model:** It is a sparse, distributed, persistent multidimensional sorted map. Data is indexed by three parameters: a **row key**, a **column key**, and a **timestamp**.
    - **Rows:** Row keys are arbitrary strings ordered lexicographically. Row ranges are dynamically partitioned into units called **tablets**, which serve as the basis for load balancing.
    - **Columns:** Column keys are grouped into sets called **column families**. Access control and memory accounting are managed at the column family level, and data within a family is of the same type.
    - **Timestamps:** BigTable stores multiple versions of the same cell data, ordered by decreasing timestamps (so the most recent version is read first). Older versions are automatically garbage-collected based on configured rules (e.g., keeping only the last $n$ versions).
- **Building Blocks & Architecture:** BigTable utilizes a single **Master server** (to assign tablets and balance loads) and many **Tablet servers** (to handle read/write operations). It is built on top of the **Google File System (GFS)** and stores data in **SSTables** (a persistent, immutable map of keys to values optimized for single disk seeks). It also heavily relies on **Chubby**, a distributed lock service, to elect the master and manage metadata.

### **9. Amazon Dynamo**

Dynamo is a highly available, decentralized key-value store designed to manage the state of Amazon's most critical services, such as shopping carts, where outages have direct financial consequences.

- **Prioritizing Availability:** Dynamo explicitly trades strict consistency for high availability. It uses optimistic replication to remain **"always writable,"** meaning updates are never rejected, even during network failures.
- **Conflict Resolution:** Because it is always writable, divergent data versions can emerge. Dynamo pushes the complexity of conflict resolution to the read phase. Conflicts are ultimately reconciled by the client application itself (e.g., merging two versions of a shopping cart).
- **Partitioning via Consistent Hashing:** To scale incrementally, Dynamo distributes data across nodes using a consistent hashing ring. To prevent unbalanced loads and account for heterogeneous hardware, it assigns multiple **virtual nodes** to each physical machine on the ring.
- **Replication Strategy:** A data item is replicated across $N$ hosts. The system hashes the key to find the coordinator node on the ring, which stores the data locally and also replicates it to the next $N-1$ clockwise successor nodes.
- **Data Versioning:** Every modification is treated as a new, immutable version. Dynamo uses **vector clocks** (a list of node and counter pairs) to track causality between versions and identify conflicts when a `Get()` operation is called.