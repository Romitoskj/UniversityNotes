# Autoscaling

## 1. The Auto-Scaling Process

Autoscaling refers to the capability of automatically adjusting the number of computational resources (such as VMs, containers, etc.) allocated to an application, based on the current workload.

Auto-scaling systems continuously manage cloud resources by following the **MAPE loop** from autonomic computing:

- **Monitoring (M):** Collects data about user demands, application status, and SLA compliance. Because obtaining detailed application-level metrics is difficult, auto-scalers often rely on **proxy metrics**, such as hypervisor-level CPU utilization, to approximate the real workload.
- **Analysis (A):** Evaluates the collected data to estimate current utilization or predict future resource demands.
- **Planning (P):** Determines the appropriate scaling adjustments (e.g., adding or removing Virtual Machines).
- **Execution (E):** The service provider carries out the decided adjustments.

#### Vertical vs. Horizontal Scaling

- **Vertical Scaling** Adjusting the resources CPU, memory) within a single instance.
- **Horizontal Scaling** Adding or removing instances to handle the load, also known as scale-out/in. 

#### Reactive vs. Proactive Auto-scaling:

- **Reactive:** Responds to changes only _after_ they are detected. Because it takes time to boot up and configure a new Virtual Machine (VM), reactive systems often suffer from delayed responses, leading to temporary SLA violations during sudden traffic bursts.
- **Proactive:** Anticipates future demands to provision resources in advance, which is crucial for handling sudden bursts effectively.

## 2. Core Scaling Risks

Any auto-scaling system must navigate three primary risks:

- **Under-provisioning:** The application lacks sufficient resources to meet its Service Level Agreement (SLA), leading to system congestion and poor performance during traffic spikes.
- **Over-provisioning:** The application is assigned more resources than necessary. While performance is met, the client pays unnecessary costs for idle machines.
- **Oscillation:** Rapidly bouncing between adding and removing resources. This happens when the system executes a scaling action too quickly before the impact of the previous action is fully realized.

## 3. Auto-Scaling Techniques

### A. Threshold-Based Rules

- **Structure:** Consists of a condition (evaluating a metric against upper/lower thresholds) and an action (adding/removing a fixed number of VMs).
- **Cooldown Periods:** A "calm" period is strictly necessary to prevent **oscillation**. It forces the auto-scaler to wait after an action, allowing the 5-10 minute VM boot-up process to finish before evaluating the system again.
- **Threshold Spacing:** Setting the upper threshold equal to the lower threshold completely eliminates the "capacity buffer", causing severe oscillation as the system immediately reacts to any minor fluctuation.
- **Multiple Thresholds:** Defining multiple threshold tiers (e.g., add 1 VM at 60%, add 2 VMs at 70%) is more effective than a single rigid rule. It prevents over-provisioning for minor load increases while allowing rapid scaling (preventing slow convergence) during massive traffic spikes.
- **Duration Conditions:** A condition like "if metric > X _for Y seconds_" acts as a **Conservative** triggering strategy, filtering out temporary workload spikes before enacting a change.

### B. Time Series Analysis (TSA)

- Operates in the analysis phase to find repeating patterns in historical data and forecast future resource demands.
- It is the primary enabler of **proactive** auto-scaling, allowing systems to acquire resources _before_ a predicted traffic burst occurs, avoiding boot-up delays.

### C. Control Theory (CT)

- **Feedback Controllers:** Monitor actual system output and correct deviations _after_ they occur.
- **Feed-forward Controllers:** Proactively use a system model to predict behavior and adjust resources before a deviation happens.
- These are often **combined** because predictive models can fail; the feedback controller acts as a safety net to correct any residual errors or miscalculations from the feed-forward predictions.

## D. Queuing Theory (QT)

- Models applications as a system of waiting lines (e.g., the load balancer) and servers (e.g., the VMs) to estimate metrics like response time using the relationship between arrival rates and service rates.
- **Limitations:** QT models (like M/M/1) are extremely rigid, assume stationary systems (constant arrival/service rates), and are computationally expensive to recalculate dynamically. They only analyze the system and must be paired with a separate planning algorithm to form a complete auto-scaler.

### E. Reinforcement Learning (RL)

- A trial-and-error approach that requires no _a-priori_ system knowledge.
- **Disadvantages:** It suffers from bad initial performance and unfeasibly long training times. It also faces the **"Curse of Dimensionality"** (exponentially growing state spaces) and struggles to adapt to sudden traffic bursts because it relies on random exploratory actions.
- _Hybrid Use:_ RL can be combined with CT, such as using an RL agent to continuously learn and estimate the derivative term of a PID controller.

## 4. Evaluating and Triggering Auto-scaling Strategies

### The Auto-Scaling Demand Index (ADI)

- ADI is a metric that evaluates a strategy by calculating the total sum of deviations between actual resource usage and the user's desired target range.
- **Advantage:** It provides a unified score that equally penalizes two opposing operational issues: **deteriorated QoS** (utilization goes above the maximum limit) and **wasted IT costs** (utilization drops below the minimum limit).

### Triggering Strategies:

- **Conservative Triggering:** By waiting for confirmation over consecutive time-steps before acting, this strategy ignores momentary spikes, significantly reducing the constant turning on and off of machines.

### Step Size Strategies:

- **Fixed Step Size Problem:** If the fixed number of machines added/removed is too small, the system suffers from slow response times. If the number is too large, the system overreacts and misses the optimal utilization target.
- **Adaptive Step Size:** Solves the fixed-step inefficiencies for peaky or bursty workloads by mathematically calculating the exact step size in real-time based on current utilization, instantly resizing the pool to reach a satisfactory state. Note that using multiple threshold rules (e.g., adding 1 vs. 2 VMs) acts as a _pseudo-adaptive_ strategy, though it is not as mathematically fluid.

## 5. Experimental Methods for Cloud Research

- **Simulation vs. Real Platforms:** Cloud simulators (like CloudSim) are vastly superior for testing algorithms because they provide time efficiency (minutes vs. hours), isolated and controlled environments free of external interference, massive cost savings, and highly simplified monitoring setups compared to public clouds.
- **Workloads:** Real-world traces (like _ClarkNet_ or _World Cup 98_) are preferred over synthetic workloads because they capture genuine user demand, burstiness, and complex patterns that synthetic generators struggle to replicate realistically.
- **Benchmarks:** Researchers commonly use application benchmarks like **RUBiS** (an eBay-like auction simulator) and **TPC-W** (an online e-commerce bookshop simulator) to generate test environments.