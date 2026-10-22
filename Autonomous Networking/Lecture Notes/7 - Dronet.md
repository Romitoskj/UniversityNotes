1. Introduction & Foundational Concepts of Autonomous Drone Networks (FANETs)

Context & Strategic Importance

The paradigm of wireless networking is undergoing a fundamental transformation: shifting from static, two-dimensional infrastructures toward highly dynamic, three-dimensional Flying Ad-Hoc Networks (FANETs, also referred to as Dronets or UAVNETs). In traditional networking environments, nodes remain stationary or move within bounded planar geometries, relying on pre-established access points or centralized infrastructure. In contrast, autonomous drone networks deploy unmanned aerial vehicles (UAVs) into unstructured, hostile, or physically inaccessible environments—such as post-disaster zones, agricultural fields, or industrial complexes—where fixed ground infrastructure is either destroyed or entirely absent. In these operational theaters, human remote control of individual nodes becomes unscalable and communication links fluctuate on sub-second timescales. Consequently, node-level autonomous decision-making is mandatory. Each aerial node must independently observe its local environment, evaluate link qualities, execute routing decisions, and adjust its flight trajectory to preserve system-wide data delivery and operational efficiency.

Heterogeneity and Physical Constraints

Drone hardware represents an exceptionally broad technological spectrum. System design parameters, physical dimensions, and energy budgets vary by multiple orders of magnitude across different platform classes.

|Feature / Metric|Spectrum / Characteristic Range|
|---|---|
|**Weight**|$0.5\text{ g}$ (e.g., RoboBee) to $15,000\text{ kg}$ (e.g., Global Hawk)|
|**Operational Speed**|Commercial class: $50\text{--}150\text{ km/h}$|
|**Energy Sources**|Hydrocarbon Fuel, Lithium-Polymer (Li-Po) Batteries, Solar Cells|
|**Flight Autonomy**|Commercial battery-powered UAVs: $\approx 30\text{ minutes}$|
|**Airframe Geometry**|Single-rotor, Multi-rotor, Fixed-wing, Airship|

A fundamental physical reality governs all autonomous drone networking: **energy is the single scarcest resource**. However, the thermodynamic distribution of energy consumption in a FANET differs radically from conventional Wireless Sensor Networks (WSNs).

$$\text{Energy Expenditure Ratio}: P_{\text{flight}} \gg P_{\text{radio}}$$

For a small multi-rotor aircraft, mechanical propulsion and aerodynamic lift consume power on the order of **hundreds of Watts** ($\approx 10^2\text{ W}$ to $10^3\text{ W}$). Conversely, onboard Wi-Fi radio transmission consumes approximately **1 Watt** ($\approx 1\text{ W}$). This disparity establishes a critical strategic constraint: **physically moving an aerial node costs orders of magnitude more energy than transmitting data across the wireless medium**. Every flight maneuver or trajectory adjustment drastically reduces battery autonomy, dictating that networking protocols must strictly minimize redundant flight distance and hovering time.

Comparative Architectural Analysis (WSN vs. Dronet)

To design effective protocols for aerial nodes, architectural assumptions inherited from terrestrial sensor networks must be re-evaluated across fundamental operational axes.

|Axis of Comparison|Wireless Sensor Networks (WSNs)|Flying Ad-Hoc Networks (Dronets)|
|---|---|---|
|**Mobility**|None (static) or highly partial|High, dynamic, full 3D volumetric movement|
|**Topology**|Random deployment, structurally stable|Fluid mesh topology changing within seconds|
|**Infrastructure**|Absent; relies on fixed Base Stations / Sinks|Absent; relies on mobile Depots ($d_1, d_2$)|
|**Energy Constraints**|Battery-constrained (dominated by radio activity)|Battery-constrained (dominated by propulsion/flight)|
|**Primary Use Cases**|Static environmental & structural monitoring|Search & rescue, surveillance, emergency payload delivery|

Within this operational domain, FANET systems must address four core domain problems:

1. **Emergency Communication:** Deploying UAVs as rapid, temporary network infrastructure to restore civilian connectivity without specialized user software.
2. **Mission Planning & Trajectory Selection:** Allocating multi-trip flight trajectories across heterogeneous squads to maximize mission coverage while accounting for periodic depot visits for battery recharging and data offloading.
3. **Continuous Multi-Hop Data Delivery:** Transporting high-bandwidth sensor streams (e.g., video, thermal imaging) across fluid multi-hop paths while nodes move continuously.
4. **Periodic Data Offloading:** Managing network connectivity over vast geographical areas where available nodes are too sparse to maintain continuous multi-hop paths to the depot.

These physical, mechanical, and energetic constraints directly dictate the design of specialized dual-radio hardware stacks and zero-configuration emergency networking architectures.

---

2. Emergency Communications & System Integration: The DANGER Architecture

Context & Strategic Importance

When natural disasters or industrial accidents collapse conventional cellular towers and landline infrastructure, disaster response hinges on rapidly establishing emergency communications. In these environments, survivors and first responders cannot be expected to possess specialized hardware or pre-installed software applications. An ideal emergency networking deployment requires zero-configuration, application-free access, enabling standard civilian smartphones to instantly connect to an ad-hoc aerial relay network.

The DANGER System Architecture

To solve Emergency Communication (Problem 1), the **DANGER** system architecture (Coletta et al., ACM MobiHoc 2020) deploys autonomous drones as airborne Access Points (APs) and multi-hop relays.

