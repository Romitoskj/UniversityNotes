### I. Introduction and Overview

- **Definition:** Peer-to-Peer (P2P) systems are system could share files or code without a central authority.
- **Context:** BitTorrent, invented in 2001, became famous for allowing the sharing of **large** files, such as movies or operating system distributions.
- **The Problem:** The core challenge for sharing large files, particularly when simultaneous demand is high (like on "click days"), is that a single central server can become easily **overloaded**, potentially leading to a Denial of Service (DoS) situation.
- **The P2P Solution:** Instead of downloading from one central server, peers download pieces of the file **concurrently** from **multiple other peers**, distributing the load and allowing the file to be reconstructed. The system can continue to work even if the original central server goes down, provided the file is fully shared by the community.

### II. System Components and Structure

#### `.torrent` File:
To start downloading a file, a user must first obtain a **.torrent file**, typically from the web. The torrent file contains critical metadata:
- URL of the tracker
- name of the file
- piece length (typically 256 KB)
- version
- length of the file
- root hash of the Merkel Tree
- piece layers ($R_1,R_2, ..., R_n$)
#### Tracker:
This is a server responsible for **coordination**. There are multiple of them and each file refers to one of them.
The user contacts the tracker to obtain a list of **peers** who are currently sharing that specific file. The tracker usually does _not_ keep the file itself.w
#### File Division and Integrity:
- The entire file is divided into **pieces** (typically 256 kilobytes).
- These pieces are further divided into **sub-pieces** (typically 16 kilobytes).
- Integrity is maintained by including the **root hash of the Merkle Tree** of the file within the torrent file.
- To check if a piece has been tampered with, the user must download the data and the necessary sibling hashes (the Merkle proof) to verify it against the Merkle root (==?or they are included in the piece layers?==).

>[!info]- Merkle Trees
>In peer-to-peer (P2P) systems, a Merkle tree is used to **verify that a specific piece of data is part of a larger collection** without requiring the user to possess or download the entire collection first. This is particularly useful when downloading large files, such as movies or operating system distributions, where data is received in **small chunks from multiple different peers** who may not be trusted.
> 
> To prevent a system from being "polluted" with fake or corrupted data, a user must be able to verify that each chunk they receive is authentic. A standard hash of the entire file is insufficient for this because the user cannot check it until they have received every single piece. Merkle trees solve this by allowing for **incremental verification**.
>![](../assets/Pasted%20image%2020251224200840.png)
> The verification process follows these key steps:
> 
> - **Tree Construction:** The entire collection of data is divided into chunks, and each chunk is hashed individually. These hashes are then paired up and hashed again, a process that continues until only a single **root hash** remains at the top of the tree.
> - **The Root Hash:** The user typically obtains the **authentic root hash** from a trusted source before starting the download. This root serves as the definitive "fingerprint" for the entire collection.
> - **The Merkle Proof:** When a peer sends a data chunk, they also provide a **Merkle proof**, which is a specific sequence of "brother" hashes. These are the hashes located on the path from that chunk up to the root, representing the "missing pieces" needed to reconstruct the path (e.g. in the image above if we get the data chunk L-4 we also get the hash 1-0 and 1).
> - **Verification:** The receiver hashes the chunk they just received and then uses the hashes in the Merkle proof to **recalculate the root**. They take their calculated hash, combine it with the first brother hash, hash the result, and repeat the process until they reach the top. If the final calculated hash matches the known authentic root hash, the chunk is verified as authentic.
> 
> The main advantage of this system is its **efficiency**. Because the tree structure is logarithmic, the size of the proof ($O(\log n)$) remains very small even for massive collections of data.

### III. Download and Contribution Mechanisms

![](../assets/Pasted%20image%2020251218094702.png)

#### Peer Roles:
- **Leecher:** A user who is currently **downloading** the file, by downloading its pieces concurrently from different users. Each piece (all its sub-pieces) is downloaded from a single peer. After collecting some pieces the user starts uploading them to other users.
- **Seeder:** A user who has downloaded the entire file and **remains in the system to upload** pieces to others. Usually when a user finish to download the file the client starts act as a seed. While this is considered a "good habit" to contribute to the community, the user can chose to leave the system.
#### Piece Selection Strategy:
The strategy for choosing which pieces to download first is key:
1. **Random Selection:** This is good for **system balancing**, ensuring all pieces are requested by peers simultaneously (not the always the same piece).
2. **Rarest First:** This strategy is good for the **community** as it prioritises downloading pieces held by very few peers, preventing files from being "lost" if those rare peers disconnect.

**Implementation:** BitTorrent starts with a **Random** strategy to quickly acquire an initial set of pieces (allowing the new user to start uploading quickly), and then switches to **Rarest First**.

#### Handling Free Riders:
A free rider is a participant who only downloads and refuses to upload.
- **Tit-for-Tat (T4T):** This technique limits free riding by requiring that a user can only download a piece if they are simultaneously uploading another piece to the other peer.
- **Optimistically Unchoking:** Since a new user (leecher) starts with nothing to upload, the system must allow them to acquire their first pieces. This is done through the **unchoking algorithm**. Every minute (or period), the system optimistically unchokes (uploads to) a random peer, regardless of T4T, giving new users a chance to join and start contributing.
#### End Game Mode:
Near the end of the download (e.g., 98-99% complete), speed often slows down because the last remaining pieces are often held only by slow or distant peers,.
To solve this, when only the last piece (or few pieces) remain, the system enters **End Game Mode**, which allows the peer to download sub-pieces simultaneously from different users.

### IV. System Properties and Attacks

- **CAP Properties Analysis:**
    - **Partition Tolerance:** Considered **pretty good**. The system can work even if the network is partitioned, provided there are seeders remaining in the partition.
    - **Consistency:** Not strictly well-defined because the system is primarily **read-only**.
    - **Availability:** Not always guaranteed, but generally acceptable, depending on how common the file is and the number of active peers/seeders.
- **Attacks:** Corporations may fight the system by:
    - Becoming **seeders** themselves to log the IP addresses of downloaders.
    - Trying to **pollute the system** by distributing fake pieces (less effective due to hash checks) or distributing entirely fake files (polluting the entire file).
- **Modern Usage:** Although BitTorrent is less used by the public now, the underlying ideas—such as decentralized downloads and fetching content from multiple locations—are used by major corporations to distribute operating system updates quickly without overloading a central server.

>[!question]- Questions
> - I read that in the first version of bittorent there wasn't merkle trees but the list of hashes of each file piece and in the second version the merkle tree was implemented and the root stored in the .torrent file. Doing so and having peers send the merkle proof along the sub-piece is it possible to check the integrity of each sub-piece. Is that correct?
> - If so what are piece layers ($R_1,R_2, ..., R_n$) in torrent file?