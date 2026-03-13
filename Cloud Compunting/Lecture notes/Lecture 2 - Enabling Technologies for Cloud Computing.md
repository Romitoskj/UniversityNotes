*March 10 2026*
### 1. Distributed Systems vs. Distributed Computing

- **Distributed System:** A collection of independent, networked computers that communicate and coordinate via message passing, but appear to the end-user as a single, coherent system. It focuses on the physical components and hiding the underlying network complexity.

- **Distributed Computing:** A computational model where a single task is broken down into smaller units executed concurrently across different computing elements (e.g., different cores, processors, or networked machines).

- **Cloud Computing as a Distributed System:** Cloud computing is a specialized form of distributed computing designed to provision scalable and measured resources. The layers of a traditional distributed system map directly to cloud models: the physical hardware and OS form the Cloud Infrastructure, the Distributed Computing Middleware maps to Platform as a Service (PaaS), and the Distributed Applications map to Software as a Service (SaaS).![](Images/Pasted%20image%2020260313173544.png)![532](Images/Pasted%20image%2020260313173602.png)

### 2. Architectural Styles

An architectural style serves as a blueprint or pattern for organizing a system. It defines the vocabulary of the components, their roles, how they are distributed across machines, the communication mechanisms (connectors) they use, and constraints on how they can be combined. Key styles include:

- **Call and Return:** Components call sub-programs or lower layers, which eventually return results (e.g., the TCP/IP stack or standard C programming libraries).

- **Pipe and Filter (Data Flow):** A chain of processing stages where the output of one becomes the input of the next. A classic example is the **MapReduce** paradigm (e.g., word count applications).

- **Client/Server:** Features presentation, application logic, and data storage tiers.
    - _Thin-client:_ Client handles only presentation; server does logic and storage.
    - _Fat-client:_ Client handles presentation and logic; server does storage.
    - _Multi-tiered (N-tier):_ Distributes tiers across multiple servers to isolate performance bottlenecks and allow independent scaling.

- **Peer-to-Peer (P2P):** A symmetric, highly decentralized architecture where all nodes (peers) act as both clients and servers. This scales very well and is commonly used for distributed storage (e.g., Cassandra, Amazon Dynamo).

- **Independent Components:** Components have independent lifecycles and interact either through **communicating processes** (using RPC, REST, Web Services) or **event-based systems** (publish/subscribe via a message broker).

### 3. Service Oriented Architecture (SOA)

SOA is an architectural style based on "service orientation," where applications are built by aggregating services rather than writing monolithic functions. A service is a logical representation of a repeatable business activity (e.g., checking a credit card). **Four Characteristics of a Service:**

1. **Explicit Boundaries:** Interfaces are kept minimal to foster reuse.
2. **Autonomous:** Services handle failures and operate independently, acting as a "black box" to the consumer.
3. **Shared Schema and Contracts:** They share strict contracts defining message structures.
4. **Policy-driven:** Semantic compatibility is determined by defined policies.

**SOA Roles:**

- **Service Provider:** Maintains and publishes the service.
- **Service Registry:** A central directory storing service metadata, contracts, and binding information.
- **Service Consumer:** Queries the registry to discover services and bind to the provider.

**Coordination in SOA:**

- **Orchestration:** A centralized approach where a single executable business process (the orchestrator) invokes and combines services sequentially (e.g., a travel planner booking flights, then hotels, then cars).
- **Choreography:** A decentralized approach where independent services exchange messages according to rules without a central controller (e.g., an e-commerce site communicating with a credit card authentication service).

### 4. Microservices Architecture**

Microservices emerged as an evolution of SOA to solve the problem of SOA services becoming too large and monolithic. It structures applications as collections of very small, single-purpose services. **Key Features:**

- **Highly maintainable and testable:** Because the codebases are small.
- **Loosely coupled and independently deployable:** Managed by small, autonomous teams with their own continuous integration (CI) pipelines and source code repositories.
- **The Scale Cube:** Microservices enable scaling along three axes.
    - _X-axis:_ Horizontal duplication (cloning instances behind a load balancer).
    - _Z-axis:_ Data partitioning (sharding data based on user ID).
    - _Y-axis:_ Functional decomposition (scaling specific hotspots, like replicating only the "Order Service" during high traffic rather than the whole application).

### 5. SOA vs. Microservices: Key Differences

While both use services, their implementations differ significantly:

- **Taxonomy & Middleware:** SOA relies heavily on **messaging middleware (Enterprise Service Bus)** to handle routing, transformation, and protocol conversion between layers (Business, Enterprise, Application). Microservices eliminate middleware, placing an **API layer/Gateway** in front of direct, point-to-point service calls.
- **Granularity:** SOA services can range from small to massive enterprise subsystems. Microservices are strictly single-purpose and fine-grained.
- **Transactions:** SOA can easily guarantee ACID transactions because services often share a central data storage. Microservices struggle with distributed transactions because each service manages its own isolated database.
- **Component Sharing:** SOA is a **"share-as-much-as-possible"** architecture to prevent duplicate code, leading to tight coupling. Microservices is a **"share-as-little-as-possible"** architecture using "bounded contexts," meaning a service and its data are completely self-contained, even if it violates the DRY (Don't Repeat Yourself) principle.
- **Team Coordination:** SOA requires high coordination across teams (business users, developers, infrastructure). Microservices require almost zero coordination because small teams own everything for their specific service.
- **Heterogeneous Interoperability:** SOA handles high heterogeneity, easily translating different protocols and languages via its middleware. Microservices prefer standardized, simple remote-access protocols like REST and JSON.
- **Application Scope:** SOA is best for large, complex, enterprise-wide legacy systems. Microservices are ideal for well-partitioned, highly scalable web-based systems.