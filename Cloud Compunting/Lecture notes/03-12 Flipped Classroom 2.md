# Autonomic Computing Systems

> [!important] Definition
> Autonomic computing systems are computing systems that can manage themselves given **high-level objectives** from administrators.

## 1. The Vision of Autonomic Computing & Self-Management

The primary motivation behind autonomic computing is the looming software complexity crisis: modern IT systems are becoming too massive and interconnected for human administrators to manually install, configure, and maintain. The solution is computing systems that manage themselves according to high-level goals set by administrators.

### The Four Self-Properties (Self-CHOP):

- **Self-configuration:** Automated configuration of components following high-level policies (e.g., seamlessly integrating new resources).
- **Self-healing:** Detecting, diagnosing, and repairing localized software and hardware problems. _Example: An Autonomic Manager monitoring health states and automatically spinning up new application instances when the number of healthy instances drops below a threshold._
- **Self-optimization:** Continually seeking opportunities to improve performance and efficiency. _Example: Dynamically adding or removing Virtual Machines to meet Service Level Objectives (SLOs) like response time and availability while minimizing costs_
- **Self-protection:** Automatically defending against malicious attacks and cascading failures.

## 2. Autonomic Architectures (Chapter 4 AC)

### The MAPE-K Loop:

The standard logical architecture for an autonomic manager.
- **Monitor:** Collects details and data from managed resources.
- **Analyze:** Processes data to determine if a system change is needed.
- **Plan:** Creates a specific set of actions to achieve the system's goals.
- **Execute:** Applies the planned actions to the managed resources.
- **Knowledge (K):** Throughout the MAPEK cycle, the autonomic manager relies on a knowledge base, which contains information about the system, its components, historical data, policies, and best practices. This knowledge is crucial for making informed decisions during the analysis and planning phases. The knowledge base can also be updated as the system learns from new data and experiences.

### Touchpoints:

The interfaces bridging the autonomic manager and the managed artefacts. **Sensors** allow the manager to apprehend the resource's state, while **effectors** allow it to influence the resource. This allows the manager to control artefacts that are too complex to be completely known internally.

### Types of Knowledge:

- **Innate Knowledge:** Engraved into the heart of the manager at design time, capturing domain expertise.
- **Knowledge by Acquaintance (Acquired):** Captured dynamically at runtime via touchpoints from the managed environment.

### Reasoning & Decision-Making Approaches:

- **Rule-Based (Reflex) Reasoning:** Uses Event-Condition-Action (ECA) rules. While they are fast and simple for rapid responses, they are hardwired and can fail when dramatic, unforeseen environmental changes occur.
- **Model-Driven Autonomicity:** Uses explicit models to support reasoning. Managers typically maintain three types of models: Structure (components and connectors), Environment (external factors), and Non-functional (security, performance). These are separated so they can evolve independently. To be effective, architectural models must be **"causally connected,"** meaning changes in the real system are reflected in the model and vice versa.
- **Goal-Based vs. Utility-Based Systems:** Goal-based systems evaluate actions based on whether they reach a specific predefined state. Utility-based systems assign a "score" to different states, which is necessary when goals conflict, allowing the system to make trade-offs and choose the best overall outcome.
- **Search-Based Reasoning:** Defines problem-solving as finding a path from a current state to a target state. It requires three components: (1) A representation of the initial state, (2) A set of possible operations/actions, and (3) A goal test or utility function to evaluate the end state.
- **Probabilistic Reasoning and Uncertainty:** In dynamic environments, sensor data is often noisy or conflicting. Systems use a **Probability of Correctness (PoC)** to weigh inputs from multiple Context Providers (sensors). Combining this data reduces uncertainty and allows the system to reach reliable conclusions.
- **Learning Systems:** The most ambitious type of autonomic system. They utilize techniques like **reinforcement learning** (inspired by biological reward signals) to evaluate past actions. This allows the system to update the manager's knowledge base and adapt its utility functions over time.