```
+-------------------------------------------------------------------+
|                           ON-BOARD DRONE                          |
|  +-------------------------------------------------------------+  |
|  |                Raspberry Pi Processing Unit                 |  |
|  |  +-------------------------------------------------------+  |  |
|  |  |                    Core Engine                        |  |  |
|  |  | [Drone Controller] [Mesh Manager] [DNS] [WebRTC] [Catch]|  |  |
|  |  +-------------------------------------------------------+  |  |
|  |                             |                               |  |
|  |  +-------------------------------------------------------+  |  |
|  |  |                    Radio Module                       |  |  |
|  |  |   [Wi-Fi AP]       [Wi-Fi Mesh]      [Wi-Fi Drone]    |  |  |
|  +--+-------+-----------------+-----------------------------+--+  |
+-------------|-----------------|-----------------------------------+
              |                 |
     Civilian Smartphones   Inter-Drone Mesh Connection
     (Captive Web Portal)    (Data & Signal Relaying)
```

The hardware and software stack onboard each drone integrates the following components:

- **On-Board Processing Unit:** A local Raspberry Pi acts as the central compute node, executing the system's **Core Engine**. This suite includes the _Drone Controller_, _Mesh Manager_, custom _DNS server_, _WebRTC_ signaling service, and a _Page Catch_ (Captive Portal) mechanism.
- **Dual Wi-Fi Interface Radio Stack:**
    1. **Wi-Fi Access Point (AP) Interface:** Emits an open IEEE 802.11 network broadcasting an unencrypted SSID.
    2. **Wi-Fi Mesh Interface:** Operates on a dedicated wireless channel reserved exclusively for high-speed inter-drone communication, allowing multiple flying APs to relay data across multi-hop links back to a ground station.
- **Captive Portal & WebRTC Signaling Operations:** When a survivor connects using an unmodified commercial smartphone, the local DNS server hijacks Domain Name System requests (intercepting Port 53 traffic). Raw HTTP GET requests issued by iOS or Android operating system captive-portal detectors are intercepted by the _Page Catch_ module. This automatically pushes a zero-configuration mobile web UI to the device without requiring cellular backhaul, active internet access, or app installation. Peer-to-peer real-time voice and text sessions between survivors and responders are established using the onboard _WebRTC_ signaling server.
- **User & Ground Station Interface:** Survivors access a web page featuring emergency broadcast alert messages, direct voice/text links to the _Emergency Coordinator_ and _Health Service_, and active peer contact listings. At the ground command center, an operator laptop ingests relayed packet streams, mapping civilian distress messages and precise GPS coordinates onto a spatial GIS interface.

Mission Planning & Trajectory Selection

Deploying systems like DANGER over expansive regions introduces Problem 2: **Mission Planning & Trajectory Selection**.

```
                          [Drone Tour Database (Cu)]
                                    /     \
                                   /       \
                                  v         v
   (Depot d1) <====== [Drone Squad 1]     [Drone Squad 2] ======> (Depot d2)
  Recharge/Offload      Heterogeneous      Heterogeneous     Recharge/Offload
                         Trajectories       Trajectories
```

Instead of optimizing individual single-flight paths, multi-UAV deployments require joint trajectory allocation over heterogeneous fleets that differ in battery capacity, sensor payloads, maximum speeds, and energy discharge rates. Input parameters consist of defined target observation locations, strict required target hovering durations, and multiple spatially separated depots ($d_1, d_2$). The mission planning framework schedules multi-trip flight paths, calculating intermediate returns to depots for battery replacement/recharging and data offloading to sustain continuous long-term operational coverage.

Sustaining these high-level architectural deployments requires an understanding of the underlying low-level radio link dynamics and MAC layer protocols necessary to sustain them.

---

3. Link Dynamics and Dual-Channel MAC Layer Protocols

Context & Strategic Importance

In a three-dimensional FANET, rapid physical movement transforms wireless channel dynamics. High relative velocities, spatial displacement, and unobstructed line-of-sight (LOS) propagation cause rapid fluctuations in signal-to-noise ratios (SNR). Static Medium Access Control (MAC) protocols designed for fixed terrestrial networks fail in these environments because their parameters cannot adapt quickly enough to sub-second topology changes.

Dual-Channel Heterogeneous Radio Architecture

To balance high-bandwidth data transmission with robust long-range signaling, advanced FANET platforms utilize a dual-channel radio configuration.

|Architectural Axis|Primary Data Channel|Secondary Control Channel|Terrestrial Alternative (Cellular)|
|---|---|---|---|
|**Protocol Standard**|Wi-Fi (IEEE 802.11)|LoRa (LPWAN)|LTE / 5G Mobile Networks|
|**Operational Range**|Hundreds of meters ($140\text{ m}$ in standard simulations)|Kilometers ($6\text{ km}$ in standard simulations)|Wide-area coverage via ground base stations|
|**Data Throughput**|Up to hundreds of Mbit/s|$0.3\text{ -- }50\text{ kbit/s}$|High throughput|
|**MAC Layer Scheme**|CSMA/CA|ALOHA-like (Duty Cycled)|Base-station scheduled|
|**Primary Function**|High-bandwidth data payload transmission (video streams, thermal imaging)|Essential control traffic, flight coordinates, topology commands|Auxiliary data/control (frequently unavailable in disaster zones)|

Kinematic Link Lifetime Mathematics

