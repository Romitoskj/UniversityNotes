1. Case Study: Battery-Free Smart Home & MAC Protocol Challenges

1.1 Smart Home Architecture & Battery-Free RFID Backscatter

The primary physical deployment studied in this course is an autonomous smart home ecosystem operating entirely without conventional batteries. All installed sensing, actuation, and user-interface nodes are battery-free devices that rely on ambient/RF energy harvesting and **radio frequency identification (RFID) backscatter technology** for data transmission.

In this architecture, a centralized **Reader** acts as the primary network controller and medium access arbitrator. The device **tags** possess no active radio frequency (RF) transmitters or local power supplies (such as lithium cells). Instead, tags operate passively: when the Reader emits a continuous RF carrier wave, the passive tags harvest energy directly from the incident electromagnetic field to power their onboard microcontrollers and low-power sensors. To transmit data back to the Reader, a tag dynamically modulates its antenna input impedance, thereby altering the reflection coefficient and reflecting (**backscattering**) the incident RF wave back to the Reader.

Because tags cannot energize their microcontrollers or modulate the channel without an incident RF wave, all network communications are strictly synchronous and **Reader-initiated**.

```
                                +-------------------+
                                |  Central Reader   |
                                | (Network Master)  |
                                +-------------------+
                                  /       |       \
               RF Carrier Wave   /        |        \  RF Carrier Wave
               & Query Signal   /         |         \ & Query Signal
                               v          v          v
                       +----------+  +----------+  +----------+
                       | Tag 1    |  | Tag 2    |  | Tag 3    |
                       | Sensor A |  | Sensor B |  | Sensor C |
                       +----------+  +----------+  +----------+
                               \          |          /
                                \         |         /  Backscattered
                                 v        v        v   Modulated Signal
                                +-------------------+
                                |  Central Reader   |
                                | (Signal Receiver) |
                                +-------------------+
```

The smart home deployment integrates a heterogeneous set of hardware components categorized by their operational roles:

- **Smart Video Camera**: Captures continuous visual data streams for real-time indoor security surveillance.
- **Presence Sensor**: Detects human spatial movement across physical boundaries (e.g., doorways or hallways).
- **Thermostat / Temperature Sensor**: Monitors ambient thermal conditions and tracks microclimate shifts.
- **Smoke Sensor**: Continuously samples atmospheric particulate densities for emergency fire detection.
- **Light Sensor**: Measures ambient indoor illumination levels to automate lighting controls.
- **Smart Meter**: Monitors real-time electrical power consumption metrics across home appliances.
- **TV Remote**: Transmits asynchronous user control signals to multimedia systems.
- **Light Switch**: Captures discrete human actuation events for room illumination.
- **Joystick / Game Controller**: Transmits high-density interactive input sequences during active gameplay sessions.

1.2 Heterogeneous Traffic Patterns Across IoT Devices

Data production across the smart home is governed by local environmental phenomena and human physical interactions. Consequently, devices exhibit extreme heterogeneity in data generation pacing, burstiness, and temporal stability.

|Device Type|Traffic Generation Pattern|Environmental / User Trigger|Data Pace|
|---|---|---|---|
|**Smart Video Camera**|Continuous Streaming|Active surveillance mode|High-frequency continuous stream across sequential slots.|
|**Joystick / Game Controller**|Bursty / Interactive|Active gameplay by a user|Dense bursts of high-frequency data during gameplay; completely silent otherwise.|
|**Temperature Sensor / Thermostat**|Periodic / Threshold-driven|Ambient thermal changes or fire emergency|Low-frequency periodic intervals during normal state; shifts to high-frequency bursts during thermal anomalies (e.g., a fire).|
|**Presence Sensor**|Asynchronous / Event-driven|Physical movement through monitored space|Spontaneous, intermittent data generation only when an occupant crosses the sensor zone.|

1.3 The Core Operational Constraint: Reader-Initiated Communication

The physical layer architecture of battery-free RFID backscatter enforces an absolute operational boundary: **tags cannot speak first**. Lacking onboard batteries and active transmitters, tags remain functionally inert until illuminated by the Reader's continuous RF carrier signal.

