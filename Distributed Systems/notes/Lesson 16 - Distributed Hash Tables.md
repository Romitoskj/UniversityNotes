This notes explores decentralized storage systems, focusing on the **Interplanetary File System (IPFS)** and the mathematical infrastructure that allows them to function, specifically **Distributed Hash Tables (DHT)** like **Chord**.

### I. IPFS
#### Content-Addressable Storage

IPFS is a distributed system with no central authority, where peers can store and retrieve files.

- **Content Addressing:** Unlike traditional systems where files are accessed by a name or location, IPFS uses **Content IDs**. The "name" of the file is effectively a **hash** of its content.
- **Immutability:** Files in IPFS cannot be modified. If a user wants to change a file, they must save a new version, resulting in two different files with two distinct hashes.
- **Chunks:** To manage storage, files are broken down into smaller pieces called **chunks**, which are then distributed across various servers in the network.

####  System Properties and Persistence

- **Consistency:** This is generally not a problem because files are read-only; there aren't multiple conflicting versions of the same hash.
- **Partition Tolerance:** low, if a chunk of a file is in another partition it can't be retrieved
- **Availability and Replication:** Built-in availability is **low** because there is no automatic replication. If a server holding a specific chunk leaves the network forever, that data is lost.
- **Pinning:** To prevent data loss, users can "**pin**" a file. A pinned file is guaranteed by a server (often the user's own server or a paid service like **Pinata**) not to be deleted.
- **Confidentiality:** By default, files are stored in plain text. For privacy, users must **encrypt** their files themselves before uploading them to the system.

### II. The Chord Protocol
#### A. Mapping the Ring

The core infrastructure used to locate files in these systems is a **DHT**, specifically the **Chord** protocol, is a "ring".

![](../assets/Pasted%20image%2020251219121219.png)

- **The Identifier Space:** Identifiers (fingerprints) are treated as numbers in a circular space or **ring**, ranging from $0$ to $2^n-1$ where $n$ is the dimension of the space (*e.g.* $0$ to $2^{256} - 1$ if using SHA-256).
- **Projection:** Both servers (using their IP addresses hash) and files (using their content hashes) are projected onto this ring.
- **Storage Rule:** A file is stored on the **next server** encountered when moving **clockwise** around the ring from the file's hash position (the first server with greater hash).
- **Load Balancing:** Because hash functions "act like random functions", files and servers are distributed **uniformly**. While distribution isn't perfect, the server with the most files typically only handles a factor of $O(\log n)$ files.

####  B. Routing with Finger Tables

To avoid contacting every server one by one to find a file, Chord uses a routing mechanism called a **Finger Table**.

- **Finger Logic:** Each server maintains a table of "fingers" pointing to other servers at exponential distances ($p + 2^0, p + 2^1, p + 2^2, \dots$ , where $p$ is the hash of the IP address of the server containing the table). The table doesn't contains all the servers but only does at this distances.
- **User request:** When a user request the CID of a file to a server it can be stored by the server or, if not, the server relays the request to the one with preceding hash value that does the same thing.
- **$\log n$ Search ("Binary search"):** Every time a request "hops" to a server in the finger table, the distance to the target file is reduced by at least **half**. Consequently, finding any file in a network of $n$ servers only requires **$O(\log n)$ communications**.

#### C. Dynamic Membership

Distributed systems must handle nodes joining or leaving, a process known as **churn**.

- **Joining the Ring:** When a new server joins, it must contact its successor to transfer the files for which it is now responsible. It then builds its own finger table, which takes approximately $O(\log^2 n)$ time.
- **Leaving the Ring:** If a server leaves abruptly without its files being pinned elsewhere, those files are lost to the system.

---

**Analogy for Understanding Chord:** Imagine a **circular library** where every book has a specific number, and several librarians are standing at different points along the circular shelf. Each librarian is responsible for all the books sitting between them and the librarian standing to their left. Instead of walking past every single book to find what you need, each librarian has a "cheat sheet" (the Finger Table) that tells them exactly where colleagues are standing at distances of 10, 20, 40, and 80 metres away. This allows them to point you to the right section of the library very quickly, skipping over vast sections of the circle in just a few steps.