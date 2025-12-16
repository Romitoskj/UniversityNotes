### I. Introduction and Overview

- **Definition:** Peer-to-Peer (P2P) systems are system could share files, ideas, or code without a central authority.
- **Context:** BitTorrent, invented in 2001, became famous for allowing the sharing of large files, such as movies or operating system distributions.
- **The Problem Solved:** The core challenge for sharing large files, particularly when simultaneous demand is high (like on "click days"), is that a single central server can become easily **overloaded**, potentially leading to a Distributed Denial of Service (DDoS) situation.
- **The P2P Solution:** Instead of downloading from one central server, peers download pieces of the file from **multiple other peers**, distributing the load and allowing the file to be reconstructed. The system can continue to work even if the original central server goes down, provided the file is fully shared by the community.

### II. System Components and Structure

- **Torrent File:** To join a BitTorrent community and start downloading, a user must first obtain a **torrent file**,.
    - **File Structure:** The torrent file contains critical metadata, including the file name, the file length, and the **URL of the tracker**,.
- **Tracker:** This is a centralized server responsible for **coordination**.
    - The user contacts the tracker to obtain a list of at least a few **peers** who are currently sharing that specific file,. The tracker usually does _not_ keep the file itself.
- **File Division and Integrity:**
    - The entire file is divided into **pieces** (typically 256 kilobytes).
    - These pieces are further divided into **sub-pieces** (typically 16 kilobytes).
    - Integrity is maintained by including the **root hash of the Merkle Tree** of the file within the torrent file.
    - To check if a piece has been tampered with, the user must download the data and the necessary sibling hashes (the Merkle proof) to verify it against the Merkle root.

### III. Download and Contribution Mechanisms

- **Peer Roles:**
    - **Leecher:** A user who is currently downloading the file.
    - **Seeder:** A user who has downloaded the entire file and remains in the system to upload pieces to others. This is considered a "good habit" to contribute to the community.
- **Piece Selection Strategy:** The strategy for choosing which pieces to download first is key:
    
    1. **Random Selection:** This is good for system **balancing**, ensuring all pieces are requested by peers simultaneously.
    2. **Rarest First:** This strategy is good for the **community** as it prioritises downloading pieces held by very few peers, preventing files from being "lost" if those rare peers disconnect.
    
    - **Implementation:** BitTorrent starts with a **Random** strategy to quickly acquire an initial set of pieces (allowing the new user to start uploading quickly), and then switches to **Rarest First**,.
- **Handling Free Riders:** A free rider is a participant who only downloads and refuses to upload.
    - **Tit-for-Tat (T4T):** This technique limits free riding by requiring that a user can only download a piece if they are simultaneously uploading a piece to the other peer,.
    - **Optimistically Unchoking:** Since a new user (leecher) starts with nothing to upload, the system must allow them to acquire their first pieces. This is done through the **unchoking algorithm**. Every minute (or period), the system optimistically unchokes (uploads to) a random peer, regardless of T4T, giving new users a chance to join and start contributing.
- **End Game Mode:** Near the end of the download (e.g., 98-99% complete), speed often slows down because the last remaining pieces are often held only by slow or distant peers,.
    - To solve this, when only the last piece (or few pieces) remain, the system enters **End Game Mode**, which allows the peer to download sub-pieces simultaneously from different users.

### IV. System Properties and Attacks

- **CAP Theorem Analysis:**
    - **Partition Tolerance:** Considered **pretty good**. The system can work even if the network is partitioned, provided there are seeders remaining in the partition.
    - **Consistency:** Not strictly well-defined because the system is primarily **read-only** (the file is not modified).
    - **Availability:** Not always guaranteed, but generally acceptable, depending on how common the file is and the number of active peers/seeders.
- **Attacks:** Corporations may fight the system by:
    - Becoming **seeders** themselves to log the IP addresses of downloaders.
    - Trying to **pollute the system** by distributing fake pieces (less effective due to hash checks) or distributing entirely fake files (polluting the entire file).
- **Modern Usage:** Although BitTorrent is less used by the public now, the underlying ideas—such as decentralized downloads and fetching content from multiple locations—are used by major corporations (like Apple or Microsoft) to distribute operating system updates quickly.