# Class 1 Notes: Introduction to Autonomous Networking (A.Y. 2026–2027)

## 1. Defining Autonomy in Networking

In the context of this course, autonomy is characterised by a continuous four-stage cycle known as the **Autonomous Loop**:

- **Observe:** The system monitors the environment and its internal operating conditions, including traffic patterns, channel states, network topology, and energy levels.
- **Decide:** The system selects the most appropriate networking action based on the gathered observations.
- **Act:** The system executes the decision—such as modifying transmission parameters, routing, or mobility—without requiring continuous human intervention.
- **Adapt:** The system reacts to changing conditions in real-time to maintain optimal performance.

### Practical Scenario: Smart Home Congestion Management

In a typical smart home, numerous devices share a single communication channel. Under standard conditions, high-bandwidth video streaming may dominate the channel capacity.

When a critical event occurs, such as smoke detection in the kitchen, an autonomous network responds as follows:

1. **Observe:** The network recognises the smoke sensor's signal and detects that the shared channel is currently congested by entertainment traffic.
2. **Decide:** The system determines that emergency sensor data must be prioritised over video streaming to ensure safety.
3. **Act:** It executes this decision by **changing the channel-access behaviour**, ensuring kitchen devices can transmit without delay.
4. **Adapt:** The network successfully reconfigures its operation to satisfy the demands of the new, critical operating environment.

## 2. Core Application Domains

The course focuses on a variety of networking systems, which are **mainly wireless** in nature:

- **Wireless Sensor Networks:** Dedicated to sensing the physical world.
- **Internet of Things (IoT):** Large-scale infrastructures connecting devices at scale.
- **RFID and Backscatter Networks:** Communication protocols designed for limited energy environments.
- **Unmanned Aerial Networks (Dronets):** Providing aerial connectivity and high mobility.
- **Robophysical Networks:** Focused on collective behaviour and networked robotic systems.
- **Network Performance Evaluation:** Integrating basic notions of metrics, modelling, and experimentation.

## 3. Foundations of Reinforcement Learning (RL)

Reinforcement Learning serves as the core framework for autonomous decision-making within this course. The approach is built upon three pillars:

- **Pillar 1: Sequential decision-making:** Learning how an agent can make a series of interconnected decisions over time.
- **Pillar 2: Utility/Goodness measures:** Defining a metric to evaluate the quality of decisions.
- **Pillar 3: Learning through experience/uncertainty:** Developing strategies through trial and error, as the agent does not know in advance how its actions will affect the environment.

### RL Problem Structure and Challenges

- **Problem Formulation:**
    - **States / Observations:** The data representing the environment's current status.
    - **Actions:** The set of possible choices available to the agent.
    - **Rewards:** The feedback signal indicating the success of an action.
- **Core Challenges:**
    - **Exploration vs. Exploitation:** Balancing the search for new strategies against the use of known successful ones.
    - **Delayed Consequences:** Managing scenarios where the reward for an action is not immediate.
- **RL Techniques Covered:**
    - Multi-Armed Bandits.
    - Q-Learning.

## 4. Conclusion and Forward Look

The course facilitates a conceptual shift from merely identifying "networking problems" to engineering "autonomous decision-making" solutions. This involves both learning-based approaches (such as adaptive routing) and adaptive approaches that function without learning (such as rule-based coordination).

### Next Class

The next session will begin a technical overview of wireless technologies, focusing on the specific operational requirements of:

- RFID.
- Sensor networks.
- IoT networks.
- Backscattering networks.
- Dronets.