Because Wi-Fi data range is bounded (e.g., $r = 140\text{ m}$), rapid node kinematics curtail the duration of active wireless links. Consider two UAVs, Node $A$ and Node $B$, moving directly toward each other in a head-on trajectory at constant speeds $v_1 = v_2 = 8\text{ m/s}$. The relative closing velocity is:

$$v_{\text{relative}} = v_1 + v_2 = 8\text{ m/s} + 8\text{ m/s} = 16\text{ m/s}$$

```
t = 0.00s : [ A ] ---------------------- 280m ---------------------- [ B ]  (Link Starts: Distance = 140m + 140m)
                  ==============>                       <==============
                               (v_rel = 16 m/s)

t = 8.75s :                               [A/B]                             (Nodes Cross: Distance = 0m)

t = 17.5s : [ B ] ---------------------- 280m ---------------------- [ A ]  (Link Breaks: Distance = 140m + 140m)
```

The mathematical derivation of link lifetime $T$ proceeds across two distinct phases:

1. **Approach Phase:** Node $B$ enters Node $A$'s communication boundary at maximum range $r = 140\text{ m}$ ($t = 0\text{ s}$). The time elapsed until the nodes intersect ($d_{\text{relative}} = 0\text{ m}$) is: $$t_{\text{approach}} = \frac{r}{v_{\text{relative}}} = \frac{140\text{ m}}{16\text{ m/s}} = 8.75\text{ seconds}$$
2. **Separation Phase:** Following intersection, the nodes diverge until Node $B$ exits Node $A$'s radio range at $r = 140\text{ m}$: $$t_{\text{separation}} = \frac{r}{v_{\text{relative}}} = \frac{140\text{ m}}{16\text{ m/s}} = 8.75\text{ seconds}$$
3. **Maximum Head-on Link Lifetime:** Total continuous contact duration $T$ for a direct head-on path equals: $$T = t_{\text{approach}} + t_{\text{separation}} = 8.75\text{ s} + 8.75\text{ s} = 17.5\text{ seconds}$$

_Pythagorean Chord Geometry Derivation for Lateral Offsets:_ In non-coaxial trajectories where nodes pass each other with a lateral offset distance $d$ (where $d < r$), the flight trajectory forms a straight line cutting through the circular coverage area of radius $r$ centered at the opposing node.

```
                     +-----------------------+
                     |    RADIO RANGE (r)    |
                     |       /|              |
                     |      / |              |
                     |  r  /  | d            |
                     |    /   |              |
                     |   /____|              |
                     |  x = √(r² - d²)       |
                     +-----------------------+
                     |<--- Trajectory --->|
                       Length = 2√(r² - d²)
```

By the Pythagorean theorem, the distance from the midpoint of closest approach to the entry or exit boundary is $x = \sqrt{r^2 - d^2}$. Thus, the total chord length traversed inside the radio coverage envelope is $2x = 2\sqrt{r^2 - d^2}$. Dividing this physical chord length by the relative speed $v_{\text{relative}}$ yields the generalized link lifetime formula:

$$T = \frac{2\sqrt{r^2 - d^2}}{v_{\text{relative}}}$$

_Worked Mathematical Example:_ Calculate the link duration $T$ for two UAVs flying at relative speed $v_{\text{relative}} = 16\text{ m/s}$, with radio range $r = 140\text{ m}$ and lateral offset distance $d = 100\text{ m}$.

\begin{aligned} T &= \frac{2\sqrt{140^2 - 100^2}}{16} = \frac{2\sqrt{19600 - 10000}}{16} \ &= \frac{2\sqrt{9600}}{16} = \frac{2 \times 97.9796}{16} = \frac{195.959}{16} \approx 12.25\text{ seconds} \end{aligned}

With a $100\text{ m}$ lateral offset, the available communication window drops from $17.5\text{ s}$ to approximately $12.25\text{ s}$.

MAC Layer Challenges in 3D Space

These short operational windows induce several operational challenges at the MAC layer:

- **Rapid Neighbor Table Aging:** Link entries stored in MAC routing tables become obsolete within seconds. Dynamic neighbor discovery protocol overhead increases significantly to keep pace with topology changes.
- **Heightened Line-of-Sight (LOS) Contention:** Operating high above ground terrain removes physical obstructions. While this extends line-of-sight range, it expands the carrier-sensing zone, increasing the number of nodes competing for channel access under CSMA/CA.
- **Dynamic Hidden Terminal Conditions:** Node displacement dynamically alters node visibility patterns, causing spatial contention domains and hidden terminal collisions to shift continuously across 3D space.
- **Adaptive Parameterization:** Traditional fixed contention window sizes and static retry limits degrade throughput. MAC protocols must dynamically adapt operational parameters using online learning mechanisms.

Because short-lived link durations constrain physical contact windows, routing protocols must rely on real-time link quality estimation or location awareness to route data effectively.

---

4. Multi-Hop Routing Protocols: Quality-Driven vs. Geographic Paradigms

Context & Strategic Importance

Multi-hop data delivery (Problem 3) presents a fundamental engineering dilemma: **balancing route freshness against control message overhead** in rapidly changing topologies. To maintain valid end-to-end paths across fast-moving UAVs, protocols exchange control packets. However, over-allocating radio bandwidth to control packets starves data payloads and accelerates battery drain.

