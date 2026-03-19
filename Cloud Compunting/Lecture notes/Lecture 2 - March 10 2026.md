# Enabling Technologies for Cloud Computing

## 1. Distributed Systems vs. Distributed Computing

- **Distributed System:** A collection of independent, networked computers that communicate and coordinate via message passing, but appear to the end-user as a single, coherent system. It focuses on the physical components and hiding the underlying network complexity.

- **Distributed Computing:** A computational model where a single task is broken down into smaller units executed concurrently across different computing elements (e.g., different cores, processors, or networked machines).

- **Cloud Computing as a Distributed System:** Cloud computing is a specialized form of distributed computing designed to provision scalable and measured resources. The layers of a traditional distributed system map directly to cloud models: the physical hardware and OS form the Cloud Infrastructure, the Distributed Computing Middleware maps to Platform as a Service (PaaS), and the Distributed Applications map to Software as a Service (SaaS).
   
![Lecture 2 - enabling technologies - part 1](../Slides/Lecture%202%20-%20enabling%20technologies%20-%20part%201.pdf#page=6)

## 2. Architectural Styles

An architectural style serves as a blueprint or pattern for organizing a system. It defines the vocabulary of the components, their roles, how they are distributed across machines, the communication mechanisms (connectors) they use, and constraints on how they can be combined. Key styles include:

- **Call and Return:** Components call sub-programs or lower layers, which eventually return results (e.g., the TCP/IP stack or standard C programming libraries).

- **Pipe and Filter (Data Flow):** A chain of processing stages where the output of one becomes the input of the next. A classic example is the **MapReduce** paradigm (e.g., word count applications).![](Images/Pasted%20image%2020260313174122.png)

- **Client/Server:** Features presentation, application logic, and data storage tiers.
    - _Thin-client:_ Client handles only presentation; server does logic and storage.
    - _Fat-client:_ Client handles presentation and logic; server does storage.
    - _Multi-tiered (N-tier):_ Distributes tiers across multiple servers to isolate performance bottlenecks and allow independent scaling.![](Images/Pasted%20image%2020260313174247.png)

- **Peer-to-Peer (P2P):** A symmetric, highly decentralized architecture where all nodes (peers) act as both clients and servers. This scales very well and is commonly used for distributed storage (e.g., Cassandra, Amazon Dynamo).

- **Independent Components:** Components have independent lifecycles and interact either through **communicating processes** (using RPC, REST, Web Services) or **event-based systems** (publish/subscribe via a message broker).

## 3. Service Oriented Architecture (SOA)

SOA is an architectural style based on "service orientation," where applications are built by aggregating services rather than writing monolithic functions. A service is a logical representation of a repeatable business activity that has a specified outcome(e.g., checking a credit card). 

### Four Characteristics of a Service:
1. **Explicit Boundaries:** Interfaces are kept minimal to foster reuse and simplify interaction.
2. **Autonomous:** Services handle failures and operate independently, acting as a "black box" to the consumer.
3. **Shared Schema and Contracts:** They share strict contracts defining message structures.
4. **Policy-driven:** Semantic compatibility is determined by defined policies.

### SOA Roles:
- **Service Provider:** Maintains and publishes the service.
- **Service Registry:** A central directory storing service metadata, contracts, and binding information.
- **Service Consumer:** Queries the registry to discover services and develop the client componente to bind and use the service.

### Coordination in SOA:
- **Orchestration:** A centralized approach where a single executable business process (the orchestrator) invokes and combines services sequentially (e.g., a travel planner booking flights, then hotels, then cars).
- **Choreography:** A decentralized approach where independent services exchange messages according to rules without a central controller (e.g., an e-commerce site communicating with a credit card authentication service).![](Images/Pasted%20image%2020260313175617.png)

## 4. Microservices Architecture

Microservices emerged as an evolution of SOA to solve the problem of SOA services becoming too large and monolithic. It is an architectural style that structures applications as collections of very small, single-purpose services. 

### Key Features:
- **Highly maintainable and testable:** Because the codebases are small.
- **Loosely coupled and independently deployable:** Managed by small, autonomous teams with their own continuous integration (CI) pipelines and source code repositories.
- **The Scale Cube:** Microservices enable scaling along three axes.
    - _X-axis:_ Horizontal duplication (cloning instances behind a load balancer).![](Images/Pasted%20image%2020260313235658.png)
    - _Z-axis:_ Data partitioning (sharding data based on user ID).![](Images/Pasted%20image%2020260313235720.png)
    - _Y-axis:_ Functional decomposition (scaling specific hotspots, like replicating only the "Order Service" during high traffic rather than the whole application).![](Images/Pasted%20image%2020260313235740.png)
### Monolithic Architecture vs. Microservices:
- **Monolithic Architecture:**
    - **Initial Simplicity:** They are initially simple to develop, and straightforward to test and deploy because all functionalities are bundled into a single unified codebase.
    - **Shared Data:** The entire application typically interacts with a single shared database.
    - **Scaling Limitations:** Scaling is generally restricted to horizontal duplication (replicating the entire application across servers). This can be bottlenecked by the shared database and is highly inefficient if only one specific function needs more resources.
    - **Complexity Issues:** As the application grows in complexity or as technologies evolve, it becomes significantly more difficult to maintain, test, update, and scale the system.
- **Microservices Architecture:**
    - **Decoupled Services:** Applications are structured as a collection of highly maintainable, loosely coupled, and independently deployable services that are organized around specific business capabilities (e.g., a dedicated service just for orders or delivery).
    - **Private Data:** Unlike a monolith, each microservice manages its own private data and does not share its database with other services.
    - **Functional Scaling:** Organizations can utilize Y-axis scaling (functional decomposition) to replicate and scale only the specific services experiencing high load, rather than duplicating the entire application.
    - **Production:** Small, autonomous teams manage their own services using independent source code repositories and continuous integration (CI) deployment pipelines, removing the massive coordination overhead required by monolithic systems.

## 5. SOA vs. Microservices: Key Differences

While both use services, their implementations differ significantly:

- **Taxonomy & Middleware:** SOA relies heavily on **messaging middleware (integration Bus)** to coordinate service calla, handle routing, message transformation, and protocol conversion between layers (Business, Enterprise, Application). Microservices eliminate middleware, placing an **API layer/Gateway** in front of direct, point-to-point service calls, allowing to ignore the actual location of the service endpoints and also to change the granularity level of a service withour impacting service consumer.
- **Granularity:** SOA services can range from small to massive enterprise subsystems. Microservices are strictly single-purpose and fine-grained.
- **Transactions:** SOA can easily guarantee ACID transactions because services often share a central data storage. Microservices struggle with distributed transactions because each service manages its own isolated database.
- **Component Sharing:** SOA is a **"share-as-much-as-possible"** architecture to prevent duplicate code, leading to tight coupling. Microservices is a **"share-as-little-as-possible"** architecture using "bounded contexts," meaning a service and its data are completely self-contained, even if it violates the DRY (Don't Repeat Yourself) principle.
- **Service Choreography and Orchestration**: Microservices favor choreography because architecture topology lacks a centralized middleware component, while SOA relies on both service orchestration and choreography.
- **Team Coordination:** SOA requires high coordination across teams (business users, developers, infrastructure). Microservices require almost zero coordination because small teams own everything for their specific service.
- **Heterogeneous Interoperability:** SOA handles high heterogeneity, easily translating different protocols and languages via its middleware. Microservices prefer standardized, simple remote-access protocols like REST and JSON.
- **Application Scope:** SOA is best for large, complex, enterprise-wide legacy systems that require integration with many heterogeneous applications and services. Microservices are ideal for well-partitioned, highly scalable web-based systems.

|**Feature / Category**|**SOA (Service-Oriented Architecture)**|**Microservices**|
|---|---|---|
|**Taxonomy & Middleware**|Relies heavily on messaging middleware (Integration Bus) for routing, transformation, and protocol conversion.|Eliminates middleware; uses an API Gateway for direct point-to-point calls, hiding endpoint locations.|
|**Granularity**|Ranges from small services to massive enterprise subsystems.|Strictly single-purpose and fine-grained.|
|**Transactions**|Easily guarantees ACID transactions due to shared central data storage.|Struggles with distributed transactions; each service manages an isolated database.|
|**Component Sharing**|"Share-as-much-as-possible" to prevent duplicate code, leading to tight coupling.|"Share-as-little-as-possible" (bounded contexts); self-contained even if it violates the DRY principle.|
|**Choreography & Orchestration**|Relies on both service orchestration and choreography.|Favors choreography due to the lack of centralized middleware.|
|**Team Coordination**|Requires high coordination across varied teams (business, developers, infrastructure).|Requires minimal coordination; small, autonomous teams own everything for their service.|
|**Heterogeneous Interoperability**|Handles high heterogeneity by translating protocols and languages via middleware.|Prefers standardized, simple remote-access protocols like REST and JSON.|
|**Application Scope**|Best for large, complex, enterprise-wide legacy systems needing broad integration.|Ideal for well-partitioned, highly scalable web-based systems.|
![](Images/Pasted%20image%2020260314003155.png)