Time across the network is divided into discrete, fixed-duration slots ($t = 1, 2, 3, \dots$). In each time slot $t$, the central Reader executes channel arbitration by transmitting an explicit query targeted at exactly one device tag $i$. The selected tag utilizes the incident RF wave during that slot to power its circuit and backscatter its buffered measurement.

**The Core Constraint: Reader-Initiated Arbitrations** Battery-free backscatter tags are completely passive and incapable of initiating asynchronous channel access. The central Reader must explicitly query a specific tag to provide the RF energy needed for backscattering. Consequently, channel scheduling is governed entirely by the Reader's query sequence across discrete time slots ($t$).

1.4 Critical Evaluation of Traditional MAC Protocols

Standard Medium Access Control (MAC) protocols engineered for active, battery-powered networks fail when deployed in reader-driven, battery-free backscatter environments.

Option 1: Random Access Protocols

In conventional random access schemes (such as Slotted ALOHA variants), tags containing unread data attempt to backscatter immediately upon detecting a general Reader query. When multiple tags hold data simultaneously, their backscattered signals interfere at the Reader's receiver interface. This results in **channel collisions**, preventing the decoding of any payload. Wasted slots reduce overall network throughput and expend reader transmit power without retrieving useful state information.

Option 2: Fixed Round-Robin / Slotted Assignment Protocols

Fixed assignment schemes avoid collisions by establishing a rigid, cyclic query sequence (e.g., $T_1 \to T_2 \to T_3 \to T_4 \to T_5 \to T_1 \dots$). While deterministic scheduling eliminates multi-tag contention, it introduces three severe operational failure modes in heterogeneous IoT environments:

- **(a) Wasted Queries**: The Reader queries tags whose memory buffers contain no new data since their prior visit. Channel slots that could have served active nodes are expended on empty responses.
- **(b) Loss of Data / Overwriting**: Passive backscatter tags feature minimal onboard memory (often restricted to a single measurement register). High-frequency devices (e.g., streaming cameras or active joysticks) generate new data samples at a rate faster than the round-robin cycle returns to query them. Consequently, Sample 2 arrives and **overwrites** Sample 1 in the tag's local register before Sample 1 is ever read by the Reader, resulting in unrecoverable data loss.
- **(c) Redundant Queries**: The Reader over-visits slow or currently inactive devices (e.g., an unused game controller or a temperature sensor in a thermally stable room). The Reader receives repeated identical measurements, yielding high transmission redundancy and inefficient channel utilization.

```
Round-Robin Pitfall 1: Wasted Queries
Slot t:    [ Query T1 ] ----> Response: Empty Buffer  (Wasted Slot)
Slot t+1:  [ Query T2 ] ----> Response: Empty Buffer  (Wasted Slot)

Round-Robin Pitfall 2: Buffer Overwrite & Data Loss (Tag T5)
Time t:    Tag T5 generates Sample 1 (Stored in 1-sample register)
Time t+1:  Tag T5 generates Sample 2 (Sample 1 is OVERWRITTEN and lost)
Slot t+k:  [ Query T5 ] ----> Reader receives Sample 2 only
```

1.5 The Ideal MAC Protocol Objectives

To maximize channel efficiency in a dynamic smart home, an autonomous MAC protocol must achieve four primary operational objectives:

1. **Selective Querying**: Execute a query to tag $i$ _only_ at the precise instant when tag $i$ has a new, unread sample ready in its memory buffer.
2. **Zero Data Loss**: Maintain a query frequency for high-rate devices that strictly prevents buffer overwrites, guaranteeing that every generated sample is extracted before a subsequent measurement overwrites it.
3. **Zero Overhead Waste**: Eliminate wasted channel slots, collision states, and redundant queries to maximize effective payload throughput.
4. **Dynamic Adaptation**: Continuously reorganize the query schedule in real time in response to dynamic environmental events and shifting human activity patterns, eliminating reliance on static, offline-designed schedules.