```
                  +-----------------------------------+
                  |      FANET ROUTING DILEMMA        |
                  +-----------------------------------+
                  | High Route Freshness              |
                  | Low End-to-End Latency            |
                  | High Packet Delivery Ratio (PDR)  |
                  +-----------------------------------+
                                    |
                    COMPETING PERFORMANCE GOALS
                                    v
                  +-----------------------------------+
                  | Low Control Packet Overhead       |
                  | High Scalability                  |
                  | Minimal Energy Consumption        |
                  +-----------------------------------+
```

Taxonomy of Routing Protocol Families

To address this dilemma, four main families of mobile ad-hoc routing protocols have been adapted for drone networks:

|Protocol Family|Representative Examples|Operational Mechanism|Trade-offs & Failure Modes in FANETs|
|---|---|---|---|
|**Proactive**|OLSR, DSDV, B.A.T.M.A.N.|Continuously updates routing tables to all destinations via periodic background messages.|High control overhead; slow to converge when topology changes in seconds, leading to stale routes.|
|**Reactive**|AODV, DSR|Initiates route discovery on-demand only when a data packet needs to be sent.|High route discovery latency; active paths frequently break mid-transmission as drones fly apart.|
|**Hybrid**|ZRP, TORA|Combines proactive routing within local zones with reactive routing for distant targets.|Scales better structurally, but introduces implementation complexity and high memory overhead.|
|**Geographic**|Greedy Forwarding, GDSTR-3D|Selects next-hop relays based strictly on physical GPS coordinates of local neighbors.|Eliminates routing tables and discovery latency; susceptible to planar graph dead ends (local optima).|

Deep-Dive: B.A.T.M.A.N. (Better Approach To Mobile Ad-hoc Networking)

B.A.T.M.A.N. is a proactive, decentralized, distance-vector routing protocol driven by statistical link-quality measurement. It does not calculate complete global network topology. Instead, each node determines only the optimal **next-hop** neighbor for each known destination.

_Operational Mechanism & Originator Messages (OGMs):_ Every node periodically broadcasts a small control packet called an Originator Message (OGM). Neighboring nodes receive and rebroadcast these OGMs across the mesh network.

```
+---------------------------------------------------------------------------------+
|                                 OGM PACKET STRUCTURE                            |
+---------------------+-----------------+----------+----------------+-------------+
| Originator Address  | Sequence Number |   TTL    | Link Quality   |  Hop Count  |
| (Source Node ID)    | (Loop Control)  | (Limits) | Metric (TQ)    | (Transits)  |
+---------------------+-----------------+----------+----------------+-------------+
```

_The Asymmetric Link Flaw:_ Early protocol revisions (such as B.A.T.M.A.N. v3) evaluated link quality solely by counting the total number of OGMs received from a given neighbor (Receive Quality, $RQ$). This introduced a critical flaw over asymmetric wireless links:

```
  Node A  ================= 100% OGM Reception ================>  Node B
  (Sends Data)                                                   (Receives)
  
  Node A  <----------------- 5% OGM Reception ------------------  Node B
```

If Node $A$ receives $100\%$ of OGMs sent by Node $B$, but Node $B$ receives only $5\%$ of OGMs sent by Node $A$, Node $A$ would incorrectly conclude that the link to $B$ is perfect ($100\%$). If Node $A$ forwards high-bandwidth data payloads along this asymmetric path, $95\%$ of the data packets will be dropped in transit ($A \to B$).

_Mathematical Resolution via Sliding Windows:_ Modern B.A.T.M.A.N. resolves link asymmetry by tracking both directional paths using a sliding window buffer of size $W$ (typically $W = 64$ packets).

1. **Receive Quality (**$RQ$**):** Total OGMs generated by neighbor $B$ and received by node $A$ within window $W$: $$RQ = \text{Count}(\text{OGM}_{B \to A})$$
2. **Echo Quality (**$EQ$**):** Total OGMs originally generated by node $A$ that neighbor $B$ successfully received and echoed back to $A$ within window $W$: $$EQ = \text{Count}(\text{OGM}_{A \to B \to A})$$
3. **Local Transmit Quality (**$TQ_{\text{local}}$**):** The directional probability of successfully transmitting a packet from $A$ to $B$. Demonstrating the explicit ratio algebra: $$\frac{EQ}{W} \approx P(A \to B) \times P(B \to A), \quad \frac{RQ}{W} \approx P(B \to A)$$ $$TQ_{\text{local}} = \frac{EQ / W}{RQ / W} = \frac{EQ}{RQ} \approx \frac{P(A \to B) \times P(B \to A)}{P(B \to A)} = P(A \to B)$$ Dividing Echo Quality by Receive Quality cancels out the reverse probability $P(B \to A)$, isolating the forward transmission success rate $P(A \to B)$.
4. **Path** $TQ$ **Propagation Rule:** As an OGM traverses multi-hop paths, its metric is updated hop-by-hop. When node $C$ receives an OGM from Originator $O$ via intermediate neighbor $B$, path quality is calculated as: $$TQ_{\text{path}} = TQ_{\text{received}} \times TQ_{\text{local}}$$ Node $C$ selects the neighbor that yields the highest cumulative $TQ_{\text{path}}$ as its primary next-hop destination.

Deep-Dive: Geographic Routing & Void Recovery

Geographic routing eliminates routing tables by leveraging physical coordinates. Each drone acquires its position via onboard GPS and discovers neighbor coordinates through periodic local broadcast exchange messages (_Hello_ packets containing $[ID, x, y]$).

