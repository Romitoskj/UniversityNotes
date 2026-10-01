# Class 2 Notes: RFID Architecture, MAC Protocols, and Autonomous Adaptation

### 1. Introduction to RFID Systems

The fundamental objective of any **RFID (Radio Frequency Identification)** system is **Object Identification**. This entails retrieving a unique identifier associated with a specific tag attached to an object (e.g., inventory, assets, or access cards) to facilitate tracking and management within a digital repository.

#### Component Synthesis

An RFID architecture is composed of three primary segments, each with distinct physical constraints and functional responsibilities.

|   |   |   |
|---|---|---|
|Component|Physical Characteristics|Functional Role|
|**RF Tags**|Passive, small-scale, and cost-effective. Consist of a microchip and an integrated antenna [SOURCE_IMAGE_5, 11].|Data carriers that store unique identifiers. They lack an internal power source for active transmission.|
|**Interrogators (Readers)**|High-power devices available in **Stationary (Fixed)** or **Handheld (Mobile)** form factors [SOURCE_IMAGE_8, 9].|Acts as the central controller. Queries tags to retrieve IDs and manages the communication link.|
|**Servers & Data Repositories**|Backend computational infrastructure and databases.|Processes raw data from the reader and maps IDs to specific application logic.|

#### Tag Data Specifications

Tags transmit unique bit-strings, typically utilizing a **96-bit** standard known as the **Electronic Product Code (EPC)**. Depending on the application requirements, identifier lengths can scale up to a maximum of **256 bits** [SOURCE_IMAGE_14].

### 2. Physical and Link Layer Characteristics

#### The RFID Channel

The communication medium in RFID systems presents several challenges for reliable network design:

- **Wireless and Shared:** Multiple tags and the reader occupy the same frequency spectrum, leading to interference.
- **Asymmetric Low Data Rate:** The typical **uplink** (tag-to-reader) data rate is constrained to **40 kbps** [SOURCE_IMAGE_13].
- **Collision Vulnerability:** If multiple tags reply to a query simultaneously, their signals overlap, resulting in a collision that renders the data unreadable.

#### Passive Backscatter Communication

Because RFID tags are **Passive**, they cannot initiate communication or transmit active radio signals. Instead, they respond via **Backscattering**: tags modulate and reflect the continuous wave signal transmitted by the reader.

#### Fundamental MAC Constraints

The Medium Access Control (**MAC**) design for RFID deviates significantly from protocols like Wi-Fi or Ethernet due to hardware limitations:

- **Lack of Sensing Capabilities:** Tags are incapable of "hearing" one another. There is **NO Carrier Sense (CS)** and **NO Collision Detection (CD)** on the tag side [SOURCE_IMAGE_15].
- **Half-Duplex Limitations:** Tags cannot support the power-intensive transceivers required to listen while transmitting.
- **Reader-Driven Arbitration:** Because tags cannot coordinate, the **Reader** must serve as the central arbiter, orchestrating channel access to resolve collisions.

### 3. Classification of Anti-Collision Protocols

To achieve **Singulation**—the process of identifying tags one by one—several **Sequential** protocols are employed [SOURCE_IMAGE_16]:

- **Tree-Based Protocols:** Deterministic algorithms that recursively split the tag population.
    - **Binary Splitting (BS)**
    - **Query Tree (QT)**
    - **Query Tree Improved** (and other variations).
- **Aloha-Based Protocols:** Probabilistic algorithms where tags choose random transmission slots.
    - **Framed Slotted Aloha (FSA)**
    - **EPC Gen Standard**
    - **Tree Slotted Aloha (TSA)** and **Binary Splitting Tree Slotted Aloha (BSTSA)**.

_Note:_ _**Concurrent**_ _protocols, which attempt to decode overlapping transmissions, are outside the scope of this course._

### 4. Tree-Based Anti-Collision Protocols

#### Binary Splitting (BS) Principles

**Binary Splitting** utilizes a virtual stack mechanism to isolate tags. This is managed via internal counters (**C**) within the tags.

**Algorithmic Logic:**

1. **Initialisation:** All tags set their internal counter **C = 0**.
2. **Query:** Only tags with **C = 0** are permitted to respond.
3. **Collision Handling:** If a collision occurs:
    - Replying tags generate a random binary number (**0 or 1**) and add it to their counter.
    - All silent tags (those with **C > 0**) increment their counter by one (**C = C + 1**).
4. **Success or Idle Slot:** If the slot is non-colliding:
    - All tags decrement their counter by one (**C = C - 1**). This effectively "pops" the next subgroup from the virtual stack, moving them to the root (**C = 0**) for the next query.

#### Query Tree (QT) Protocols

Unlike **BS**, which uses random numbers, **QT** uses **ID Prefixes**. The reader broadcasts a prefix; only tags matching that prefix respond.

- **Synthesis:** In scenarios with a uniform ID distribution, the tree structure of **QT** mirrors **BS**, as the population naturally splits into approximately equal halves at each bit-level branch.

### 5. Aloha-Based and Hybrid Protocols

#### Evolution to BSTSA

While basic **Aloha** variants like **FSA** organize access into frames and slots, they can struggle with high collision rates in dense environments. **Binary Splitting Tree Slotted Aloha (BSTSA)** is a hybrid approach that applies **BS** counter logic _within_ specific slots of an **Aloha** frame to resolve collisions more efficiently.

#### Performance Evaluation: The Transmission Time Model

To evaluate protocol performance, we must look beyond raw throughput and consider the ratio of productive vs. non-productive time:

- **Idle Slots:** Wasted time because no tags responded to the query.
- **Collision Slots:** Wasted time because multiple tags responded, resulting in unreadable data.
- **System Efficiency:** Defined as the ratio of **Successful Slots** (where exactly one tag is identified) to the total number of slots required to complete the singulation process.

### 6. Autonomous Adaptation and Reinforcement Learning

#### Dynamics of Tag Populations

In real-world deployments, the tag field is rarely static. We classify the population into three categories [SOURCE_IMAGE_20]:

1. **Staying Tags:** Resident within the reader range across multiple cycles.
2. **Arriving Tags:** New entrants to the field.
3. **Leaving Tags:** Tags exiting the identification range.

#### Adaptive BSTSA via Reinforcement Learning (RL)

Static protocols fail to maintain efficiency as these populations shift. **Autonomous Networking** solves this by enabling the system to **Observe, Decide, and Adapt** [SOURCE_IMAGE_1].

By integrating **Reinforcement Learning (RL)**, the reader acts as an intelligent agent. It monitors real-time channel feedback (the ratio of successes, idles, and collisions) to optimize:

- **Frame Size (L):** Adjusting the number of available slots to match the estimated tag population.
- **Splitting Depth:** Fine-tuning the recursion level in **BSTSA** to minimize collision duration.

### 7. Summary of Key Takeaways

- **Tree-based protocols** (e.g., **BS**, **QT**) are deterministic. They provide guaranteed identification but can be slower in very high-density, low-collision scenarios.
- **Aloha-based protocols** (e.g., **FSA**, **EPC Gen**) are probabilistic. They are highly efficient for general use but require dynamic adjustment to prevent "thrashing" during high collision periods.
- **Research Perspective:** The industry standard is moving toward **Hybrid/Autonomous** systems. Utilizing **RL** allows for a self-optimizing MAC layer that balances the deterministic nature of tree splitting with the flexibility of **Aloha** frames, ensuring high efficiency regardless of tag mobility patterns.