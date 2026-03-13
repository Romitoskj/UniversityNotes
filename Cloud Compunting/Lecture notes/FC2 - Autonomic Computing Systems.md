### 1. The Vision of Autonomic Computing & Self-Management

The primary motivation behind autonomic computing is the looming software complexity crisis: modern IT systems are becoming too massive and interconnected for human administrators to manually install, configure, and maintain. The solution is computing systems that manage themselves according to high-level goals set by administrators.

- **Self-Management in Practice (Self-CHOP):**
    - **Self-configuration:** Automated configuration of components following high-level policies (e.g., seamlessly integrating new resources).
    - **Self-healing:** Detecting, diagnosing, and repairing localized software and hardware problems. _Example: An Autonomic Manager monitoring health states and automatically spinning up new application instances when the number of healthy instances drops below a threshold._
    - **Self-optimization:** Continually seeking opportunities to improve performance and efficiency. _Example: Dynamically adding or removing Virtual Machines to meet Service Level Objectives (SLOs) like response time and availability while minimizing costs_
    - **Self-protection:** Automatically defending against malicious attacks and cascading failures.

### 2. Foundations and Inspirations (Chapter 3 AC)

Autonomic computing consolidates theories from various non-computing domains to manage complexity.

- **The Biological Metaphor:**
    - **Autonomic Nervous System (ANS):** In biology, the ANS subconsciously regulates vital functions to ensure survival. In computing, this equates to the **Autonomic Manager**, which controls low-level, subconscious activities.
    - **Conscious Brain:** Represents the **Human Administrator**, who specifies high-level objectives and only intervenes during major failures or to change system goals.
    - **Motility:** The biological emergence of motility (movement) necessitated a nervous system because movement brings organisms into changing environments. Similarly, dynamic software programs require "change management" coordination to adapt to unpredictable shifts.
    - **Homeostasis & Vital Parameters:** Homeostasis is a system’s ability to maintain internal equilibrium despite external or internal changes. In computing, the "vital parameters" are functional services and Quality of Service (QoS) properties that must be kept within a defined "survivability" or viability zone.
- **Economics & Game Theory:**
    - **Microeconomics in Resource Allocation:** Concepts like supply and demand patterns and "virtual currencies" are used as negotiation mechanisms for autonomic processes competing for limited resources (like CPU cycles or bandwidth).
    - **Nash Equilibrium vs. Pareto Optimality:** A Nash Equilibrium is a local optimum where no participant benefits from unilaterally changing their strategy. Pareto Optimality is a global optimum where no one can be made better off without making someone else worse off; moving a system to the Pareto boundary requires cooperation among players.
    - **Stackelberg vs. Conjectural Equilibrium:** Stackelberg leadership applies when one player has _private knowledge_ of competitors and optimizes accordingly. Conjectural equilibrium is preferred when players have _imperfect knowledge_; they rely on beliefs (conjectures) built through repeated interactions.
- **Complex Adaptive Systems (CAS) & Cybernetics:**
    - **CAS Definition:** A system that modifies itself in response to environmental changes and learns from experience. It is critical to distinguish a CAS (which features "nonlinear composition" where the whole is not a simple sum of parts) from a system that is merely "extremely complicated" (having many parts).
    - **Emergence and Self-Organization:** Self-organization consists of simple, local interactions between agents. Emergence is the coherent, complex global behavior that arises from these local interactions.
    - **Ashby’s Ultra-Stable System:** A double-feedback control concept. The **first loop** is reactive, handling small, short-term disturbances. The **second loop** (meta-management) operates over a longer time scale and adapts the behavior of the first loop when "essential variables" hit their viability limits.

### 3. Autonomic Architectures (Chapter 4 AC)

- **The MAPE-K Loop:** The standard logical architecture for an autonomic manager.
    - **Monitor:** Collects details and data from managed resources.
    - **Analyze:** Processes data to determine if a system change is needed.
    - **Plan:** Creates a specific set of actions to achieve the system's goals.
    - **Execute:** Applies the planned actions to the managed resources.
    - **Knowledge (K):** The heart of the architecture; the central repository of models and data that informs every active stage of the loop.
- **Touchpoints:** The interfaces bridging the autonomic manager and the managed artefacts. **Sensors** allow the manager to apprehend the resource's state, while **effectors** allow it to influence the resource. This allows the manager to control artefacts that are too complex to be completely known internally.
- **Types of Knowledge:**
    - **Innate Knowledge:** Engraved into the heart of the manager at design time, capturing domain expertise.
    - **Knowledge by Acquaintance (Acquired):** Captured dynamically at runtime via touchpoints from the managed environment.
- **Reasoning & Decision-Making Approaches:**
    - **Rule-Based (Reflex) Reasoning:** Uses Event-Condition-Action (ECA) rules. While they are fast and simple for rapid responses, they are hardwired and can fail when dramatic, unforeseen environmental changes occur.
    - **Model-Driven Autonomicity:** Uses explicit models to support reasoning. Managers typically maintain three types of models: Structure (components and connectors), Environment (external factors), and Non-functional (security, performance). These are separated so they can evolve independently. To be effective, architectural models must be **"causally connected,"** meaning changes in the real system are reflected in the model and vice versa.
    - **Goal-Based vs. Utility-Based Systems:** Goal-based systems evaluate actions based on whether they reach a specific predefined state. Utility-based systems assign a "score" to different states, which is necessary when goals conflict, allowing the system to make trade-offs and choose the best overall outcome.
    - **Search-Based Reasoning:** Defines problem-solving as finding a path from a current state to a target state. It requires three components: (1) A representation of the initial state, (2) A set of possible operations/actions, and (3) A goal test or utility function to evaluate the end state.
    - **Probabilistic Reasoning and Uncertainty:** In dynamic environments, sensor data is often noisy or conflicting. Systems use a **Probability of Correctness (PoC)** to weigh inputs from multiple Context Providers (sensors). Combining this data reduces uncertainty and allows the system to reach reliable conclusions.
    - **Learning Systems:** The most ambitious type of autonomic system. They utilize techniques like **reinforcement learning** (inspired by biological reward signals) to evaluate past actions. This allows the system to update the manager's knowledge base and adapt its utility functions over time.