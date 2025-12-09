# Cryptographic Foundations

## Hash Functions
> These functions take variable-length data (like a file) and output a fixed, typically short, hash or digest

They are crucial because they possess two key properties: 
- it is very difficult to find the original data from the hash (preimage resistance)
- it is very hard to find a different file that produces the same hash (collision resistance). 

Examples of hash functions include SHA-256. They are used to verify data integrity, such as checking if two files are the same without exchanging the files themselves
## Public Key Cryptography
> This asymmetric encryption system uses two connected keys, a **private key** (kept secret) and a **public key** (which can be widely published)

**Signatures:** To digitally sign a message, the owner uses their **private key** to encrypt the hash of the message
Anyone can use the corresponding **public key** to decrypt the signature and verify its authenticity

## Merkle Tree
> This data structure is used to prove that a specific data record (D) belongs to a larger collection

By hashing individual records and then hashing pairs up to a **root of the tree**, one only needs the root hash (H) and a small **Merkle proof** (a sequence of hashes along the path) to verify D's inclusion. The size of this proof is **logarithmic** (logN), making it highly efficient

---
# Bitcoin and the Double Spending Problem

Bitcoin is a digital cash system, functioning similarly to a signed check.

A transaction transfers ownership of a value.

The "name" of the owner in Bitcoin is actually their **public key**, and the transfer is secured using a **digital signature** created with the private key.

• A transaction references the hash of the previous transaction (input) and specifies the new owner(s) (output) and the values transferred

. An owner can send a portion of a bitcoin to a new owner and send the remainder back to themselves.

## Double spending problem
The central challenge for any digital cash system is the **double spending** problem: the ability to spend the same coin twice. To prevent this, Bitcoin requires a system where all transactions are stored and checked for prior spending. Since the creators desired a decentralized system, this ledger must be managed by a **distributed system** rather than a single central server.

## Consensus, Blocks, and Proof of Work

The Bitcoin network must achieve **consensus** on which transactions are committed

. Since traditional consensus protocols like Paxos are not suitable for large, dynamic networks due to high message overhead and malicious nodes (Bzantine faults)

, Bitcoin uses a different approach:

1. **Blocks and Blockchain:** Consensus is run not on individual transactions, but on a **collection of transactions** called a **block**

. A new block is perpetually linked to its predecessor by including the hash of the previous block in its header, forming the **blockchain**

.

2. **Block Structure:** A block header contains the hash of the previous block, the **Merkle root** of all its transactions (used for verification), and a crucial value called the **nonce**

.

3. **Proof of Work (PoW):** For a block to be valid, the hash of the entire block must meet a specific difficulty requirement, such as having the K **least significant bits set to zero**

. The probability of this is 1/2K

.

    ◦ Miners repeatedly try different **nonce** values until they find a valid hash, a process that requires a substantial amount of computational effort

. This is called **Proof of Work**

.

4. **Incentives:** The first miner to find a valid block gets to commit that block and is rewarded with newly **minted bitcoins** (created in the first transaction of the block)

. This is why these actors are called **miners**

.

5. **Forks and Safety:** If two miners find valid blocks simultaneously, the chain splits into two branches (**forks**)

. The consensus rule is that the **longest chain prevails**. Miners stop working on shorter branches. The system is calibrated so that a new block is produced roughly **every 10 minutes**. A transaction is typically considered permanently committed only after **six new blocks** have been added to its branch, which can take about an hour

.

6. **Safety and Liveness:** The Bitcoin protocol is considered **safe** and **live with high probability**

.

Drawbacks

The main disadvantages of this PoW-based consensus mechanism are the **very high power consumption** required to perform the computational work

and the lack of speed, as transactions can take up to an hour to be fully committed.