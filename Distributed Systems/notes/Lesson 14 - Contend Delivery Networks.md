
### I. The Domain Name System (DNS)

The DNS is a distributed system designed to translate memorable domain name strings into numerical IP addresses.

#### A. DNS Structure and Process

- **Purpose:** IP addresses are difficult to remember (e.g., four bytes as a number), so DNS provides human-readable names.
- **Hierarchy:** The structure of domains is hierarchical (e.g., top-level domains like `.it` having numerous subdomains).
- **Resolution Process:** A local DNS resolver connects to the DNS system.
    1. The resolver contacts a **Root Server** to find the IP of the top-level domain server.
    2. This is an iterative process: the resolver is directed down the hierarchy until the final IP address is obtained.
- **Root Servers:** Logically, there are **13 root servers** in the world.

![](../assets/Pasted%20image%2020251212120140.png)

#### B. Consistency, Availability, and Partition Tolerance (CAP)

- **Caching:** All retrieved IP information is **cached** by the DNS resolver to avoid repeating the entire resolution process.
- **Consistency Challenge:** Caching breaks consistency. If a domain's IP address is changed, the cached information becomes invalid.
- **Mitigation:** Consistency problems are reduced by propagating changes to everybody (though this takes time) or by giving cached elements a Time-To-Live (TTL) after which they are refreshed.
- **Trade-off:** The caching system gives the system more **partition tolerance** and **availability** at the expense of consistency.

#### C. Reliability through Anycast

- **Replication:** The root servers are extensively replicated to handle the load of the entire internet.
- **Scale:** Each of the 13 logical root servers has **hundreds of physical replicas**.
- **Anycast Mechanism:** All replicas share the **same IP address**. This shared IP is announced by a large number of **Autonomous Systems (AS)** using the **BGP routing protocol**.
- **Benefit:** When a user queries a root server, they are automatically routed to the **closest copy**. This makes the DNS system **extremely reliable** and resistant to Denial of Service (DoS) attacks.

> [!Info]-  The Border Gateway Protocol
> BGP (Border Gateway Protocol) is the routing protocol used for communication **among Autonomous Systems (AS)** on the internet.
> 
The internet is fundamentally composed of these Autonomous Systems, which, as their name suggests, operate autonomously. While routing within a single AS can use internal rules, communication _between_ different Autonomous Systems follows the BGP protocol.
>
BGP routing works like that:
>
> - **Announcing IPs:** Under the BGP protocol, an Autonomous System can **announce** which IP addresses are located within its system. Essentially, it declares, "If you are looking for these IPs, they are within my system. Send the packet to me to reach them".
> - **Propagation and Path Building:** These announcements are propagated across other Autonomous Systems. Every time an announcement is propagated, the receiving Autonomous System adds information indicating the path that must be followed to reach those IPs.
> - **Path Construction:** If Autonomous System A announces IPs, saying traffic must go through A, and then this announcement is sent to Autonomous System B, B will propagate the information saying, "To reach those IPs, you have to go through B, and then A". This process **builds the paths** necessary to route traffic across the internet.
>
This protocol is vital for reliability, as demonstrated by the use of **Anycast** for DNS root servers: the IP address of a root server is announced by all the many Autonomous Systems that host a replica of that server, routing users to the closest copy.

---

### II. Content Delivery Networks (CDNs) – The Akamai Idea

CDNs were developed to make access to internet content much faster and more reliable. A CDN is a network of servers that act as a cache of some content, for example static images from a news website, movies from a streaming platform ecc.

#### A. The Problem and the Solution

- **Problem:** While HTML page text is light, pages are full of media content (pictures, videos) which require multiple HTTP connections to retrieve and take significant time to download.
- **Akamai's Solution:** Build a globally distributed network of servers (CDNs) that hold **copies of this media content**.

#### B. CDN Mechanism

1. **"Akamization":** HTML pages are modified ("Akamization") by changing the links to media files to include the CDN's domain (e.g., `cnn.com.akamai.com/pictures/34.jpg`).
2. **DNS Redirection:** When a user's resolver queries the Akamai name, the Akamai DNS system intercepts the request.
3. **Closest Server Selection:** Akamai's DNS analyzes the location of the request and returns the IP address of the **closest CDN server (Point of Presence)** holding the content. The next HTTP connection is then made to this closer server, not the origin server (e.g., CNN server in the US).
4. **Load Balancing:** The CDN DNS system is also used to perform load balancing, routing users to a less-loaded server even if it is slightly farther away.

![](../assets/Pasted%20image%2020251212115807.png)

#### C. CDN Trade-offs (Prioritizing Availability)

- CDNs are distributed systems that prioritize **availability** and **partition tolerance**.
- **Consistency Trade-off:** For applications like news distribution or social media (e.g., Facebook), **consistency is traded for availability**. It is acceptable to retrieve a picture or post that is a **few minutes old** if it ensures the user can access the system.
- **High Consistency Systems:** Conversely, applications like banking systems or plane ticket sales require high **consistency** (e.g., ensuring no one sells more tickets than available), even if it means reduced availability.