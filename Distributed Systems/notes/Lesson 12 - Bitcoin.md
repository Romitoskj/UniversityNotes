### I. Introduction to Bitcoin

- **Origins:** Bitcoin was introduced in a 2008 paper by an anonymous author known as **Satoshi**.
- **Purpose:** The goal was to create **digital cash** independent of government control, rooted in the libertarian movements of the time.
- **Core Ingredients:** The system is built on two pillars: **cryptography** (considered the "easy" part) and **consensus** (the "hard" part).

### II. Essential Cryptographic Tools

The lesson outlines three primary tools used to secure the network:

1. **Hash Functions:**
    
    - Functions that convert variable-length data into a short, fixed-length number called a **digest** or hash.
    - **Key Properties:** It is computationally difficult to find the original data from a hash (pre-image resistance) or to find two different files that produce the same hash.
    - **Examples:** SHA-256 (used in Bitcoin) and the older, weaker MD5.
    - **Use Case:** Verifying file integrity without exchanging the entire file.
2. **Public Key Cryptography & Digital Signatures:**
    
    - **Asymmetric Encryption:** Uses two mathematically linked keys: a **public key** (shared openly) and a **private key** (kept secret).
    - **Signatures:** To sign a message, you encrypt its hash with your private key. Anyone with your public key can decrypt it to verify the message came from you and has not been tampered with.
    - **Identity:** In Bitcoin, a user's **public key serves as their "name"** or address.
3. **Merkle Trees:**
    
    - A structure used to prove a record is part of a large collection without needing the whole collection.
    - **Process:** Records are hashed in pairs repeatedly until a single **root hash** is reached.
    - **Merkle Proof:** To prove a transaction is in a block, one only needs a sequence of "brother" hashes ($O(\log n)$ size) to reconstruct the path to the root.

### III. The Double-Spending Problem

- **Definition:** Unlike physical cash, digital files can be easily duplicated. **Double-spending** occurs if a user tries to spend the same digital coin twice.
- **The Decentralisation Challenge:** Traditional systems solve this with a **trusted central server** (like a bank) to track all transactions. Bitcoin aims to replace this central authority with a **distributed system** of thousands of nodes that must reach consensus on which transactions are valid.

### IV. Transactions and Blockchain Structure

- **Transactions:** Function like digital checks. They consist of **inputs** (references to previous transactions where the money came from) and **outputs** (the new owner's public key and the value being transferred).
- **Blocks:** To improve performance, the network does not run consensus on every individual transaction. Instead, transactions are grouped into **blocks**.
- **Blockchain:** Each new block contains the **hash of the previous block**. This creates a chain where any modification to a past block would invalidate all subsequent blocks.

### V. Consensus and Proof-of-Work (Mining)

- **The Mining Process:** For a block to be considered valid, its total hash must meet a specific difficulty requirement (e.g., the last $k$ bits must be zero).
- **Nonce:** Miners repeatedly change a variable called a **nonce** and re-hash the block until they find a valid hash by chance.
- **Proof-of-Work (PoW):** Finding a valid hash serves as proof that the miner performed a vast amount of computational work.
- **Incentives:** The first transaction in every block (the **coinbase**) grants the miner newly created Bitcoin. This reward halves approximately every four years (e.g., from 50 BTC in 2009 to 25, 12.5, etc.), converging toward a **fixed supply of 21 million BTC**.

### VI. System Dynamics and Limitations

- **Forks:** If two miners find a block simultaneously, the chain **forks**. The network resolves this by following the **longest chain**, which represents the most cumulative work.
- **Confirmation Time:** The system adjusts difficulty to ensure a new block is found roughly every **10 minutes**. A transaction is generally considered secure only after **six blocks** (roughly one hour) have been added to the chain.
- **Drawbacks:**
    - **Energy Consumption:** Mining requires massive power, leading to super-clusters of computers in cold climates for cooling.
    - **Latency:** It is relatively slow compared to centralized systems.
    - **Security Risk:** If one entity controls more than **50% of the hashing power**, the decentralised nature of the system is compromised.

---

**Analogy for Proof-of-Work:** Imagine a global lottery where the only way to get a ticket is to spend energy running a treadmill. The "winning" ticket is a number that starts with ten zeros. You cannot predict which number you will get; you just have to keep running and checking your tickets. When someone finally shouts that they have won, everyone else can easily see the ten zeros on the ticket and agree that the winner must have put in the hard work to find it.