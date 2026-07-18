
# **1. Google Bigtable: A Distributed Storage System**

## Data Storage and File Format (SSTables and Tablets)

- **SSTables:** Bigtable stores data internally using the Google SSTable file format, which contains a sequence of blocks (typically 64KB) and a block index located at the end of the file. When an SSTable is opened, the index is loaded into memory. A lookup requires only a single disk seek: a binary search is performed on the in-memory index to find the correct block, which is then read directly from the disk.
- **Tablets:** A Bigtable is dynamically partitioned by row ranges into smaller units called tablets, which act as the basic unit of distribution and load balancing. A table initially consists of one tablet, but it automatically splits into multiple tablets (typically 100-200 MB each) as the data grows. The set of existing tablets only changes when a table is created or deleted, when two tablets are merged, or when a tablet splits.

**System Architecture (Master, Tablet Servers, and Chubby)**

- **Chubby Lock Service:** Bigtable relies heavily on Chubby, a persistent, highly-available distributed lock service consisting of five replicas. Bigtable uses Chubby to ensure only one master is active, to store the bootstrap location of data, to discover live tablet servers, and to store schema information and access control lists.
- **Master Server:** The master assigns tablets, balances load, detects when tablet servers join or expire, manages garbage collection in GFS, and handles schema changes.
    - _Startup sequence:_ The master grabs a unique lock in Chubby, scans the servers directory to find live servers, communicates with them to discover assigned tablets, and finally scans the METADATA table to find and assign unallocated tablets.
    - _Failures:_ If the master's session with Chubby expires, it kills itself to prevent network vulnerabilities, but this does not impact the current assignment of tablets to servers.
- **Tablet Servers:** These servers manage their assigned tablets, handle incoming read and write requests, and perform tablet splits when they grow too large. The master discovers them by monitoring the Chubby "servers directory" and periodically checks their lock status. If a tablet server fails or loses its lock, the master acquires the lock, deletes the server's Chubby file to permanently stop it from serving, and reassigns its tablets.

**Metadata and Tablet Location**

- Tablet locations are managed in a three-level hierarchy (similar to a B+-tree):
    1. A file in Chubby stores the location of the **root tablet**.
    2. The root tablet (which never splits) points to all other **METADATA tablets**.
    3. Each METADATA tablet points to the actual **user tablets**.
- Clients cache these locations locally; if a location is unknown or stale, the client recursively moves up the hierarchy to discover and prefetch the correct location.

**I/O Operations and Recovery**

- **Writes:** Incoming writes are validated for format and authorization, appended to a commit log using group commit, and then inserted into an in-memory sorted buffer called a **memtable**.
- **Reads:** Reads are similarly validated and then executed on a merged view of the on-disk SSTables and the in-memory memtable.
- **Commit Log & Recovery:** To maximize write performance, a tablet server co-mingles mutations for all its assigned tablets into a **single physical commit log file**. If the server fails and its tablets are distributed to many new servers, reading the entire log would be highly inefficient. To fix this, the master sorts the log entries by `<table, row name, log sequence number>`, making mutations for specific tablets contiguous. This allows a recovering server to rebuild its memtable efficiently via a single disk seek and sequential read.

---

# **2. Amazon Dynamo: Highly Available Key-Value Store**

**Partitioning and Data Assignment**

- **Consistent Hashing:** Partitioning allows the system to scale incrementally. It is implemented using consistent hashing, treating the hash function's output as a fixed circular space or "ring".
- **Data Assignment:** A data item's key is hashed to find its position on the ring. The system walks clockwise to find the first node with a larger position value, meaning each node is responsible for the ring region between itself and its immediate predecessor.
- **Virtual Nodes:** Basic consistent hashing causes non-uniform load distribution and ignores hardware heterogeneity. Dynamo resolves this using "virtual nodes", assigning each physical node multiple tokens (positions on the ring) proportional to its capacity.

**Replication and Availability**

- **Replication Strategy:** Data is replicated across _N_ hosts. The coordinator stores the key locally and copies it to the _N-1_ clockwise successors. To ensure replicas land on entirely distinct physical hosts despite the use of virtual nodes, the system creates a "preference list" that skips ring positions until _N_ unique physical machines are found.
- **Handling Failures (Hinted Handoff):** If the top _N_ nodes are unavailable (e.g., due to a network partition), Dynamo employs **hinted handoff**. The write request is sent to the first _N_ healthy nodes found while walking the ring, even if they aren't the intended recipients. These replicas are stored in a separate local database with a "hint" regarding their true destination, and they are automatically moved to the correct node once it comes back online.

**Data Versioning and Conflict Resolution**

- Because Dynamo prioritizes availability, writes are not rejected. Consequently, a `get()` operation may return multiple, unreconciled versions of data.
- Dynamo treats every modification as a new, immutable version identified by a **vector clock**.
- If the system cannot syntactically reconcile divergent versions (e.g., two concurrent writes occurring before either is aware of the other), it returns all unreconciled versions to the client application, which must perform **semantic reconciliation**.

**Partitioning Strategies** Dynamo evolved through three specific partitioning strategies to handle load and node churn:

1. **T random tokens per node & partition by token value:** Nodes get random positions. _Limitations:_ Node joins/departures require expensive local database scans and Merkle tree recalculations, and taking snapshots for archival is difficult because key ranges are scattered randomly.
2. **T random tokens & equal-sized partitions:** The hash space is divided into _Q_ equal partitions. Tokens only build the mapping function, assigning partitions to the first _N_ unique nodes encountered. This successfully decouples partitioning from placement but yields the worst load-balancing efficiency.
3. **Q/S tokens per node & equal-sized partitions (Optimal):** The hash space is divided into _Q_ equal partitions, and each node receives an equal share of tokens (_Q/S_). Tokens are stolen or redistributed when nodes join or leave. This fixed-partition setup allows partitions to be stored in separate files, enabling much faster bootstrapping, recovery, and archiving, while also achieving the best overall load-balancing efficiency.