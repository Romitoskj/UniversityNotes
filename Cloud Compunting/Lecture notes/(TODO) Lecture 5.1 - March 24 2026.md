# Automation

## 1. IT Automation vs. Autonomic Computing

While both disciplines address the management of IT systems, there is a fundamental difference in their execution cycles and the role of the human administrator:

- **Autonomic Computing:** The goal is to completely *remove* humans from the loop. The human simply provides high-level objectives, and the system continuously manages itself across its entire lifecycle using the MAPE (Monitor, Analyze, Plan, Execute) loop.
- **IT Automation:** The goal is to *reduce* human interaction and manual processes for repetitive tasks, but it does not replace the human. There is no continuous autonomous cycle; instead, a human defines a well-defined plan (a set of instructions) and "clicks a button" to trigger the execution. The benefits are:
	- Improved efficiency and productivity by reducing time and effort required for manual tasks.
	- Reduced human error.
	- Saved time and costs.
- **Applying them to Data Center Challenges:**
    - _Server Consolidation:_ This requires Autonomic Computing because it relies on continuously monitoring the data center to optimize resources and minimize energy consumption over time.
    - _Bug Fixes and Updates:_ These are typically handled via Automation, as they require pushing new code or configurations when a patch is released.

## 2. Ansible

### A. The Agentless "Push" Model

Ansible is presented as a practical example of an IT automation tool used to manage operations and infrastructure.

- **Agentless Architecture:** Many legacy management tools require installing a "client agent" on every target node. Ansible is agentless, which is highly advantageous because running agents adds overhead to the target machine, creates another point of potential failure, and introduces an extra piece of software that could harbor security vulnerabilities.
- **The Push Model:** Because there are no agents, the Ansible control server stores the code and "pushes" instructions directly to the target nodes. It leverages native, standard operating system remote management tools—specifically **SSH** for Linux/UNIX and **WinRM** for Windows. This ensures stability and requires no extra setup on the target node.
- **Exception for Network Appliances:** Network devices (like small routers or access points) generally cannot execute pushed code. For these devices, the automation tool runs the script locally on the control server and interacts with the appliance via its API endpoints.

![697](../../Pasted%20image%2020260329120340.png)

### B. Playbooks, Modules, and the "Desired State"

Ansible automates execution using **Playbooks**, which are written in YAML syntax and act as repeatable, reusable scripts for configuration management and deployment.

- **Inventory, Plays, and Tasks:** A playbook acts on a set of target nodes known as the **inventory**. A playbook is partitioned into "plays," and each play contains multiple "tasks".
- **Modules:** Each task calls a specific module—a small piece of code designed for a specific action. Examples of core modules include:
    - `ansible.builtin.yum`: To ensure software (like httpd or PostgreSQL) is at the latest version.
    - `ansible.builtin.template`: To copy a configuration file from a source to a destination.
    - `ansible.builtin.service`: To bring a service into a specific state (e.g., starting it).
- **Desired State and Idempotence:** This is a crucial concept in Ansible. Instead of just blindly running commands, tasks declare a "desired state". Before executing an action, the system checks if the node is already in that state (e.g., checking if the Apache server is already running). If the desired state is already met, the action is skipped. This makes playbooks **idempotent**, meaning if a script fails halfway through, you can safely rerun the entire playbook without causing duplicate errors or side effects.

### C. Practical Example: Rolling Updates

Ansible can be used to orchestrate complex operations, such as a rolling update on a traditional three-tier web application (Web Servers, App Servers, Database Servers). The process involves automating a sequence of steps for a subset of servers at a time:

1. Checking the target nodes in the inventory.
2. Enforcing the base OS configuration.
3. Signaling the monitoring system of an upcoming outage.
4. Telling the load balancer to stop routing traffic to the targeted servers.
5. Stopping the application, deploying the updated code, and starting the server.
6. Running tests, adding the server back to the load balancer pool, and resuming monitoring alerts. This is repeated across all application servers, and then moved down to the database tiers.

## 3. The Link Between Automation and Autonomic Computing

While distinct, Automation and Autonomic Computing are directly connected: automation tools essentially serve as the **"Execute" phase** of the autonomic MAPE loop. Once an autonomic manager has monitored, analyzed, and planned a change, it can use an automation script (like an Ansible playbook) to execute it.

- **Self-configuration:** Deploying a newly generated configuration playbook.
- **Self-optimization:** Executing a playbook to provision and add new servers to a cluster.
- **Self-healing:** Triggering a playbook to detach and restart a degrading application instance.
- **Self-protection:** Automatically deploying an update playbook to patch a discovered vulnerability.
