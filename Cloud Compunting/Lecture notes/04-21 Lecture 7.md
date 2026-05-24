# Performance Evaluation

## Overview

Performance evaluation assesses how efficiently a computer system utilizes its underlying resources, such as CPU, memory, and storage, to achieve desired outputs. While there are multiple techniques (like analytical or simulation modeling), cloud computing primarily relies on **empirical or testbed analysis**, which involves monitoring metrics and conducting tests on a deployed system. This evaluation helps identify bottlenecks, optimizing resource utilization, ensure business continuity, improving scalability and flexibility, and verify that Service Level Agreements (SLAs) are met.

## Key Performance Metrics

Metrics are categorized into two main perspectives:

- **System-oriented (Server-side):** Focuses on infrastructure efficiency. Key metrics include:
	- **Throughput:** The amount of data or work a system can process within a given timeframe (e.g., transactions, read/write blocks, or requests processed per second),
	- **Resource Utilization** (CPU, memory, disk usage),
	- **Scalability:** The ability of the system to handle increasing workloads or demands without compromising performance.
- **User-oriented (Client-side):** Focuses on the quality of service perceived by the end-user. Key metrics include:
	- **End-to-End Response Time:** The total time from sending a request to receiving the result over the network,
	- **Availability:** The percentage of time a service is operational and accessible to users,
	- **Error Rate:** The frequency of errors or failures within the system.

## Performance Testing Techniques 

To test system behavior, researchers generate different workload conditions:

- **Load Testing:** Evaluates system performance under normal operating conditions.
- **Stress Testing:** Exposes the system to continuous extreme loads to identify capacity limits and weaknesses.
- **Spike Testing:** Simulates sudden, unpredictable bursts of activity to evaluate the system's reaction to load spikes.
- **Endurance Testing:** Runs the system over an extended period to identify issues that manifest over time.

## **The 7-Step Evaluation Methodology**

When evaluating cloud services, a rigorous seven-step procedure is standard:

**1. Specify Purpose and Scope** Determine the exact goal (e.g., identifying bottlenecks, verifying auto-scaling) and the perspective you are taking (user vs. service provider).

**2. Identify Cloud Features to Evaluate** Modern cloud architectures have many features (e.g., front-end web servers, app logic, data stores, API gateways). A performance study can focus on all, few or one isolated specific component. That allows to determine the appropriate workload to generate and the relative performance metrics to collect.

**3. Determine Performance Metrics** Select the exact metrics to track. Cloud providers offer built-in tools (like AWS CloudWatch) that track server-side metrics:

- **EC2 Instances:** `CPUUtilization`, `NetworkIn`/`NetworkOut`, `DiskReadOps`.
- **Auto-Scaling Groups:** `GroupInServiceInstances`, `GroupPendingInstances`.
- **AWS Lambda:** `ConcurrentExecutions`, `Duration`, `Invocations`. 

User-oriented metrics (like end-to-end latency) cannot be measured internally and must be collected on the client side by the workload generator. These metrics are also influenced by everything between the client and the server.

**4. Select the Appropriate Tool** Choose a workload generator based on the complexity of your system.

- **ApacheBench (ab):** A simple command-line tool, but limited to sending a single type of static request.
- **JMeter or Artillery:** Highly sophisticated tools that allow you to define testing durations, mix different request types, and evaluate complex web applications.

**5. Design Performance Evaluation Experiments** Each experiment must explicitly define the **workload type** (CPU, disk, memory, or network intensive) and its **intensity**, the **duration** and the **number of time** a single experiment is executed (cannot rely on a single outcome).

- **Workload Shape:** For continuous scaling tests, the load should transition smoothly through four phases: a **Warm-up (WU)** period to activate components, a **Ramp-up (RU)** period to increase load gradually, a **Steady (S)** period at maximum load, and a **Ramp-down (RD)** period to observe scale-in.
	![](Images/Pasted%20image%2020260511160927.png)
- **Repetition:** You cannot rely on a single experimental outcome due to network variances. In theory, experiments should be repeated 20 times; in practice, running them 3 to 10 times is acceptable to calculate an accurate average for the project.
- **Common Mistake:** When testing scale-out policies, do not use simple CPU stress tools that only run on the first instance. When a new instance scales out, the tool must automatically run on the new instance as well, or the average CPU load will drop artificially.

**6. Setup the Experimental Environment** To avoid having your cloud provider classify your intensive testing as a malicious attack and ban your account, **run your workload generation tools from virtual machines inside the AWS perimeter**. While this prevents you from measuring real-world internet latency, it ensures the test executes without network blocking. Be careful not to generate load that exceeds platform maximums (e.g., AWS Lambda's default concurrency limits). Configure CloudWatch to collect all the planned metrics and prepare the configuration file for the workload generation tool.

![](Images/Pasted%20image%2020260511161528.png)

**7. Run and Analyze** Once tests complete, **download the raw datapoints** for your metrics, calculate the averages across all runs, and plot them manually to show the correlation between workload intensity and system response. **Do not rely on screenshots of the AWS CloudWatch dashboard.** Screenshots are poor practice because they only reflect a single run, the graphs are hard to read, and AWS automatically deletes the data after a few months.

![472](Images/Pasted%20image%2020260511162102.png)

**Setting Thresholds and Testing Availability**

- **Auto-Scaling Thresholds:** Setting metric thresholds is largely a heuristic process. A strong rule of thumb is to set your scaling threshold at **66% of the system's maximum capacity** (e.g., if max load that the test generates  to the CPU is 50% utilization, the threshold can be set to ~35% for the project). Another way it is setting an adaptive threshold.
- **Availability Testing:** To test system resilience, you should generate random failures manually by killing active services or forcefully stopping running VMs/containers.