_Greedy Forwarding Algorithm:_ A node holding a packet computes the Euclidean distance from all immediate neighbors to destination $D$. It forwards the packet to the neighbor yielding maximum single-hop progress toward $D$.

_The Dead End (Local Optimum) Problem:_ Greedy forwarding fails when a packet encounters a node $F$ whose distance to destination $D$ is less than that of all its immediate neighbors, yet $F$ is out of radio range of $D$. Node $F$ enters a **Dead End** (local optimum), halting greedy progress.

```
                           (Dead End Node)
                             [ Node F ]  <--- d(F, D) = 171m (Local Optimum)
                            /          \
                           /            \
                d=242m    /              \   d=265m
            [ Node C ] --+                +-- [ Node E ]
                 |                                |
                 v                                v
            [ Node A ] (d=330m)              [ Node H ] (d=180m)
                 ^                                |
                 |                                v
            [ Source S ]                     [ Node I ] (d=87m)
            (d=422m)                              |
                                                  v
                                            [ Destination D ]
```

To break out of local optima, protocols utilize three recovery mechanisms:

1. **Backtracking:** Dead-end node $F$ returns the packet to previous sender $C$, which marks $F$ as explored and attempts forwarding to an alternative neighbor ($E$). _Cost & Trade-offs:_ Increases hop count (e.g., 9 hops instead of 5), generates extra transmissions, and requires packet headers to store state history of visited nodes.
2. **Store-Carry-Forward:** Node $F$ retains the packet in memory and physically flies toward destination $D$ at velocity $v$ (e.g., $8\text{ m/s}$) until $D$ or a closer forwarding relay enters radio range. _Cost & Trade-offs:_ Incurs physical transport delay (e.g., carrying a packet over $71\text{ m}$ at $8\text{ m/s}$ requires $\approx 9\text{ seconds}$), consumes flight energy, and temporarily diverts the drone from its primary mission.
3. **Face Routing (GPSR - Greedy Perimeter Stateless Routing):** Switches from greedy mode to perimeter mode, traversing the perimeter of the network graph using the _Right-Hand Rule_ (navigating counter-clockwise around the spatial void) until reaching a node closer to $D$ than $F$, where greedy forwarding resumes. _Planarization Requirement:_ Face routing requires graph planarization using algorithms such as the Relative Neighborhood Graph (RNG) or Gabriel Graph (GG). Planarization removes crossing edges in 2D spatial graphs, ensuring that applying the Right-Hand Rule does not enter infinite loops around intersecting link edges.

Transitioning from continuous data routing during flight to managed mobility techniques becomes necessary when continuous links cannot be sustained over vast mission areas.

---

5. Managed Mobility & Periodic Data Offloading: The Stop & Route Framework

Context & Strategic Importance

In expansive operational areas with small drone fleets, nodes cannot maintain a continuously connected multi-hop mesh network. Adapting classical network protocols to chaotic node movement under these conditions leads to network partitions and packet loss.

The **Stop & Route** framework introduces a paradigm shift: **instead of adapting protocols to unpredictable mobility, controlled physical mobility is harnessed to establish periodic, guaranteed multi-hop network topologies**.

The Stop & Route Operational Cycle

Stop & Route establishes an interleaved operational lifecycle:

```
+-------------------------------------------------------------------+
|                  STOP & ROUTE OPERATIONAL LOOP                    |
+-------------------------------------------------------------------+
| 1. SENSE     : Drones spread out independently to perform mission |
|                sensing over target areas.                         |
| 2. STOP      : Drones halt sensing at synchronized time intervals.|
| 3. CONNECT   : Drones fly to form a connected physical chain.    |
| 4. OFFLOAD   : Data offloads to depot d1 via multi-hop Wi-Fi.     |
| 5. RESUME    : Drones return to sensing locations. Loop repeats.  |
+-------------------------------------------------------------------+
```

_Optimization Objective:_ Topology selection is formulated to **maximize the worst residual battery energy** among all participating UAVs at the end of the rendezvous cycle:

$$\text{Maximize } \min_{i \in \text{Drones}} \left( E_{\text{residual}, i} \right)$$ $$E_{\text{residual}, i} = E_{\text{current}, i} - E_{\text{flight}, i} - E_{\text{hover}, i}$$

Where $E_{\text{flight}, i}$ represents the energy expended flying to the rendezvous point and $E_{\text{hover}, i}$ is the energy burned hovering while waiting for offloading to complete.

Centralized Topology Formation (GAA Heuristic)

Finding the globally optimal rendezvous topology via a Mixed-Integer Linear Program (MILP) is computationally expensive. Solving the MILP requires **18.6 seconds** for 8 drones and **64 seconds** for 25 drones. Because drones burn significant battery power while hovering in place waiting for instructions, long computation delays severely degrade overall operational lifetime.

To solve this, the centralized **Greedy Assignment Algorithm (GAA)** computes near-optimal topologies in polynomial time ($O(m^5)$):

1. **Spatial Discretization:** The mission space is mapped into a grid of hexagonal tiles, placing candidate rendezvous points at tile centers.
2. **Region Expansion:** A connected region expands outward starting from depot $d_1$.
3. **Greedy Selection:** Candidate tiles adjacent to the connected region are evaluated. GAA iteratively assigns the drone-tile pair that minimizes the reduction in worst residual battery energy across the fleet until all nodes are connected.

