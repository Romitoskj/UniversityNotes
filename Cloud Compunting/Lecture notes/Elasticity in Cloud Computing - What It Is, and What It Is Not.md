**The Problem with "Elasticity"** Despite being a central concept in cloud computing, the term "elasticity" is often used inconsistently as a marketing buzzword and lacks a precise definition. The authors point out that existing definitions—including those from NIST, IBM, and others—often confuse elasticity with related concepts like scalability and efficiency, and fail to provide ways to actually quantify it.

**A New, Precise Definition** To solve this, the authors propose a new definition: **Elasticity is the degree to which a system can automatically adapt to workload changes by provisioning and de-provisioning resources, ensuring that the available resources** **match** **the current demand as closely as possible at any given time**.

**Key Differentiations (Crucial for Students to Know)** The paper explicitly separates elasticity from similar terms:

- **Scalability vs. Elasticity:** Scalability is the ability to handle growing workloads by adding resources, making it a _prerequisite_ for elasticity. However, scalability does not account for time. Elasticity adds the temporal dimension—how fast, how often, and at what granularity the system scales to constantly match demand.
- **Efficiency vs. Elasticity:** Efficiency is the amount of resources consumed to process a specific amount of work. While an elastic system often leads to higher efficiency (by avoiding over-provisioning), an inherently efficient system is not necessarily elastic.

**The Two Core Aspects of Elasticity** When evaluating how elastic a system is for a specific resource (like CPU or RAM), the authors identify two main dimensions:

1. **Speed:** The time it takes the system to transition from an underprovisioned state to an optimal state (scaling up), or from an overprovisioned state to an optimal state (scaling down).
2. **Precision:** The absolute deviation between the amount of resources currently allocated and the actual resource demand needed at that moment.

**Measurement and Benchmarking** To quantify elasticity, the authors suggest deriving a **"matching function"**—a baseline that defines exactly how many resources a system needs for a specific workload to meet performance goals. Using this baseline, they propose metrics that calculate the time spent in under/over-provisioned states and the volume of under/over-provisioned resources.

Finally, they note a major challenge in **benchmarking**: to fairly compare the elasticity of two different systems, testers must induce identical _resource demand curves_ rather than just identical user workloads. This is because a highly efficient system will demand fewer resources for the exact same user workload, meaning its elasticity mechanisms won't be stressed to the same degree as a less efficient system.