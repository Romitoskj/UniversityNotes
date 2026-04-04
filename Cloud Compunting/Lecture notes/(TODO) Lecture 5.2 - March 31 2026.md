# Orchestration & Kubernetes

Orchestration refers to automating a series of individual tasks to work together as a process or workflow.

## 1. Container Orchestration Concept

While service orchestration coordinates application functionalities, container orchestration coordinates container lifecycle.

- **Purpose:** It allows providers to select, deploy, monitor, and dynamically control multi-container packaged applications.
- **Lifecycle Phases:** Orchestration is specifically concerned with automating the deployment, execution (run time), and maintenance in run time phases of a container's lifecycle.![](Images/Pasted%20image%2020260403212051.png)
- **Core Features:** An orchestrator typically handles resource limit control, complex scheduling policies, load balancing, health checks, fault tolerance, and auto-scaling.

## 2. Kubernetes (K8s) Overview and Features

Kubernetes is a widely used orchestration platform for managing containerized workloads and services. Its automated features include:

- **Service Discovery & Load Balancing:** Automatically locate and identify services needed for orchestration and distributing workload traffic efficiently among the platform's services.
- **Storage Orchestration:** Automatically mounting and unmounting storage volumes to containers, or moving volumes between physical nodes.
- **Automated Rollouts/Rollbacks:** Continuously working to keep deployed containers in a specific "desired state".
- **Automatic Bin Packing:** Fitting containers onto cluster nodes based on their specific CPU and memory resource demands.
- **Self-Healing:** Automatically restarting, replacing, or killing containers that are no longer healthy.
- **Secret and Configuration Management:** Handling and deploy sensitive information such as keys along with configuration files.

## 3. Three Core Kubernetes Concepts

### 1. Objects:
Persistent entities used to represent the state of the cluster (e.g., pods, deployments, nodes). They can describe:
- What containerized applications are running
- The resources avilable to those applications
- The policies around how those applications behave (restart policies, upgrade, fault-tolerance)

They act as a "record of intent": once created, K8s constantly works to ensure the object exists in its target state. Objects rely on two fields:

- **spec:** Defines the _desired state_ (e.g., "I want 3 replicas of this application").
- **status:** Describes the _current state_ (e.g., "Currently, 2 instances are running"). K8s monitors the difference between the two and takes action to match the status to the spec.

### 2. Pods:
The smallest deployable unit in Kubernetes. A Pod is a "logical host" representing a boundary around one or more tightly coupled containers.

- Containers within a Pod share storage, network resources, specification for how to run the containers
- They are strictly co-located and co-scheduled, meaning all containers in a Pod are always executed together on the exact same node.
- Run in a shared context: a set of Linux namespaces and cgroups (individual application may have further isolations applied), shared filesystem volumes.

### 3. Nodes:
The physical or virtual machines that execute the Pods managed by the control plane.

- Nodes join the cluster either via `kubelet` self-registration or by a human manually adding a Node object. Both of them by adding a node object via the API-server.
- Once a node object is created the control plane checks if it is valid (all the services are running) and if so it could run a pod.
- **Node Status:** Defined by its 
	- Address (hostname and IP)
	- Capacity (available CPU/memory and maximum number of pods that can be scheduled onto the node)
	- Allocable resources (usable portion of those resources that can be used by a pod)
	- Info (general information such as kernel version, kubernetes version, containers runtime details, OS)
	- **Conditions**. Conditions indicate the node's health, such as _Ready_ (to accept pods), _DiskPressure_, _MemoryPressure_, or _NetworkUnavailable_. If a condition fails (e.g., the node is unresponsive), the node controller will evict and reschedule the Pods running on it.

## 4. K8s Cluster Architecture: Control Plane vs. Nodes

A Kubernetes architecture is divided into two main sections: the **Control Plane** (the managing entity) and a **Cluster of Nodes** (where the workloads are actually executed).

![697](Images/Pasted%20image%2020260403212851.png)
### A. Control Plane Components

- **kube-apiserver:** The front-end entry point that allows the control plane to communicate with the cluster of nodes. It receives state information and pushes actions, scaling horizontally to handle requests.
- **etcd:** A highly available key-value store used to hold all the configuration data for the cluster.
- **kube-scheduler:** Responsible for assigning newly created Pods to the most appropriate physical or virtual Node.
- **kube-controller-manager:** Runs continuous control loops to maintain the cluster's desired state:
	- Node controller for noticing and responding when nodes go down,
	- Job controller that watches for job objects that represent one-off tasks then creates pods to run those tasks,
	- Endpoint slice controller for populating EndpointSlice objects that provide link between services and pods,
	- Service account controller that creates default service accounts for new namespaces.
- **cloud-controller-manager:** Allows the local cluster to interact with external cloud provider APIs, such as extending the cluster using remote virtual machines.

### B. Node Components

- **kubelet:** The primary agent running on each node that self-registers the node to the API server and ensures that the containers described in a Pod specification are running and healthy.
- **kube-proxy:** a network proxy that maintains the network rules that allow for proper communication and connectivity to the cluster's Pods inside and outside from the cluster.
- **Container runtime:** The underlying software (like Docker) required to actually run the containers.

## 5. Controllers and Autoscaling

Controllers are continuous autonomic loops that watch a specific resource (like nodes or jobs). They compare the `status` against the `spec` and send messages to the `kube-apiserver` to enact changes.

- **Example (Job Controller):** Watches for new Job objects, requests the API server to create the necessary Pods, monitors them until the task is successfully finished, and then marks the job as completed.
- **Autoscaling:** K8s supports both **Cluster Autoscaling** (adding/removing physical nodes by communicating with external cloud resources) and **Horizontal Pod Autoscaling** (dynamically calculating the ratio of current utilization to target utilization to add or remove Pod replicas).

## 6. The Scheduling Process

The `kube-scheduler` assigns unscheduled Pods to Nodes through a two-step process:

1. **Filtering:** The scheduler finds all "feasible nodes" that satisfy the Pod's resource requirements. To save time in massive clusters, the scheduler often samples a subset of nodes rather than checking every single machine.
2. **Scoring:** The scheduler runs a scoring function (configured via Scheduling Profiles) to rank the feasible nodes. For instance, it might select the node with the lowest CPU load, or alternatively, pack workloads tightly onto nodes with the highest number of running pods to conserve resources.

### Affinity and Anti-Affinity Rules:

The scheduler's decisions are heavily influenced by affinity rules, which rely on node labels:

- **Node Affinity:** Rules preferring certain nodes. Used for optimizing network latency, routing HPC workloads to specialized hardware, or ensuring compliance with data sovereignty laws (e.g., keeping processing in a specific geographic area).
- **Node Anti-Affinity:** Rules preventing Pods from being scheduled on certain nodes. Highly useful for keeping instances separated for high availability (preventing a single point of failure) or separating workloads that might cause performance interference.