---

2. Reinforcement Learning (RL) Fundamentals

2.1 Definition & The Core Paradigm

**Reinforcement Learning (RL)** is a computational paradigm wherein an agent learns to map situations (states) to actions to maximize a scalar cumulative reward signal through continuous trial-and-error interaction with an environment.

**Formal Definition: Reinforcement Learning** Reinforcement Learning is the formal framework where an active decision-making agent learns an optimal strategy or policy:

$$\pi: \mathcal{S} \to \mathcal{A}$$

mapping environment states $s \in \mathcal{S}$ to actions $a \in \mathcal{A}$, purely through trial-and-error interactions. The objective is to maximize the expected cumulative sum of scalar rewards $R_t \in \mathbb{R}$ over time, without relying on explicit supervisor labels or full environmental models.

Unlike supervised learning—which relies on pre-labeled datasets mapping inputs to ground-truth targets—the RL agent is never explicitly told which action is correct. Instead, the paradigm models biological learning processes: an infant explores motor control through unguided physical interactions, receiving cause-and-effect sensorimotor feedback and gradually discovering how to navigate and manipulate its physical world.

2.2 Online Learning in Dynamic, Non-Static Environments

Offline optimization and open-loop control strategies are ill-suited for autonomous networking because real-world physical channels and user habits are non-static. Environmental dynamics undergo continuous shifts: new human occupants enter a room, residents alter their daily routines, or atmospheric anomalies trigger unexpected sensor bursts.

Under the RL framework, the decision-maker conducts **online learning**:

- The agent does not pre-compute a static schedule offline and execute it blindly.
- The agent continuously updates its internal decision policies _while actively operating_ within the network.
- The agent incorporates its immediate experience stream to adapt dynamically to dynamic environmental shifts rather than relying on stale memory models.

2.3 Foundational Characteristics of Reinforcement Learning

Reinforcement Learning is distinguished by four core operational characteristics:

1. **Trial-and-Error Search**: The agent receives no explicit instructional guidance; it must actively execute diverse actions across states to discover which choices yield optimal scalar rewards.
2. **Delayed Reward**: Actions chosen at time step $t$ influence not only the immediate scalar reward $R_t$, but also shift the environmental state $S_{t+1}$, thereby altering all future reward trajectories.
3. **Short-term vs. Long-term Trade-offs**: The agent must balance immediate feedback against cumulative long-term returns, frequently sacrificing immediate short-term gains to transition into high-value future states.
4. **Sensation, Action, and Goal**: The operational loop binds together **sensation** (perceiving environmental state representations), **action** (executing decisions that alter environmental dynamics), and **goal** (maximizing cumulative long-term scalar returns).

---

3. The General RL Framework & State Representation

3.1 The Agent-Environment Interaction Loop

