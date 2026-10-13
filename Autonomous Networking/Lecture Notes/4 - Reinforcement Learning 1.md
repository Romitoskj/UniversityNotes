### 1. Course Structure, Roadmap & Class Logistics

The **Autonomous Networking** curriculum addresses a fundamental shift in modern system design: transitioning from manually configured communication protocols to intelligent, self-governing networks that observe, decide, and adapt independently without human intervention. Rather than focusing on vehicular or robotic control, this course focuses strictly on making network-level operations—specifically Medium Access Control (MAC) and routing—fully autonomous.

To build this expertise systematically, the course is structured into three integrated parts. Crucially, **Part 1 and Part 2 run in parallel rather than sequentially**. Every new physical networking architecture introduces a unique operational challenge, and every reinforcement learning (RL) mechanism provides the exact mathematical tool required to solve it.

#### Course Architecture & Roadmap

|   |   |   |
|---|---|---|
|Course Part|Focus Area|Key Topics / Protocols Covered|
|**Part 1**|**Types of Networks**|RFID backscatter networks, Wireless Sensor Networks (WSNs), Drone networks, Internet of Things (IoT) networks, and Robophysical networks.|
|**Part 2**|**RL Basics**|Agent-environment interaction, scalar reward signals, exploration vs. exploitation dilemma, state representations, policies, value functions, multi-armed bandits, and Q-learning.|
|**Part 3**|**RL-Based Protocols**|Autonomous MAC protocols (who speaks, and when) and autonomous routing protocols (who forwards, and where), learned directly from network feedback.|

#### 📌 Key Administrative Milestone: Midterm Examination

- **Date & Time:** Friday, October 30th at 1:00 PM.
- **Setting & Structure:** Combined examination room setting covering both network protocol theory (Part 1) and reinforcement learning foundations (Part 2).
- **Coverage:** Evaluates theoretical mastery of protocol failure modes alongside formal RL mathematical concepts.

#### Project vs. Presentation Logistics

Students have two distinct pathways for fulfilling the practical portion of the course following the midterm exam:

- **Option A (In-Class Hands-On Project):**
    - Focuses on algorithm design, protocol implementation, and creative network adaptation.
    - Dedicated class hours are allocated directly within the course schedule, allowing teams to complete project milestones during lecture sessions without heavy outside overload.
    - Planning and group formation begin early in the semester.
- **Option B (Research Paper Presentation):**
    - Focuses on analyzing, presenting, and critiquing a state-of-the-art research paper in autonomous networking.
    - Formal student presentations are scheduled sequentially following the midterm exam.

Having mapped our administrative path, we turn directly to a concrete physical engineering challenge where traditional pre-programmed networking protocols completely breakdown, illustrating why dynamic reinforcement learning is strictly required.

### 2. Case Study: MAC Protocol Design in a Battery-Free Smart Home

Consider a smart home environment populated by a wide variety of interconnected heterogeneous devices, including smart video cameras, presence sensors, smoke detectors, light sensors, thermostats, smart meters, TV remotes, light switches, and game controllers.

**System Topology & Physical Constraint Layout**

- **Centralized RFID Reader (The Agent):** Powered reader node; arbitrates all channel access; issues energy queries per time slot.
- **Passive Backscatter Tags (Heterogeneous Devices):**
    - _Joystick / Game Controller:_ Battery-free, bursty high-volume traffic during active gameplay.
    - _Thermostat / Smoke Sensor:_ Battery-free, low-frequency periodic baseline; immediate burst during anomalies/fire.
    - _Presence Sensor:_ Battery-free, spontaneous/aperiodic data generated upon physical motion.
    - _Video Camera:_ Battery-free, continuous high-bandwidth streaming data.

#### Environment & Hardware Constraints

1. **Battery-Free Operation:** Smart home devices operate without batteries or active RF power amplifiers. They survive entirely on energy harvested from the environment or from the reader's incident radio frequency (RF) signal.
2. **Backscatter Communication:** Devices communicate strictly through passive backscatter—modulating and reflecting the RF carrier signal emitted by the centralized RFID Reader.
3. **Passive Initiation Constraint:** Because backscatter tags lack active transmitters and energy reserves, **tags cannot speak first**. They cannot initiate transmission or notify the network when new sensor measurements are ready. The centralized Reader alone must arbitrate channel access by explicitly querying individual tags.

#### Device Traffic Pace Comparison

Because each device serves a distinct physical function, their data generation patterns vary drastically across space and time:

|   |   |   |
|---|---|---|
|Device Type|Data Generation Pattern / Pace|Environmental Trigger / Context|
|**Joystick / Game Controller**|Bursty, high-volume traffic during gameplay; completely silent otherwise.|Triggered exclusively when a human user actively plays an interactive game.|
|**Temperature Sensor / Thermostat**|Low-frequency periodic sampling during normal states; immediate high-rate bursts during emergencies.|Triggered by routine thermal tracking or environmental anomalies (e.g., detecting a fire).|
|**Presence Sensor**|Spontaneous, aperiodic data generation.|Triggered only when a person physically walks past the doorway sensor.|
|**Video Camera**|Continuous, high-bandwidth data stream.|Continuous frame generation during active surveillance or streaming.|

#### Failure Analysis of Traditional MAC Protocols

The Reader operates in discrete time slots and can query exactly **one tag per slot**. When attempting to schedule queries across these dynamic devices, traditional MAC mechanisms fail fundamentally:

**Comparison of Traditional MAC Access Strategies**

- **Option 1: Random Access**
    - _Slot_ _t__:_ Tag 1, Tag 2, and Tag 4 attempt to backscatter simultaneously \longrightarrow **RF Channel Collision** (Transmissions corrupted; time slot completely wasted).
- **Option 2: Fixed Round-Robin Slotting**
    - _Slot 1:_ Query Tag 1 (Thermostat) \longrightarrow **Wasted Query** (No new thermal data generated).
    - _Slot 2:_ Query Tag 2 (Presence) \longrightarrow **Wasted Query** (No person walked by).
    - _Slot 3:_ Query Tag 3 (Joystick) \longrightarrow **Successful Read** (Sample 1 collected).
    - _Slot 4:_ Query Tag 4 (Camera) \longrightarrow **DATA LOSS** (Camera generated Sample 2 before query; Sample 1 overwritten in single-sample buffer and lost forever).

##### 1. Random Access Protocols

If tags attempt to transmit as soon as data is generated, multiple backscatter devices respond simultaneously within the same slot. Because backscatter tags cannot perform carrier sensing, severe **channel collisions** occur, corrupting the signal and wasting channel energy.

##### 2. Fixed Round-Robin / Time-Slotted Protocols

The Reader queries devices in a rigid, pre-fixed sequence (T_1 \to T_2 \to T_3 \to T_4 \to T_5 \to T_1 \dots). While fixed time-slotting completely eliminates collisions, it suffers from **three major systemic inefficiencies**:

1. **Wasted Queries:** The Reader spends valuable slots querying idle tags that have generated no new data since their last visit (e.g., 4 out of 5 queries yielding no new information).
2. **Data Loss / Overwriting:** Fast-producing tags (e.g., active joysticks or video cameras) generate a new sample (Sample 2) before the Reader returns. Because micro-tags feature minimal memory buffers holding only the most recent reading, unread measurements (Sample 1) are permanently overwritten and lost.
3. **Redundant Queries:** The Reader over-visits slow, static sensors, reading identical duplicate measurements repeatedly while ignoring active devices.

#### The Reinforcement Learning Solution

The **ideal MAC protocol** queries a device _only and immediately_ when it holds fresh, unread data, eliminating collisions, wasted queries, and sample overwrites.

However, because human behavior is inherently unpredictable—people play games, walk through rooms, or trigger emergency sensors at arbitrary times—**no static, pre-programmed schedule can achieve optimal performance**. The Reader must observe network feedback, learn the dynamic patterns of the household, and adapt its scheduling decisions online.

### 3. Reinforcement Learning Foundations & Interaction Framework

To solve sequential decision-making under uncertainty, we formalize the framework of **Reinforcement Learning (RL)**.

**Definition:** **Reinforcement Learning** is learning what to do—how to map situations to actions so as to maximize a cumulative numerical reward signal.

#### The Three Pillars of Reinforcement Learning

Every RL problem relies on three core operational capabilities:

1. **Sensation:** The capacity to sense the state of the environment (to some extent).
2. **Action:** The capacity to execute decisions that directly influence and alter the state of the environment.
3. **Goal:** A explicit objective related to the state of the environment, formalized via scalar reward feedback.

#### The Agent-Environment Interaction Loop

The interaction between the decision-maker and the physical world is modeled as a discrete-time dynamical system across steps t = 0, 1, 2, 3, \dots:

**The Discrete Agent-Environment Cycle**