Distributed Topology Formation (DGA Protocol)

When long-range control channels (LoRa) are unavailable, drones construct a multi-hop topology using local, short-range Wi-Fi signaling alone via the **Distributed Greedy Algorithm (DGA)**.

DGA enforces **Three Local Deterministic Rules**:

```
[Is Drone in radio range of Depot or an already connected Drone?]
                       /                 \
                     YES                  NO
                     /                     \
             [ Rule 1: STOP ]     [Is Drone in range of a non-connected]
                                  [ drone closer to the Depot (Leader)? ]
                                           /                 \
                                         YES                  NO
                                         /                     \
                             [ Rule 2: FOLLOW Leader ]   [ Rule 3: MOVE to Depot ]
```

_Distributed Life Cycle:_

1. **Spread Out:** Drones execute decentralized sensing tasks across the mission area.
2. **Move Closer:** Upon triggering offloading mode, isolated drones execute Rule 3, moving directly toward depot $d_1$.
3. **Follow Leader:** Nodes detecting non-connected peers closer to the depot execute Rule 2, aligning behind them.
4. **First Connections:** The leading drone reaches depot radio range and executes Rule 1 (STOP).
5. **Region Grows:** Following drones enter range of newly stopped nodes and trigger Rule 1, expanding the connected chain outward.
6. **All Connected:** The mesh chain completes, and high-speed Wi-Fi offloading begins.

_Empirical Performance Gains:_

```
  Building Time (Seconds)
  130 |---------------------------------------------
      |   * CPVF
  110 |-----\---------------------------------------
      |      \---* Floor-CPVF
   90 |-----------\---------------------------------
      |            \---* DGA-grid (~100s -> ~70s)
   70 |---------------------------------------------
      +---------------------------------------------
      5           10          15          20          25
                              Number of UAVs
```

Simulation data confirms DGA performance improvements over competing force-based methods (CPVF):

- **Building Time:** For 5 drones, DGA constructs a connected topology in $\approx 100\text{ s}$ (compared to $> 120\text{ s}$ for CPVF). For 25 drones, DGA formation time drops to $\approx 70\text{ s}$ (compared to $85\text{--}90\text{ s}$ for CPVF).
- **Mission Extension:** Reduced topology formation time translates directly to mission performance, yielding an approximate **+7% increase in active sensing time**.

Bridging deterministic, hand-crafted protocol rules to modern machine learning abstractions allows drones to learn optimal policies autonomously.

---

6. Formal Decision-Making Frameworks & Exam Practice Solutions

Context & Strategic Importance

Deterministic protocol heuristics—such as Greedy Geographic Forwarding or DGA local rules—are hand-crafted algorithms designed for specific operational scenarios. To enable true autonomy, these heuristics can be formalized as structural policies within a **Markov Decision Process (MDP)** framework, laying the foundation for Reinforcement Learning (Q-learning).

MDP Formulation for Autonomous Drones

A discrete-time MDP model for an autonomous networking drone is defined by four core elements:

$$\text{MDP Tuple}: \langle S, A, R, P \rangle$$

- **State Space (**$S$**):** The operational condition of the drone at time $t$, defined by: $$s_t = \left[ \mathbf{p}_i(t), \mathcal{N}_i(t), Q_i(t), E_{\text{battery}, i}(t) \right]$$ Where $\mathbf{p}_i(t) = (x, y, z)$ is 3D position, $\mathcal{N}_i(t)$ is the active neighbor list, $Q_i(t)$ is packet queue depth, and $E_{\text{battery}, i}(t)$ is remaining battery energy.
- **Action Space (**$A$**):** The set of networking and kinematic decisions available to the node: $$a_t \in \{ \text{Forward to neighbor } j, \text{ Store-and-Carry payload}, \text{ Move toward depot}, \text{ Continue sensing} \}$$
- **Reward Function (**$R$**):** Scalar feedback derived from multi-criteria performance metrics: $$R(s_t, a_t) = w_1 \cdot (\text{Packet Delivery}) - w_2 \cdot (\text{End-to-End Latency}) - w_3 \cdot (\text{Energy Consumed})$$
- **Policy (**$\pi$**):** A mapping from observed states to action selection probabilities $\pi(a|s)$. Hand-written heuristics (e.g., DGA rules) represent fixed, static policies. Machine learning techniques allow nodes to discover optimal policies $\pi^*$ autonomously.

_Multi-Armed Bandits vs. MDPs:_ Multi-Armed Bandit abstractions are insufficient for FANET protocol control because bandit models assume actions do not alter future system states. In drone networks, selecting a kinematic action (e.g., moving toward a depot) directly alters spatial state $\mathbf{p}_i(t+1)$ and yields delayed rewards across multi-hop routes.

---

Analytical Exam Practice Problems & Step-by-Step Solutions

Exercise 1: Geographic Routing & Local Optimum Dead End Recovery

```
                             [ Node A ] (267m)
                            /         \
                 d=320m    /           \   d=232m
            [ Source S ] --             -- [ Node B ]
                                               |
                                               v  d=152m
                                          [ Node C ]
                                         /          \
                              d=89m     /            \   d=86m
                         [ Node F ] ----              ---- [ Node G ]
                                \                    /
                                 \                  /
                                  v                v
                                  [ Destination D ]
```

