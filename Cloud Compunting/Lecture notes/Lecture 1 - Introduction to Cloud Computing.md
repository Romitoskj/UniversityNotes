*March 03 2026*

### 1. History & The Utility Computing Model

- **Early Visions:** The concept of cloud computing dates back to 1961 when John McCarthy suggested that computing might someday be organized as a "public utility". In 1969, ARPANET pioneer Leonard Kleinrock echoed this, predicting the spread of "computer utilities" functioning similarly to electric or telephone utilities.
- **Utility Computing:** This model means consumers pay only when they access computing services, require no upfront infrastructure investments, and can access services regardless of where they are hosted.
- **Modern Era:** This evolution from mainframes and grid computing culminated in 2005/2006 when Amazon (AWS) launched Elastic Compute Cloud (EC2) and Simple Storage Service (S3), treating server time and storage as a utility.

### 2. NIST SP 800-145: Definition of Cloud Computing

> [!important] Cloud Computing definition
> A model enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources that can be rapidly provisioned with minimal management effort or service provider interaction.

- _Note on Ubiquity:_ Servers and data location remains crucial for legal compliance (e.g., GDPR), strategic control, and network performance (latency), driving the shift toward edge computing.

The NIST definition is structured around **three main components**:

#### A. Five Essential Characteristics

To be considered a true cloud service, all five of these must be present:

1. **On-demand self-service:** Consumers can provision computing capabilities automatically without requiring human interaction from the service provider, usually via web interfaces, dedicated shells, or APIs.
2. **Broad network access:** Services are accessed over the network through standard mechanisms (e.g., mobile phones, tablets, laptops). Performance and security issues are typically concentrated in the network and at the edge.
3. **Resource pooling (Multi-tenancy):** Provider resources are pooled to serve multiple consumers simultaneously. Physical and virtual resources are dynamically assigned.
    - _Multi-tenancy properties_ ensure that while users share application logic or hardware, they experience complete usage isolation, data security, transparent application upgrades, and independent recovery from failures.
4. **Rapid Elasticity:** The system can automatically scale resources up and down to match workload demands at any given time (e.g., an e-commerce site handling a Black Friday spike without crashing).
    - _Elasticity vs. Scalability:_ Scalability is the ability to add resources (horizontally via new VMs, or vertically via more CPUs), but elasticity requires the system to do this automatically and rapidly based on monitoring rules (e.g., autoscaling if CPU > 70%).
5. **Measured Service:** Cloud systems automatically control and optimize resource use through metering capabilities. This is essential for billing (pay-per-use), SLA (Service Level Agreement) management, cost control, and improving system resiliency (e.g., automatically replacing a failed container or VM).

#### B. Three Service Models

These models dictate the level of consumer control and the separation of security responsibilities:

- **IaaS (Infrastructure as a Service):** Consumer controls the OS, storage, and deployed applications. The provider secures the physical data center, network infrastructure, and virtualization isolation.
- **PaaS (Platform as a Service):** Consumer controls deployed applications and data. The provider manages the underlying infrastructure and OS.
- **SaaS (Software as a Service):** Consumer only controls user-specific application settings. The provider manages the entire underlying infrastructure and application logic.
- _Security Takeaway:_ Security is a shared responsibility. The cloud provider secures the infrastructure ("security _of_ the cloud"), while the customer secures their OS, apps, and data ("security _in_ the cloud").

#### C. Four Deployment Models

- **Public Cloud:** Open to general public use (e.g., anyone can enter a credit card and use it).
- **Private Cloud:** Dedicated entirely to a single organization. It can be physically on-premise or provisioned as a Virtual Private Cloud (VPC) providing logical isolation within a public provider's data center.
- **Hybrid Cloud:** A combination of distinct public and private clouds bounded together (e.g., keeping critical data on a private cloud while running general web services on a public cloud).
- **Community Cloud:** (Briefly mentioned as the fourth model in the NIST definition).

### 3. NIST SP 500-292: Cloud Computing Reference Architecture

This architecture defines the interactions between the five major actors in cloud computing:

1. **Cloud Consumer:** The entity using the cloud services.
2. **Cloud Provider (CSP):** The entity making the service available. They are responsible for _Service Orchestration_ (managing and composing VMs, containers, and microservices) and _Cloud Service Management_ (business support, provisioning, metering, and interoperability).
3. **Cloud Auditor:** An independent third party that conducts assessments of the provider's performance, security, and SLA compliance.
4. **Cloud Broker:** An intermediary that negotiates and manages services between consumers and providers. They provide:
    - _Service Intermediation:_ Adding value (e.g., advanced cost monitoring).
    - _Service Aggregation:_ Combining services from multiple providers into a new service.
    - _Service Arbitrage:_ Dynamically swapping providers to optimize cost and performance.
5. **Cloud Carrier:** The intermediary (like an ISP) providing the physical connectivity and network transport.