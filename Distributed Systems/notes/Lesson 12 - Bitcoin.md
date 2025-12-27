Bitcoin was introduced in a 2008 paper by an anonymous author known as **Satoshi Nakamoto**. The goal was to create **digital cash** independent of government control, rooted in the libertarian movements of the time. The system is built on two pillars: **cryptography** (considered the "easy" part) and **consensus** (the "hard" part).

### I. Cryptography

1. **Hash Functions:**
    
    - Functions that convert variable-length data into a short, fixed-length number called a **digest** or hash.
    - **Key Properties:** It is computationally difficult to find the original data from a hash (pre-image resistance) or to find two different files that produce the same hash.
    - **Examples:** SHA-256 (used in Bitcoin) and the older, weaker MD5.
    - **Use Case:** Verifying file integrity without exchanging the entire file.
	
2. **Public Key Cryptography & Digital Signatures:**
    
    - **Asymmetric Encryption:** Uses two mathematically linked keys: a **public key** (shared openly) and a **private key** (kept secret).
    - **Signatures:** To sign a message, you encrypt its hash with your private key. Anyone with your public key can decrypt it to verify the message came from you and has not been tampered with.
    - **Identity:** In Bitcoin, a user's **public key serves as their "name"** or address.
	
3. **[Merkle Trees](Notes%20-%20Merkle%20Trees.md):**
    
    - A structure used to prove a record is part of a large collection without needing the whole collection.
    - **Process:** Records are hashed in pairs repeatedly until a single **root hash** is reached.
    - **Merkle Proof:** To prove a transaction is in a block, one only needs the a sequence of "brother" hashes ($O(\log n)$ size) to reconstruct the path to the root.
	
    ![|500](../assets/Pasted%20image%2020251226201105.png)

### II. Consensus

Traditional consensus algorithms like Paxos assume only **benign failures**, while Bitcoin must tolerate **Byzantine Failures**, where participants may act maliciously. Moreover Paxos broadcasts to every node and this is not sustainable in a system with billions of users, so this problem is solved using best effort based communications. Specifically the updates are sent through the **gossip protocol**: node send message to a few number of other nodes and they will do the same till the whole system received the message.
#### A. The Double-Spending Problem

- **Definition:** Unlike physical cash, digital files can be easily duplicated. **Double-spending** occurs if a user tries to spend the same digital coin twice.
- **The Decentralisation Challenge:** Traditional systems solve this with a **trusted central server** (like a bank) to track all transactions. Bitcoin aims to replace this central authority with a **distributed system** of thousands of nodes that must reach ***consensus*** on which transactions are valid.

#### B. Transactions and Blockchain Structure

- **Transactions:** Function like digital checks. They contain:
	- **Hash of the previous transaction**
	- **New owner's public key**
	- **Previous owner signature** (created with its private key), used to verify the transaction (with the previous owner public key)
	- **BTC amount**
	
	![](../assets/Pasted%20image%2020251226153512.png)
	
	To allow value to be split and combined, transactions contain multiple inputs and outputs. Normally there will be either a single input from a larger previous transaction or multiple inputs combining smaller amounts, and at most two outputs: one for the payment, and one returning the change, if any, back to the sender.
	
	![](../assets/Pasted%20image%2020251226201529.png)
	
- **Blocks:** To improve performance, the network does not run consensus on every individual transaction. Instead, transactions are grouped into **blocks**  and **hashed in a Merkle Tree**. The root of the tree is stored in the **block header**, so that nodes does not need to store whole blocks but can still verify that a transaction belong to a block having the Merkle-proof.
	
	![|300](../assets/Pasted%20image%2020251226201645.png)
	
- **Blockchain:** Each block contains the **hash of the previous block** in the header. This creates a chain where any modification to a past block would invalidate all subsequent blocks.
	
	![](../assets/Pasted%20image%2020251226195745.png)

#### C. Proof-of-Work (Mining) to reach consensus

- **The Mining Process:** For a block to be considered valid, its total hash must meet a specific difficulty requirement (e.g., the last $k$ bits must be zero).
- **Nonce:** Miners repeatedly change a variable called a **nonce** (contained in block header) and re-hash the block until they find a valid hash by chance.
- **Proof-of-Work (PoW):** Finding a valid hash serves as proof that the miner performed a vast amount of computational work.
- **Incentives:** The first transaction in every block (the **coinbase**) grants the miner newly created Bitcoin. This reward halves approximately every four years (e.g., from 50 BTC in 2009 to 25, 12.5, etc.), converging toward a **fixed supply of 21 million BTC**.

#### D. Security

- **Mine a block is hard:** $\frac{1}{2^k}$ ​probability to compute a correct hash. For $k = 30$  the probability is $\frac{1}{1B}$.
- **Modify old transaction:** If an attacker wants to modify an old transaction he would need to **re-mine all the subsequent blocks** to convince other peers that that is the true blockchain, this is very hard.
- **Mine next malicious block:** an attacker still cannot mine the next block because the sign of the previous owner is needed

#### E. Forks

- It can happen that two miners simultaneously mine the next block (they are different blocks that contains different transactions or the same ones in a different order).
- These blocks are broadcasted with the gossip protocol so roughly $\frac{1}{2}$ of the miners will work on a branch and the other half on the other
- Eventually one branch will grow faster than the other: then the other is discarded. Transactions in the other branch will be put in the remaining one.
- As a rule of thumb a branch is confirmed to be the right one after other six blocks are added to it.

### III. System Dynamics and Limitations

- **Forks:** If two miners find a block simultaneously, the chain **forks** and the work made on one of the branch is wasted.
- **Confirmation Time:** The system adjusts difficulty to ensure a new block is found roughly every **10 minutes**. A transaction is generally considered secure only after **six blocks** (roughly one hour) have been added to the chain.
- **Drawbacks:**
    - **Energy Consumption:** Mining requires massive power, leading to super-clusters of computers in cold climates for cooling.
    - **Latency:** It is relatively slow compared to centralized systems.
    - **Security Risk:** If one entity controls more than **50% of the hashing power**, the decentralised nature of the system is compromised.

>[!success] Safety
>Manage to mine a block means that the transactions in it are the next valid ones so the systems **reached consensus** on these new transactions, that are appended to the chain and transmitted to everybody the block with gossip protocol.
>
>Moreover all nodes in the blockchain only trust **the longest branch**.

>[!success] Liveness
> Two branches can keep grow simultaneously forever but the probability decreases over time (everybody wants to add to the longest chain to not waste their work), so it is live.
> 
>Moreover since is based on best effort communication and node flexibility (a node can leave and re-join the system) it continues to make progress without stalls.

***Bitcoin is a non-deterministic consensus protocol both live and safe.***