- **Step 1:** The **Agent** (Reader scheduling algorithm) evaluates its internal state and executes **Action** **A_t** (e.g., "Query Tag 3").
- **Step 2:** The **Environment** (Smart Home, physical tags, human actions) receives A_t, transitions its internal state, and emits:
    - An **Observation** **O_t** (e.g., decoded tag payload: fresh sample, duplicate, or overwrite flag).
    - A scalar **Reward** **R_t \in \mathbb{R}** evaluating the utility of action A_t.
- **Step 3:** Discrete time increments to t+1, and the loop repeats continuously.

#### Key RL Characteristics

Unlike offline supervised learning, RL operates under three defining characteristics:

- **Trial-and-Error Search:** The agent is never given explicit ground-truth labels indicating the "correct" action. It must actively discover effective strategies through trial-and-error experience.
- **Online Continual Learning:** Learning occurs concurrently with execution. The agent does not train on a static offline dataset; it continuously adapts its strategy while interacting with an evolving world.
- **Delayed Rewards:** Current actions impact immediate scalar rewards as well as future environmental states and subsequent downstream returns.

### 4. Reward Design & The Reward Hypothesis

The primary feedback mechanism guiding an RL agent is the scalar reward signal.

#### 📜 The Reward Hypothesis

**All goals can be described by the maximization of expected cumulative reward.**

The agent's objective is not merely to maximize immediate feedback R_{t+1}, but to maximize the total cumulative reward accumulated over its entire operational lifetime.

#### Formulating Reward Signals in Engineering Systems

In physical engineering domains, environment models do not automatically provide numerical reward signals. A core insight emphasized in protocol design is that **the engineer is entirely responsible for crafting these scalar reward values through empirical trial and error**. Tuning reward structures is a fundamental design freedom used to align mathematical optimization with target system behavior.

For our battery-free smart home MAC protocol, we establish the following engineer-crafted reward formulation based on query outcomes:

|   |   |   |
|---|---|---|
|Smart Home Network Event|Numerical Reward Value|Pedagogical / Engineering Rationale|
|**Successful Query** _(New Sample Read)_|+1|**Positive Reinforcement:** Directly rewards the Reader for efficiently harvesting fresh, unread sensor data.|
|**Wasted Query** _(Duplicate / No New Data)_|-1|**Minor Penalty:** Penalizes the Reader for wasting a channel time slot that could have served an active node.|
|**Data Loss Event** _(Sample Overwritten Before Read)_|-5|**Severe Penalty:** Imposes a heavy negative penalty to teach the agent that missing high-frequency devices causes irreversible data loss.|

_Note: As protocol designers, if we determine that data loss is completely intolerable in a safety-critical setting (e.g., smoke detection), we can manually adjust the overwrite penalty from_ _-5_ _to_ _-20_ _to force the agent to prioritize high-rate sensors._

### 5. The Exploration vs. Exploitation Dilemma

Because an RL agent starts with zero prior knowledge of environment dynamics, it faces a fundamental trade-off: **The Exploration-Exploitation Dilemma**.

- **Exploitation:** Selecting actions known from past experience to yield high rewards, maximizing short-term performance.
- **Exploration:** Selecting unfamiliar or non-optimal actions to gather new information, risking poor immediate feedback to discover potentially superior long-term strategies.

#### 🍽️ Analogy: The Restaurant Selection Problem

- **The Usual Place (Exploitation):** You eat at your favorite diner every day. You have high confidence in the food quality (+2 reward), but you completely forgo the opportunity to discover an exceptional dining spot.
    - _Smart Home Network Mapping:_ **Exploitation** corresponds to repeatedly querying the known active video camera tag to maintain a steady stream of +1 rewards.
- **The Grand Opening (Exploration):** You try a newly opened restaurant around the corner. You face incomplete information: the food may be terrible (-5 reward), average (+1 reward), or extraordinary (+10 reward).
    - _Smart Home Network Mapping:_ **Exploration** corresponds to intentionally querying an unvisited thermostat or presence sensor tag to discover if an emergency or human activity burst has suddenly begun.

#### Handling Stochastic Environments

In stochastic environments—where query outcomes fluctuate randomly—taking an action a single time is insufficient to evaluate its true value. A single query to a presence sensor might yield no data simply because a person paused for two seconds. Consequently, the agent must execute the **same action multiple times** across diverse contexts to establish a reliable expected reward estimate.

#### The Strategic Exploration Recipe

1. **Early Learning Phase:** Prioritize **high exploration** when the environment is completely unknown to build accurate baseline reward estimates.
2. **Maturation Phase:** Shift toward **high exploitation** as empirical reward estimates stabilize, leveraging high-performing actions.
3. **Ongoing Minimal Exploration:** Maintain a permanent baseline rate of exploration. Because human home activity is non-static (e.g., a user suddenly picking up a game controller), pure exploitation causes the agent to rely on outdated assumptions.