**Scenario Setup:** Source $S$ holds a packet addressed to Destination $D$. Communication range $r = 100\text{ m}$. Euclidean distances to $D$ are: $d(S,D)=320\text{m}$, $d(A,D)=267\text{m}$, $d(B,D)=232\text{m}$, $d(C,D)=152\text{m}$, $d(E,D)=179\text{m}$, $d(F,D)=89\text{m}$, $d(G,D)=86\text{m}$. Node speeds are $v = 8\text{ m/s}$.

Rule: Forward to the neighbor closest to $D$, provided that neighbor is closer to $D$ than the current node.

**(a) Compute the Greedy Path from** $S$ **to** $D$**. Justify each hop.**

_Step-by-Step Solution:_

1. **At Source** $S$ **(**$320\text{ m}$**):** Neighbors within range ($100\text{ m}$) are $A$ ($267\text{ m}$) and $B$ ($232\text{ m}$). Evaluate distances: $232\text{ m} < 267\text{ m} < 320\text{ m}$. Select **Node** $B$.
2. **At Node** $B$ **(**$232\text{ m}$**):** Neighbors within range are $A$ ($267\text{ m}$), $C$ ($152\text{ m}$), and $E$ ($179\text{ m}$). Evaluate distances: $152\text{ m} < 179\text{ m} < 267\text{ m}$. Select **Node** $C$.
3. **At Node** $C$ **(**$152\text{ m}$**):** Neighbors within range are $F$ ($89\text{ m}$) and $G$ ($86\text{ m}$). Evaluate distances: $86\text{ m} < 89\text{ m}$. Select **Node** $G$.
4. **At Node** $G$ **(**$86\text{ m}$**):** Destination $D$ is within radio range ($86\text{ m} \le 100\text{ m}$). Forward directly to **Destination** $D$.

$$\text{Final Greedy Path (a)}: S \longrightarrow B \longrightarrow C \longrightarrow G \longrightarrow D$$

**(b) If Node** $G$ **experiences hardware failure, calculate the updated path.**

_Step-by-Step Solution:_

- Upon reaching Node $C$ ($152\text{ m}$), Node $G$ is unreachable.
- Node $C$ evaluates remaining neighbor $F$ ($89\text{ m}$). Since $89\text{ m} < 152\text{ m}$, forwarding to $F$ is valid.
- At Node $F$ ($89\text{ m}$), Destination $D$ is within radio range ($89\text{ m} \le 100\text{ m}$). Forward directly to $D$.

$$\text{Final Path (b)}: S \longrightarrow B \longrightarrow C \longrightarrow F \longrightarrow D$$

**(c) If both Node** $G$ **and Node** $F$ **fail, analyze the protocol state. If Store-Carry-Forward is used at flight speed** $v = 8\text{ m/s}$**, calculate the delay until delivery.**

_Step-by-Step Solution:_

1. **Protocol Analysis:** At Node $C$ ($152\text{ m}$), remaining neighbors ($A$ at $267\text{ m}$, $B$ at $232\text{ m}$) are farther from $D$ than Node $C$ ($152\text{ m}$). Node $C$ enters a **Dead End (Local Optimum)**; greedy forwarding fails.
2. **Store-Carry-Forward Latency Calculation:**
    - Current distance from $C$ to $D$: $d_{\text{current}} = 152\text{ m}$.
    - Wi-Fi radio reception range: $r = 100\text{ m}$.
    - Physical flight distance required before $D$ enters radio range: $$d_{\text{flight}} = d_{\text{current}} - r = 152\text{ m} - 100\text{ m} = 52\text{ meters}$$
    - Time delay incurred at flight speed $v = 8\text{ m/s}$: $$t_{\text{delay}} = \frac{d_{\text{flight}}}{v} = \frac{52\text{ m}}{8\text{ m/s}} = 6.5\text{ seconds}$$

---

Exercise 2: B.A.T.M.A.N. Asymmetric Link Metrics

**Scenario Setup:** Node $A$ needs to select a next-hop route toward Originator $O$. Intermediate neighbors $B$ and $C$ forward OGMs from $O$. Node $A$ collects statistics over a sliding window of $W = 64$ packets:

|Metric Parameter|Neighbor B|Neighbor C|
|---|---|---|
|OGMs of Neighbor received by Node A ($RQ$)|60|64|
|Own OGMs of Node A echoed by Neighbor ($EQ$)|54|40|
|Carried Path $TQ$ towards Originator $O$ ($TQ_{\text{incoming}}$)|0.80|0.95|

**(a) Extract packet counts for Receive Quality (**$RQ$**) and Echo Quality (**$EQ$**), and calculate local Transmit Quality (**$TQ_{\text{local}}$**) for links** $A \to B$ **and** $A \to C$**.**

_Step-by-Step Solution:_

- **Neighbor** $B$**:** $$RQ_B = 60 \text{ packets}, \quad EQ_B = 54 \text{ packets}$$ $$TQ_{\text{local}, B} = \frac{EQ_B}{RQ_B} = \frac{54}{60} = 0.90 \quad (90\%)$$
- **Neighbor** $C$**:** $$RQ_C = 64 \text{ packets}, \quad EQ_C = 40 \text{ packets}$$ $$TQ_{\text{local}, C} = \frac{EQ_C}{RQ_C} = \frac{40}{64} = 0.625 \quad (62.5\%)$$

**(b) Calculate cumulative path quality (**$TQ_{\text{path}}$**) toward Originator** $O$ **through both neighbors. Determine which next-hop Node** $A$ **selects.**

