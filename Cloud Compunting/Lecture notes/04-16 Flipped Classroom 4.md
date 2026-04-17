# Cloud Storage: GFS & HDFS

## 1. The Google File System (GFS)

### System Architecture & Master Operations

- **HeartBeat Communication:** GFS uses periodic HeartBeat messages between the master and chunkservers instead of synchronous request-response interactions. This minimizes overhead, allows the master to batch tasks (like garbage collection), piggyback lease extensions, and promptly respond to urgent client requests.
- **Chunk Location Management:** The master does not store chunk locations persistently on disk to avoid complex synchronization issues. Instead, chunkservers report their inventories during startup and via ongoing HeartBeat messages.
- **Chunk Size:** GFS uses massive **64 MB chunks**. This reduces the metadata size (allowing it to fit entirely in the master's RAM), minimizes client-master location requests, and reduces disk fragmentation.
- **Operation Log:** Central to GFS, this persistent log records all metadata changes and acts as a logical timeline for concurrent operations. It is used for fast state recovery by restoring the latest checkpoint and replaying the log.
- **High Availability:** The master's operation logs and checkpoints are replicated across multiple machines. GFS utilizes read-only **"shadow" masters** that lag slightly behind the primary to provide continuous read availability without affecting file content reads.

### Data Access & Consistency

- **Client Interactions:** Clients cache chunk locations to bypass the master and communicate directly with chunkservers for data transfers.
- **Decoupled Data Flow:** To avoid bottlenecks and minimize latency, control flow is separated from data flow. Data is pushed linearly across a chain of chunkservers via pipelined TCP connections, prioritizing the closest IP address first.
- **Consistency Model (File Regions):** GFS applies mutations in the exact same serial order across all replicas, dictated by the primary replica holding the lease.
    - _Consistent:_ All clients see the same data.
    - _Defined:_ Consistent, and reflects the entirety of a specific mutation.
    - _Undefined:_ Consistent, but consists of intermingled data fragments from concurrent successful writers.
    - _Inconsistent:_ The result of a failed mutation.
- **Byte-wise Identity:** Replicas do not need to be byte-wise identical. Concurrent atomic record appends may result in varying padding or duplicates across replicas, which applications are designed to handle.

### Replica Placement Policy

- The master places new chunks based on below-average disk space, limiting active clone operations (to prevent write hotspots), and ensuring **rack diversity** to survive single-rack failures. Replicas are re-replicated automatically if a node fails or data is corrupted.

--------------------------------------------------------------------------------

## 2. The Hadoop Distributed File System (HDFS)

### System Architecture & NameNode Operations

- **Metadata Components:**
    - _Image:_ The file system metadata kept entirely in RAM.
    - _Journal:_ A persistent write-ahead commit log for transactions.
    - _Checkpoint:_ A persistent, on-disk record of the image.
- **CheckpointNode:** Periodically merges the Checkpoint and Journal to create a new, truncated checkpoint. This prevents infinite journal growth, protects metadata, and speeds up restarts. Metadata is safely stored across multiple directories and a remote NFS server.
- **Node Communication:** DataNodes initiate a strict **handshake** at startup to verify namespace IDs and software versions, followed by a full block report. Routine Heartbeats carry capacity and transfer stats, and the NameNode uses the replies to piggyback commands.

### Data Access & I/O

- **Single-Writer Model:** HDFS enforces exclusive write leases; multiple clients cannot write simultaneously. **Random writes are not allowed**, only appends. However, concurrent reading is permitted, even while a file is actively being written to.
- **Write Flow:** The NameNode nominates a suite of DataNodes. The client forms a pipeline ordered by network proximity, buffers data into 64 KB packets, and streams them sequentially, waiting for acknowledgments before closing the block.
- **Read Flow:** Clients receive a list of replica locations ordered by network distance and read from the closest available DataNode, retrying if necessary.
- **Data Integrity:** Clients verify data checksums on read. Unlike GFS, each DataNode runs a continuous background **Block Scanner** to proactively verify resting blocks against corruption.

### Replica Placement & Balancing

- **Placement Strategy:** By default, the 1st replica is placed locally, and the 2nd and 3rd are placed on **different nodes within a single remote rack**. This balances write performance (reduces inter-rack traffic) while guaranteeing survival against a single rack failure.
- **Variable Replication:** The replication factor is user-selectable on a file-by-file basis.
- **Balancing:** HDFS disk utilization is maintained via an administrator-run **Balancer tool** that iteratively shifts blocks, and by selectively removing over-replicated blocks from the fullest nodes.

--------------------------------------------------------------------------------

## 3. Comparative Summary: GFS vs. HDFS

- **Client & Master Interactions:** Both separate application data from metadata. While GFS uses a decoupled, linear data push to maximize bandwidth, HDFS relies on a sequential packet pipeline requiring acknowledgments.
- **Node Communication:** Both utilize Heartbeats. However, HDFS requires strict startup handshakes and explicit, hourly block reports, whereas GFS relies primarily on continuous heartbeats for state updates.
- **Replication & Placement:** Both employ large blocks (64 MB for GFS; 128 MB for HDFS), rack-aware placement, and a default of 3 replicas. GFS organically balances disk usage in the background, while HDFS relies on a strict default placement ruleset and a manual Balancer tool.
- **Access Modes:** Both are optimized for Write-Once, Read-Many (WORM) patterns and large streaming reads. A major difference is that **GFS supports concurrent writers** via Atomic Record Appends, whereas **HDFS strictly enforces a single-writer model**.
- **Integrity & Recovery:** Both utilize checksums and replicated operation logs. For high availability, GFS uses read-only "shadow" masters and chunk versioning. HDFS relies on CheckpointNodes (and BackupNodes), a background Block Scanner, and admin-triggered Snapshots to roll back corrupted file system states.