### 6. History, State Representation, and Environment Observability

To select optimal actions, an agent must process its past interaction history with the environment.

#### Formal History Definition

The **History (****H_t****)** is the complete chronological sequence of all past actions, observations, and scalar rewards recorded up to time step t:

H_t = A_1, O_1, R_1, A_2, O_2, R_2, \dots, A_t, O_t, R_t

#### Why Memory Compression is Mandatory in Embedded Systems

Because H_t grows infinitely over time, an embedded RFID Reader cannot store or process the raw history directly. Real-time embedded systems operate under strict memory constraints (bounded RAM) and rigid latency bounds per time slot. An algorithm cannot scan an endlessly expanding historical log to select a query target within a sub-millisecond slot deadline.

Therefore, the agent must compress raw history into a compact, fixed-size summary vector called the **Agent State (****S_t****)**:

S_t = f(H_t)

#### Environment State vs. Agent State

- **Environment State (****S^e_t****):** The environment's private, complete representation containing all physical internal variables (e.g., exact buffer contents inside all smart home tags).
- **Agent State (****S^a_t****):** The agent's internal representation computed via its state function S^a_t = f(H_t), containing only the compressed information retained to make decisions.
- **Full vs. Partial Observability:** In fully observable domains (S^a_t = S^e_t), the agent inspects the true environment state directly (e.g., chess). The **battery-free smart home is partially observable** (S^a_t \neq S^e_t): the Reader cannot see inside tag buffers and learns tag status only _after_ executing a query.

#### Impact of State Representation: The Rat Example

How an agent constructs its state function f(H_t) dictates what patterns it can learn. Consider a classic psychological experiment where a rat observes sequences of visual/auditory stimuli ending in either a **Shock** or **Food**:

- **Episode 1:** Light \to Light \to Lever \to Bell \implies **Shock**
- **Episode 2:** Bell \to Light \to Lever \to Lever \implies **Food**
- **Episode 3 (Test Case):** Lever \to Light \to Lever \to Bell \implies **?**

Predicting the outcome of Episode 3 depends entirely on the agent's choice of state function f(H_t):

|   |   |   |   |
|---|---|---|---|
|State Function f(H_t) Strategy|Evaluated Agent State for Episode 3|Resulting Prediction for Episode 3|Pedagogical Explanation|
|**1. Last 3 Items Sequence**|`[Light, Lever, Bell]`|**Shock**|Episodes 1 and 3 share the exact same trailing 3-item sequence: Light \to Lever \to Bell.|
|**2. Aggregate Item Counts**|`1 Light, 1 Bell, 2 Levers`|**Food**|Episodes 2 and 3 contain identical total event counts, ignoring chronological sequence order.|
|**3. Complete History Match**|`Lever -> Light -> Lever -> Bell`|**No Prediction / Unknown**|The exact full chronological sequence has never been observed before.|

#### Candidate Smart Home State Representation

A practical state representation for the RFID Reader tracks a compact summary tuple per tag i:

S_t = \Big\{ \big( \Delta t_i, \text{Outcome}_i \big) \;\Big|\; i \in \{1, \dots, N\} \Big\}

Where:

- \Delta t_i: The number of time slots elapsed since tag i was last queried.
- \text{Outcome}_i: The feedback received during the last query to tag i (\text{Success}, \text{Empty}, or \text{Overwrite}).

### 7. Architecture of an RL Agent: Policy, Value Function, and Model

An RL agent is composed of three core internal components: Policy, Value Function, and Model.

#### Component 1: Policy (\pi)

The **Policy** defines the agent's behavior at any given time, mapping states to actions:

- **Deterministic Policy:** Direct functional mapping \pi(s) = a.
    - _Example:_ "Always query whichever tag has been waiting the longest."
- **Stochastic Policy:** Conditional probability distribution mapping actions to states:

\pi(a \mid s) = \mathbb{P}[A_t = a \mid S_t = s]

- _Example:_ "Query the tag waiting longest with P=0.9; query any other random tag with P=0.1."

Learning occurs when scalar rewards modify policy probabilities, favoring high-reward actions over time.

#### Component 2: Value Function (v_\pi(s))

While the immediate reward signal R_{t+1} evaluates immediate action quality, the **Value Function** **v_\pi(s)** measures long-term desirability.

##### The Discounted Return (G_t)

The total discounted return G_t accumulated from step t onward is defined as:

G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \dots = \sum_{k=0}^{\infty} \gamma^k R_{t+1+k}