_Step-by-Step Solution:_

- **Path quality via Neighbor** $B$**:** $$TQ_{\text{path}, B} = TQ_{\text{incoming}, B} \times TQ_{\text{local}, B} = 0.80 \times 0.90 = 0.72 \quad (72\%)$$
- **Path quality via Neighbor** $C$**:** $$TQ_{\text{path}, C} = TQ_{\text{incoming}, C} \times TQ_{\text{local}, C} = 0.95 \times 0.625 = 0.59375 \quad (59.375\%)$$

_Decision:_ Node $A$ selects **Neighbor** $B$ as its next hop, because $0.72 > 0.59375$.

**(c) Identify which neighbor Node** $A$ **would select if it evaluated links using only received OGMs (**$RQ$**). Explain why that choice is flawed.**

_Step-by-Step Solution:_

- _Receive-Only Selection:_ Node $A$ would measure $RQ_C = 64/64 = 100\%$ versus $RQ_B = 60/64 = 93.75\%$, incorrectly selecting **Neighbor** $C$.
- _Flaw:_ Relying solely on $RQ$ ignores link asymmetry. While $C \to A$ is clear ($100\%$ reception), outbound link $A \to C$ drops over $37.5\%$ of transmitted data packets ($TQ_{\text{local}} = 0.625$). Selecting $C$ would result in high data loss.

---

Exercise 3: Kinematic Lifetime vs. Store-Carry-Forward Latency

**Scenario Setup:** Drones fly at speed $v = 8\text{ m/s}$ with an IEEE 802.11 Wi-Fi transmission range of $r = 140\text{ m}$.

**(a) Calculate the maximum communication link duration for two drones flying head-on in opposite directions.**

_Step-by-Step Solution:_

- Closing relative speed: $v_{\text{relative}} = 8\text{ m/s} + 8\text{ m/s} = 16\text{ m/s}$.
- Total path length inside radio coverage: $D = 2 \times r = 2 \times 140\text{ m} = 280\text{ meters}$.
- Maximum link lifetime: $$T_{\text{max}} = \frac{D}{v_{\text{relative}}} = \frac{280\text{ m}}{16\text{ m/s}} = 17.5\text{ seconds}$$

**(b) A drone carrying data is** $600\text{ m}$ **from a ground depot with no intermediate relay nodes. Calculate the delay to deliver data using Store-Carry-Forward.**

_Step-by-Step Solution:_

- Distance to depot: $d_{\text{initial}} = 600\text{ m}$.
- Depot Wi-Fi coverage range: $r = 140\text{ m}$.
- Flight distance required to reach radio range: $$d_{\text{flight}} = 600\text{ m} - 140\text{ m} = 460\text{ meters}$$
- Physical delivery delay: $$t_{\text{delivery}} = \frac{d_{\text{flight}}}{v} = \frac{460\text{ m}}{8\text{ m/s}} = 57.5\text{ seconds}$$

**(c) Synthesize the structural takeaways of parts (a) and (b). Why does Stop & Route halt movement before offloading?**

_Architectural Takeaway:_ Unmanaged wireless links break in seconds ($17.5\text{ s}$ lifetime), while physical payload transport incurs significant delays ($57.5\text{ s}$ latency) and consumes substantial flight energy. Stop & Route halts drone movement during data offloading to lock the network into a temporary static topology. Hovering stabilizes wireless links, eliminates topological jitter, and enables data payload transfer over high-speed multi-hop Wi-Fi paths.

---

Concept Verification Questions (True / False)

1. **Proactive routing protocols react quickly to topology changes in a FANET.**
    - **Answer:** **FALSE**
    - _Justification:_ Proactive protocols rely on periodic control updates to recalculate routing tables. In high-mobility FANETs where topologies shift within seconds, background updates generate excessive control overhead and fail to converge fast enough, resulting in stale routes and lost packets.
2. **Greedy geographic forwarding always finds a path to the destination if one exists.**
    - **Answer:** **FALSE**
    - _Justification:_ Greedy forwarding relies strictly on local distance metrics and can get trapped at a dead end (local optimum) when no neighbor is closer to the target than the current node, even if a valid multi-hop path exists around the void.
3. **B.A.T.M.A.N. evaluates link quality solely by counting the number of OGMs it receives from a neighbor.**
    - **Answer:** **FALSE**
    - _Justification:_ Counting only received OGMs ($RQ$) fails over asymmetric links. B.A.T.M.A.N. tracks both Receive Quality ($RQ$) and Echo Quality ($EQ$) to compute local Transmit Quality ($TQ = EQ / RQ$), evaluating the link in the actual outbound direction used for data forwarding.
4. **The Stop & Route framework utilizes controlled node mobility to construct connected network topologies.**
    - **Answer:** **TRUE**
    - _Justification:_ True. Stop & Route harnesses controlled node mobility by directing drones to hover at specific rendezvous points, transforming an unconnected network into a stable, multi-hop Wi-Fi topology for periodic data offloading.

---

Summary

Mastering lower-level link dynamics, geometric routing heuristics, and managed mobility forms the mandatory prerequisite for designing fully autonomous, reinforcement-learning-driven aerial networks. System engineers can leverage these deterministic building blocks to formulate formal Markov Decision Processes, enabling next-generation drone fleets to learn optimal operational policies autonomously via Reinforcement Learning.