The interaction loop between the **Agent** (the decision-making algorithm) and the **Environment** (everything external to the agent's direct control) proceeds across discrete time steps $t = 1, 2, 3, \dots$.

```
                    +-------------------+
                    |       Agent       |
                    +-------------------+
                       |             ^
              Action   |             |  Observation O_t
               A_t     |             |  Reward R_t
                       v             |
                    +-------------------+
                    |    Environment    |
                    +-------------------+
```

The step-by-step sequential flow for each slot $t$ proceeds as follows:

1. At time step $t$, the Agent observes the environmental state representation $S_t \in \mathcal{S}$ and selects an action $A_t \in \mathcal{A}$.
2. The Environment receives action $A_t$, updates its internal physical state dynamics, and emits a scalar reward $R_t \in \mathbb{R}$ along with a new sensory observation $O_t \in \mathcal{O}$.
3. The Agent registers observation $O_t$ and reward $R_t$, updates its internal state representation to $S_{t+1}$, and the time index increments to step $t+1$.

3.2 History versus State Representation

The **History** ($H_t$) is the complete chronological sequence of all past actions, observations, and scalar rewards recorded up to time step $t$:

$$H_t = A_1, O_1, R_1, A_2, O_2, R_2, \dots, A_t, O_t, R_t$$

While $H_t$ encompasses all observable information, it cannot be utilized directly for real-time decision-making in autonomous agents: as $t \to \infty$, the length of $H_t$ grows infinitely, demanding unbounded memory storage and computational capacity.

To create a tractable control system, the agent constructs a compact **State** representation ($S_t$), defined as a function of the history:

$$S_t = f(H_t)$$

The function $f$ compresses historical details, preserving only the critical information needed to predict future reward signals and select actions.

3.3 Environment State vs Agent State

Formal RL framework design requires distinguishing between the true physical state of the environment and the internal representation constructed by the agent.

|Dimension|Environment State ($S_t^e$)|Agent State ($S_t^a$)|
|---|---|---|
|**Definition**|The complete internal physical representation of the environment.|The compact internal representation constructed by the agent via $S_t^a = f(H_t)$.|
|**System Perspective**|The absolute physical truth (e.g., exact buffer occupancy of every tag in the home).|The historical log summary used by the algorithm to evaluate choices.|
|**Accessibility**|Unobservable; hidden from the decision-maker.|Fully accessible to the agent's policy and value estimation functions.|

3.4 Partial Observability in Network Environments

**Practical Networking Constraint: Partial Observability** Physical network deployments are almost universally **partially observable** ($S_t^a \neq S_t^e$). The central Reader cannot directly inspect the physical memory registers of battery-free tags without querying them. The Reader observes only discrete feedback signals ($O_t$) resulting from past queries—specifically, whether a query yielded a new data payload, an empty response, or an overwrite notification.

To resolve this gap and avoid maintaining an infinite history log $H_t$, the Reader constructs a compact, bounded Markovian state vector:

$$S_t = (\Delta t_i, \text{Outcome}_i)_{i=1}^n$$

where $\Delta t_i$ represents the elapsed time slots since tag $i$ was last queried, and $\text{Outcome}_i$ stores the feedback status of that query. This bounded tuple eliminates history growth while retaining sufficient information for decision-making.

3.5 State Representation Case Study: The Rat Experiment

David Silver's "Rat Example" illustrates how the state function $f(H_t)$ dictates an agent's predictive conclusions and behavior.

Sequence of Observed Training Episodes:

- **Episode 1**: Light, Light, Lever, Bell $\to$ **Shock**
- **Episode 2**: Bell, Light, Lever, Lever $\to$ **Food**
- **Episode 3**: Lever, Light, Lever, Bell $\to$ **?**

Prediction Analysis Under Three State Functions:

|State Function $f(H_t)$|Evaluated Pattern in Episode 3|Predicted Outcome for Episode 3|
|---|---|---|
|**Function 1: Last 3 Events**|Trailing sequence is `(Light, Lever, Bell)`|**Shock** (Matches Episode 1 suffix)|
|**Function 2: Event Counts**|Total counts are `1 Light, 1 Bell, 2 Levers`|**Food** (Matches Episode 2 sum)|
|**Function 3: Complete Sequence**|Exact sequence is `Lever, Light, Lever, Bell`|**No Prediction Possible** (Sequence never observed)|

This case study demonstrates that the mathematical construction of the state function $f(H_t)$ fundamentally controls how an agent generalizes past experience to predict future outcomes.

---

4. Internal Components of an RL Agent

4.1 Policy ($\pi$): Deterministic vs Stochastic Strategies

A **Policy** ($\pi$) defines the agent's action selection mechanism across states. It represents the core decision logic of an RL agent.

- **Deterministic Policy**: Maps a given state directly to a single action:

$$\pi(s) = a$$

_Smart Home MAC Example_: Always query the tag $i$ that exhibits the highest elapsed time count $\Delta t_i$.

- **Stochastic Policy**: Maps a state to a probability distribution over the action space:

$$\pi(a \mid s) = P[A_t = a \mid S_t = s]$$

_Smart Home MAC Example_: Query the tag with the longest wait time with probability $0.9$, and query any remaining tag with a combined probability of $0.1$.

As the agent receives scalar rewards, it updates its policy parameters, shifting action selection probabilities toward choices that yield higher expected long-term returns.

4.2 Reward Signal ($R_t$) & The Reward Hypothesis

The **Reward Signal** ($R_t \in \mathbb{R}$) is a scalar variable generated at each step, quantifying immediate operational performance.

**The Reward Hypothesis** All operational goals can be described by the maximization of the expected cumulative scalar reward.

Illustrative Reward Model for the Battery-Free Smart Home MAC Protocol:

- **+1 Reward**: Successfully querying a tag that contains a **new data sample**.
- **-1 Reward**: Querying a tag whose buffer is empty (**redundant query / wasted slot**).
- **-5 Reward**: Querying a tag too late, resulting in a sample being **overwritten and lost**.

```
Reward Structure Trade-Offs:
+1  [ Successful Extraction ]  ---> Encourages high throughput
-1  [ Empty Query / Waste ]    ---> Discourages channel slot inefficiency
-5  [ Sample Overwritten ]     ---> Heavy penalty to protect data integrity
```

The asymmetric -5 penalty for buffer overwrites forces the RL agent to prioritize data integrity over throughput optimization, preventing unrecoverable sample loss.

4.3 Return ($G_t$), Discount Factor ($\gamma$), & Value Function ($v_\pi(s)$)

The agent evaluates choices using the **Return** ($G_t$), defined as the discounted sum of future scalar rewards accrued from time step $t$:

$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \dots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$$

where $\gamma \in [0, 1]$ represents the **Discount Factor**:

- As $\gamma \to 0$, the agent becomes _myopic_, focusing on immediate reward $R_{t+1}$.
- As $\gamma \to 1$, the agent becomes _farsighted_, weighting distant future rewards almost equally with immediate feedback.
- _Operational Rationale_: Discounting accounts for environmental uncertainty over distant time horizons. Near-term rewards carry higher predictability, whereas far-future rewards are subject to stochastic environmental variations.

The **State-Value Function** ($v_\pi(s)$) formalizes the expected return achievable from state $s$ under policy $\pi$:

$$v_\pi(s) = \mathbb{E}_\pi [G_t \mid S_t = s]$$

Multi-State Numerical Backward Induction Walkthrough ($\gamma = 0.9$):

Consider a deterministic three-state transition chain ($s_1 \to s_2 \to s_3$):

- State $s_3$ yields an immediate reward $R_3 = +10$, terminating the chain.
- State $s_2$ yields an immediate reward $R_2 = 0$ and transitions to $s_3$.
- State $s_1$ yields an immediate reward $R_1 = 0$ and transitions to $s_2$.

Evaluating state values using backward induction:

$$v_\pi(s_3) = R_3 = +10.0$$

$$v_\pi(s_2) = R_2 + \gamma \, v_\pi(s_3) = 0 + 0.9(10.0) = 9.0$$

$$v_\pi(s_1) = R_1 + \gamma \, v_\pi(s_2) = 0 + 0.9(9.0) = 8.1$$

```
Transition Chain:
[ State s1 ] ----(R1=0)----> [ State s2 ] ----(R2=0)----> [ State s3 ] (R3=+10)
 v_pi(s1)=8.1                 v_pi(s2)=9.0                 v_pi(s3)=10.0
```

This walkthrough proves that state $s_1$ holds a high value ($8.1$) despite generating an immediate reward of zero, because it guarantees access to a high-reward downstream state.

4.4 Model-Free versus Model-Based Reinforcement Learning

RL agents are categorized by whether they construct an explicit internal model of environmental state transitions:

- **Model-Based RL**: The agent constructs an explicit internal transition model $\mathcal{P}(s', r \mid s, a)$ and reward function $\mathcal{R}(s, a)$, enabling internal simulation and tree planning before taking actions.
- **Model-Free RL**: The agent bypasses modeling transition dynamics entirely. It directly estimates state-action value functions or optimizes policy parameters through direct environmental interaction.

_System Design Decision_: For the battery-free smart home MAC protocol, a **Model-Free** approach is adopted. Modeling human physical behavior and ambient sensor triggers across dynamic environments is highly complex; a Model-Free agent adapts directly through observable query feedback without needing a complex environment simulator.

---

5. The Exploration-Exploitation Dilemma

5.1 Core Dilemma & The Challenge of Incomplete Information

**The Exploration-Exploitation Dilemma** Because an agent possesses incomplete information about a dynamic environment, it faces a fundamental operational trade-off:

- **Exploitation**: Selecting actions known to yield high rewards based on past experience. _Risk_: Traps the agent in sub-optimal local decisions and prevents discovery of better alternatives.
- **Exploration**: Selecting unfamiliar actions to gather new environmental information. _Risk_: Incurs immediate penalties or wasted time slots by executing sub-optimal actions.

```
                         EXPLORATION - EXPLOITATION TENSION
                         
        EXPLOITATION                                       EXPLORATION
  Select known high-reward actions.   <=========>   Try unfamiliar actions.
  Maximizes short-term returns.                     Gathers information.
  Risk: Trapped in local optima.                    Risk: Immediate low rewards.
```

5.2 Conceptual Analogy: The Restaurant Dilemma

The trade-off can be illustrated using the decision process of selecting a restaurant:

- _Exploitation Option ("The Usual Place")_: Dining at an established local restaurant where quality is known to be acceptable. The diner guarantees a decent meal but misses the chance to discover superior food elsewhere.
- _Exploration Option ("Grand Opening")_: Dining at a newly opened restaurant around the corner. The diner risks receiving a poor meal, but gains the opportunity to discover a superior dining venue.

```
                           +----------------------+
                           |  Dining Choice Loop  |
                           +----------------------+
                             /                  \
                            /                    \
                           v                      v
              +------------------+          +------------------+
              | "The Usual Place"|          |  "Grand Opening" |
              |  (Exploitation)  |          |   (Exploration)  |
              |  Known Quality   |          | Unknown Quality  |
              +------------------+          +------------------+
```

5.3 Stochastic Environments & Short-Term Trial Sacrifices

In stochastic network environments, individual exploration trials can produce noisy outcomes: an action might fail or succeed due to random variations in physical ambient conditions. Consequently, actions must be sampled repeatedly across time to establish reliable expected reward estimates.

Short-term negative rewards during exploration provide crucial operational intelligence: executing a failing trial provides an **informative failure**, establishing negative feedback that prevents the agent from repeating sub-optimal choices in the future.

5.4 Standard Operational Recipe for Exploration Decay

To resolve this tension over an operational lifetime, autonomous agents implement an **exploration decay schedule**:

|Phase|Operational State|Environmental Knowledge|Strategy & Action Mix|
|---|---|---|---|
|**Phase 1: Initial Exploration**|System Start / Cold Initialization|Zero prior knowledge|High exploration rate ($\epsilon \approx 1.0$). Randomly sample actions to build state estimates.|
|**Phase 2: Transition Phase**|Intermediate Operation|Reward estimates stabilizing|Decay exploration rate ($\epsilon \to 0.1$). Shift toward exploiting high-value actions.|
|**Phase 3: Baseline Exploitation**|Steady-State Operation|Mature value estimates|Low baseline exploration ($\epsilon \approx 0.05$). Maintain continuous background probing.|

Maintaining a non-zero baseline exploration rate ($\epsilon > 0$) during steady-state operation is critical for network adaptability. If environmental dynamics shift—such as a new resident moving into the home or an occupant ending a gaming session to stream video—a non-zero exploration rate allows the Reader to detect changing traffic patterns and adapt its query schedule accordingly.

---

6. Problem Formulation: Smart Home as an RL System

6.1 Formal Mapping Table & System Breakdown

To optimize MAC layer channel scheduling, the battery-free smart home scheduling problem is formally mapped into the Reinforcement Learning framework as follows:

|RL Framework Element|Smart Home System Equivalent|Mathematical Notation / Concrete Definition|
|---|---|---|
|**Agent**|Central RFID Reader|System controller executing real-time channel queries.|
|**Environment**|Smart Home Ecosystem|Heterogeneous device tags, human occupants, and physical ambient conditions.|
|**Action Space (**$\mathcal{A}$**)**|Query Device Selection|$A_t = i$, where $i \in \{1, 2, \dots, n\}$ designates which tag to query in slot $t$.|
|**Observation (**$O_t$**)**|Tag Backscatter Response|$O_t \in \{\text{New Sample Ready}, \text{Empty Buffer / No Data}, \text{Overwrite Flag}\}$.|
|**Reward Function (**$R_t$**)**|Scalar Performance Feedback Signal|$R_t = \begin{cases} +1 & \text{if New Sample Received} \\ -1 & \text{if Empty / Nothing New} \\ -5 & \text{if Sample Overwritten / Lost} \end{cases}$|
|**Agent State (**$S_t$**)**|Bounded History Summary|$S_t = (\Delta t_i, \text{Outcome}_i)_{i=1}^n$: Elapsed slots since last query and last query outcome for each tag $i$.|

6.2 Detailed Component Definition & Execution Dynamics

The step-by-step operational loop for a single communication time slot $t$ proceeds as follows:

```
+-----------------------------------------------------------------------------------+
|                            SINGLE SLOT OPERATIONAL LOOP                           |
|                                                                                   |
| 1. State Inspection  ---> Reader inspects S_t = (\Delta t_i, Outcome_i)_{i=1}^n.   |
| 2. Action Selection  ---> Policy selects action A_t = i (Query Tag i).            |
| 3. Physical Query    ---> Reader emits RF signal; Tag i backscatters response.    |
| 4. Feedback Decode   ---> Reader receives observation O_t from Tag i response.    |
| 5. Reward Assignment ---> Reader evaluates R_t (+1, -1, or -5).                  |
| 6. State & Policy    ---> Timers update (\Delta t_i <- 0; \Delta t_j <- \Delta t_j + 1); |
|    Update                 Policy pi updated using R_t; Slot t increments to t+1. |
+-----------------------------------------------------------------------------------+
```

1. **State Inspection**: At the beginning of slot $t$, the Reader evaluates its current Agent State vector:

$$S_t = (\Delta t_i, \text{Outcome}_i)_{i=1}^n$$

reflecting the elapsed slot count $\Delta t_i$ since each tag $i$ was last queried and the feedback outcome of that prior query.

1. **Action Selection**: The Reader inputs state $S_t$ into its Policy $\pi(a \mid s)$ (e.g., using an $\epsilon$-greedy mechanism) to select action $A_t = i$, designating tag $i$ as the query target for slot $t$.
2. **Physical Execution**: The Reader emits a continuous RF carrier wave modulated with query command $A_t = i$.
3. **Environmental Response & Feedback**: Tag $i$ harvests energy from the incident RF wave, energizes its sensor payload, and backscatters its response. The Reader decodes the payload, yielding observation $O_t$:
    - _Case A_: A new measurement sample is successfully extracted.
    - _Case B_: The buffer is empty (no new data generated since the prior query).
    - _Case C_: The tag indicates an unread measurement was overwritten in its single-sample memory buffer prior to this query.
4. **Reward Computation**: The Reader computes scalar feedback reward $R_t$ based on observation $O_t$:

$$R_t = \begin{cases} +1 & \text{if New Sample Received} \\ -1 & \text{if Empty / Nothing New} \\ -5 & \text{if Data Overwritten / Lost} \end{cases}$$

1. **State & Policy Update**: The Reader updates the target tag's timer ($\Delta t_i \leftarrow 0$), increments all non-queried tag timers ($\Delta t_{j \neq i} \leftarrow \Delta t_j + 1$), updates recorded query outcomes, and constructs state $S_{t+1}$. The scalar reward $R_t$ is fed into the RL algorithm (e.g., Q-learning) to refine policy $\pi$ and value function estimates, continuously optimizing query choices for subsequent time slots.