Where \gamma \in [0, 1] is the discount factor. Near-term rewards are weighted more heavily than distant future rewards because near outcomes are easier to control and predict in stochastic systems.

##### State Value Definition

The state value v_\pi(s) is the expected return starting from state s under policy \pi:

v_\pi(s) = \mathbb{E}_\pi \big[ G_t \mid S_t = s \big]

##### Step-by-Step Value Propagation Trace (\gamma = 0.9)

To illustrate how value functions capture long-term returns despite zero immediate rewards, consider a 3-state linear transition chain (s_1 \to s_2 \to s_3):

**3-State Return Propagation Chain**

- **State** **s_3** **(Target State):** Yields immediate reward R_{t+3} = +10. Terminal value v(s_3) = 10.0.
- **State** **s_2** **(Intermediate State):** Yields immediate reward R_{t+2} = 0. Value v(s_2) = 0.9 \times 10.0 = 9.0.
- **State** **s_1** **(Initial State):** Yields immediate reward R_{t+1} = 0. Value v(s_1) = 8.1.

**Step-by-Step Mathematical Expansion for State** **s_1****:** Starting at s_1 at step t, the sequence of future rewards emitted by the environment across subsequent transitions is R_{t+1} = 0, R_{t+2} = 0, and R_{t+3} = 10. Expanding the discounted return formula explicitly:

G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3}

G_t = 0 + (0.9 \times 0) + (0.9)^2 \times 10

G_t = 0 + 0 + (0.81 \times 10) = 8.1

v_\pi(s_1) = \mathbb{E}_\pi[G_t \mid S_t = s_1] = 8.1

**Key Takeaway:** State s_1 yields **zero immediate reward** (R_{t+1} = 0), but possesses a **high long-term value (****8.1****)** because it guarantees path progression toward state s_3.

#### Component 3: Model & Model-Free Learning

A **Model** predicts environment transitions and expected immediate rewards. Because human household dynamics are far too complex to model explicitly, we adopt a **Model-Free RL** approach—learning optimal policies and value functions directly from interaction feedback without modeling human habits.

### 8. Synthesis: Formulating the Smart Home as a Formal RL Problem

To apply reinforcement learning to autonomous networking, we map every physical entity and operational parameter of the battery-free smart home directly into formal mathematical RL concepts:

|   |   |   |
|---|---|---|
|RL Theoretical Concept|Smart Home System Physical Equivalent|Formal Mathematical Notation|
|**Agent**|The centralized scheduling algorithm executing within the RFID Reader.|Central decision-maker|
|**Environment**|The smart home system (all physical tags, human activities, and RF channel dynamics).|The external world|
|**Action (****A_t****)**|Selecting a specific battery-free device i \in \{1, 2, \dots, N\} to query in time slot t.|A_t = i|
|**Observation (****O_t****)**|The decoded tag response returned to the Reader.|O_t \in \{\text{New Sample}, \text{Duplicate Data}, \text{Overwrite Flag}\}|
|**Reward (****R_t****)**|Engineer-crafted scalar feedback evaluating query utility.|R_t = \begin{cases} +1 & \text{if } O_t = \text{New Sample} \\ -1 & \text{if } O_t = \text{Duplicate Data} \\ -5 & \text{if } O_t = \text{Overwrite Flag} \end{cases}|
|**Agent State (****S_t****)**|Compressed summary vector per tag tracking elapsed time and last outcome.|S_t = \{(\Delta t_i, \text{Outcome}_i)\}_{i=1}^N|
|**Policy (****\pi****)**|Scheduling decision rule mapping tag states to query actions.|\pi(a \mid s) = \mathbb{P}[A_t = a \mid S_t = s]|
|**Goal**|Maximize cumulative discounted return across time slots.|\max \mathbb{E} \left[ \sum_{k=0}^{\infty} \gamma^k R_{t+1+k} \right]|

#### 💡 Discussion & Technical Review Prompts

1. **Model Complexity:** Why is it practically impossible to pre-compute an explicit environment transition probability matrix \mathbb{P}(S_{t+1} \mid S_t, A_t) for a smart home setting prior to deployment?
2. **Reward Sensitivity:** If you increase the penalty for sample overwriting from R_t = -5 to R_t = -20, how will the agent's learned policy adjust its sampling frequency for high-bandwidth devices (video cameras) versus low-rate sensors (thermostats)?
3. **Policy Selection:** Under what specific traffic generation conditions would a deterministic policy (\pi(s) = a) perform strictly better than a stochastic policy (\pi(a \mid s))?