
# Cloud Computing Definition and Architecture
## **1. Deconstructing the Prerequisites: Scalability vs. Elasticity**

- **Definition:** Elasticity is the degree to which a system can automatically adapt to workload changes by provisioning and de-provisioning resources, ensuring that the available resources match the current demand as closely as possible at any given time.
- **The Difference:** While scalability is the ability of a system to handle increasing workloads by adding resources, it does not consider how well resource demands are matched by provisioned resources at any specific point in time. Scalability is a prerequisite for elasticity, but it lacks a temporal dimension.
- **Temporal Aspects:** Elasticity distinguishes itself by introducing temporal aspects, specifically determining "how fast, how often, and at what granularity" scaling actions occur. A scalable system is not inherently elastic if it requires manual intervention or takes hours to adapt, failing to match demand continuously. Some industry definitions, like the NIST definition of "Rapid Elasticity," have been critiqued for blurring these lines by stating scaling can be "in some cases automatically" and portraying an idealistic scenario of unlimited resources.

## **2. The "Matching" Criterion and Asymmetry**

- **The Matching Function:** A system-specific matching function $m(w)=r$ maps a workload intensity ($w$) to the minimal amount of resources ($r$) needed to satisfy performance requirements.
- **Asymmetry of Scaling:** This function must be derived independently for scaling up and scaling down because the processes are asymmetric. It cannot be assumed that the optimal resource allocation level when transitioning from an underprovisioned state (scaling upwards) will be identical to transitioning from an overprovisioned state (scaling downwards).
- **System Stability:** After a workload change, systems require a "stable state" to adapt. The technical reaction time and resource overhead for adding a Virtual Machine (VM) often differs significantly from deprovisioning one.

## **3. Metrics of Speed vs. Precision**

Elasticity is evaluated using two core metrics:

- **Speed:** The time taken to switch between provisioned states (e.g., switching from underprovisioned to an optimal or overprovisioned state).
- **Precision:** The absolute deviation of the current amount of allocated resources from the actual resource demand.
- **The Trade-off Impact:**
    - _For the Provider:_ If scaling is highly rapid but lacks precision (overshooting), the system will overprovision, wasting resources and leading to poor efficiency.
    - _For the User:_ If scaling is rapid but undershoots the target (low precision), the system enters an "underprovisioned" state, leading to performance variability and a failure to meet performance requirements.

## **4. Elasticity vs. Efficiency in Benchmarking**

- **The Paradox:** Efficiency expresses the amount of resources consumed to process a given amount of work. If two systems—one highly efficient and one inefficient—are tested with an **identical workload**, the more efficient system will consume fewer resources, meaning its adaptation mechanisms are not stressed to the same extent. This makes the highly efficient system appear artificially more elastic.
- **Fair Comparison:** To accurately compare the elasticity of different systems, a benchmark must induce **identical demand curves**, not just identical workloads. This requires adjusting the workload (e.g., doubling it for the more efficient system) so that both systems' adaptation mechanisms are exercised at similar intensities.
	
	![](Images/Pasted%20image%2020260527172807.png)

## **5. The Impact of Discrete Scaling Units**

- **Discrete vs. Continuous:** In reality, resources (such as CPU cores, VMs, or physical nodes) are provisioned in "discrete units". However, real-world user demand usually follows a continuous curve.
- **Optimal Elasticity is Hypothetical:** Because supply is stepped and demand is continuous, "optimal elasticity" (where resources exactly match demand at all times) is purely hypothetical. There will always be a deviation or precision error, meaning systems will generally provide the minimal amount of resources needed, resulting in slight overprovisioning.

## **6. Benchmarking Real-Life Scenarios**

Different types of workload patterns stress different elasticity metrics:

- **Burst Patterns:** During a sudden, intense spike in demand, the most critical metric is the average speed of scaling up ($\overline{A}$). If this speed is too slow, the system gets trapped in an underprovisioned state, failing to meet the demand spike.
- **Seasonal Patterns:** For predictable downward trends, the average precision of scaling down ($P_d$) becomes the critical metric. Poor precision during a scale-down means the system remains overprovisioned for too long after the peak has passed, resulting in unnecessary costs and wasted resources.

---

# Elasticity in Cloud Computing:

**A New, Precise Definition** To solve this, the authors propose a new definition: **Elasticity is the degree to which a system can automatically adapt to workload changes by provisioning and de-provisioning resources, ensuring that the available resources** **match** **the current demand as closely as possible at any given time**.

**Key Differentiations (Crucial for Students to Know)** The paper explicitly separates elasticity from similar terms:

- **Scalability vs. Elasticity:** Scalability is the ability to handle growing workloads by adding resources, making it a _prerequisite_ for elasticity. However, scalability does not account for time. Elasticity adds the temporal dimension—how fast, how often, and at what granularity the system scales to constantly match demand.
- **Efficiency vs. Elasticity:** Efficiency is the amount of resources consumed to process a specific amount of work. While an elastic system often leads to higher efficiency (by avoiding over-provisioning), an inherently efficient system is not necessarily elastic.

**The Two Core Aspects of Elasticity** When evaluating how elastic a system is for a specific resource (like CPU or RAM), the authors identify two main dimensions:

1. **Speed:** The time it takes the system to transition from an underprovisioned state to an optimal state (scaling up), or from an overprovisioned state to an optimal state (scaling down).
2. **Precision:** The absolute deviation between the amount of resources currently allocated and the actual resource demand needed at that moment.

**Measurement and Benchmarking** To quantify elasticity, the authors suggest deriving a **"matching function"**—a baseline that defines exactly how many resources a system needs for a specific workload to meet performance goals. Using this baseline, they propose metrics that calculate the time spent in under/over-provisioned states and the volume of under/over-provisioned resources.

Finally, they note a major challenge in **benchmarking**: to fairly compare the elasticity of two different systems, testers must induce identical _resource demand curves_ rather than just identical user workloads. This is because a highly efficient system will demand fewer resources for the exact same user workload, meaning its elasticity mechanisms won't be stressed to the same degree as a less efficient system.