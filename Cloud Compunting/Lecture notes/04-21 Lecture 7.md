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
	- **Availability:** The percentage of time a service is operational and accessible to users. ,
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

**2. Identify Cloud Features to Evaluate** Modern cloud architectures have many layers (e.g., front-end web servers, app logic, data stores, API gateways). You must isolate the specific components under test to determine the appropriate workload to generate. _Note: Analyzing the scalability of database layers is often avoided in simple tests because it requires generating extremely intensive workloads_.

**3. Determine Performance Metrics** Select the exact metrics to track. Cloud providers offer built-in tools (like AWS CloudWatch) that track server-side metrics:

- **EC2 Instances:** `CPUUtilization`, `NetworkIn`/`NetworkOut`, `DiskReadOps`.
- **Auto-Scaling Groups:** `GroupInServiceInstances`, `GroupPendingInstances`.
- **AWS Lambda:** `ConcurrentExecutions`, `Duration`, `Invocations`. User-oriented metrics (like end-to-end latency) cannot be measured internally and must be collected on the client side by the workload generator.

**4. Select the Appropriate Tool** Choose a workload generator based on the complexity of your system.

- **ApacheBench (ab):** A simple command-line tool, but limited to sending a single type of static request.
- **JMeter or Artillery:** Highly sophisticated tools that allow you to define testing durations, mix different request types, and evaluate complex web applications.

**5. Design Performance Experiments** Your experiment must explicitly define the workload type (CPU, disk, memory, or network intensive) and its intensity.

- **Workload Shape:** For continuous scaling tests, the load should transition smoothly through four phases: a **Warm-up (WU)** period to activate components, a **Ramp-up (RU)** period to increase load gradually, a **Steady (S)** period at maximum load, and a **Ramp-down (RD)** period to observe scale-in.
- **Repetition:** You cannot rely on a single experimental outcome due to network variances. In theory, experiments should be repeated 20 times; in practice, running them 3 to 10 times is acceptable to calculate an accurate average.
- **Common Mistake:** When testing scale-out policies, do not use simple CPU stress tools that only run on the first instance. When a new instance scales out, the tool must automatically run on the new instance as well, or the average CPU load will drop artificially.

**6. Setup the Experimental Environment** To avoid having your cloud provider classify your intensive testing as a malicious attack and ban your account, **run your workload generation tools from virtual machines inside the AWS perimeter**. While this prevents you from measuring real-world internet latency, it ensures the test executes without network blocking. Be careful not to generate load that exceeds platform maximums (e.g., AWS Lambda's default concurrency limits).

**7. Run and Analyze** Once tests complete, **download the raw datapoints** for your metrics, calculate the averages across all runs, and plot them manually to show the correlation between workload intensity and system response. **Do not rely on screenshots of the AWS CloudWatch dashboard.** Screenshots are poor practice because they only reflect a single run, the graphs are hard to read, and AWS automatically deletes the data after a few months.

**Setting Thresholds and Testing Availability**

- **Auto-Scaling Thresholds:** Setting metric thresholds is largely a heuristic process. A strong rule of thumb is to set your scaling threshold at **66% of the system's maximum capacity** (e.g., if max capacity limits your CPU to 50% utilization, the threshold should be ~35%).
- **Availability Testing:** To test system resilience, you should generate random failures manually by killing active services or forcefully stopping running VMs/containers.

## Evaluating Project 3

For Project 3, your objective is to deploy an **AI-based application on AWS Lambda** and evaluate its performance and scalability. Here is how you should apply the 7-step performance evaluation methodology specifically to this project:

**1. Specify Purpose and Scope** Your primary goal is to collect performance measures showing how the application scales as the workload increases. You need to assess how many Lambda functions are needed to guarantee a certain level of service, identify any "hot spot" functions that could act as bottlenecks, measure throughput and response time, and verify high availability. The evaluation perspective should cover both the **client side and the server side**.

**2. Identify Features to Evaluate** You should focus your analysis specifically on the **AWS Lambda function(s) that implement the AI model**. Other peripheral components—such as the authentication service, data store layer, front-end web server, and the actual accuracy or precision of the AI model—can be kept out of the analysis, assuming they do not represent a bottleneck.

**3. Determine Performance Metrics** Because you are evaluating from both perspectives, you must collect two sets of metrics:

- **Server-Side (AWS CloudWatch for Lambda):** Track execution and concurrency metrics such as `Invocations`, `Errors`, `Throttles`, `Duration`, and specifically **`ConcurrentExecutions`** to evaluate scalability.
- **Client-Side (User-oriented):** Track end-to-end **response time/latency**, throughput (requests per second), and availability/error rates using your workload generator tool.

**4. Select the Appropriate Tool** Choose a sophisticated workload generator like **JMeter, Artillery, or Locust**. The tool must be capable of sending requests directly to the API gateway (so you do not need to manually interact with a web page). Because the time it takes to process AI requests depends on the input, your tool must be able to generate a varied mix of requests, such as sending inputs of different sizes or complexities.

**5. Design Performance Experiments**

- **Workload Shape:** Design a continuous workload that smoothly transitions through four phases: a **Warm-up** period to activate components, a **Ramp-up** period, a **Steady** period at maximum load, and a **Ramp-down** period.
- **Intensity:** To avoid having AWS throttle your application or ban your account (especially under AWS Learner Lab limitations), do not generate an overly intensive workload. Limit your intensity to reasonable bounds, such as **10 to 50 or 100 requests per second**, starting small and gradually increasing.
- **Duration and Repetition:** Run the experiment for **10 to 30 minutes** to collect a sufficient number of data points. Because network and system variances heavily impact single runs, you must **repeat each experiment 3 to 10 times** to calculate an accurate average.

**6. Setup the Experimental Environment** To prevent AWS from interpreting your load test as a malicious attack and banning your account, you must **run your workload generation tool from a Virtual Machine located inside the AWS perimeter**. While this means you will not measure true internet latency, it ensures the test will complete successfully. Make sure AWS CloudWatch is fully configured to capture your selected metrics.

**7. Run and Analyze** Once your repeated experiments are finished, **download the raw data points** (using the console or a CLI script). **Do not simply take screenshots of the CloudWatch dashboard**, as they only reflect a single run, are difficult to read, and expire over time. Instead, calculate the average values across all your experimental runs and manually plot them (e.g., showing the correlation between `ConcurrentExecutions` or response time and the intensity of the workload over time). Finally, to test the system's high availability, you can manually generate random failures by stopping running services or containers to see how the system reacts.