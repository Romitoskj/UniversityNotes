## 1. Traffic Regulation: Philosophy & Architecture

While a router’s primary job is to ask "where should this packet go?" by checking routing tables against the destination IP, a firewall adds a crucial second question: **"Is this packet allowed to go there?"**. To enforce this, network administrators design architectures based on key security principles:

- **Choke Points:** You must force all traffic through a single, heavily monitored access point to ensure no traffic bypasses the firewall via "backdoors".

- **Access Control List:** List the rights for accessing the networks, distinguishing between incoming and outgoing traffic per interface/port. It is *stateless*, every packet treated independently.

- **Bastion Host:** Hardened computer used to deal with all traffic coming to a protected network from outside. Suitable for use as Application Proxy Gateways.

- **Defense in Depth:** Security should rely on overlapping systems. Instead of a single firewall, modern networks use multiple layers of protection (e.g., an external screening router, a perimeter network, an internal firewall, and individual host-based firewalls). This add redundancy to the defensive measures, aim to remove single point of failure and find the right balance between complexity and multiplicity of defense measure.

>[!info]- Hardening
>Hardening is the task of reducing or removing vulnerabilities in a computer system:
>- Shutting down unused or dangerous services
>- Strengthening access controls on vital files
>- Removing unnecessary accounts and permissions

### 1.2 Firewall placement

- **Host Based Packet Filter:** disciplines packets in a single host, specifing what can be received or sent

- **Screening Router (ACL-based)**
   ![](images/Pasted%20image%2020260622142002.png)
   
- **Dual-homed Host**
  ![](images/Pasted%20image%2020260622142106.png)

### 1.3 The DMZ (Demilitarized Zone)

A secure, neutral sub-network placed between the public internet and the private internal network. The DMZ hosts systems that need to be publicly accessible (like Web or Mail servers) on hardened machines called **Bastion Hosts**. If a Bastion Host in the DMZ is compromised, the attacker still faces another internal firewall before they can reach the private network.

The DMZ can be arranged in multiple ways: 

![](images/Pasted%20image%2020260622143448.png)

![](images/Pasted%20image%2020260622143535.png)

Can be used also to segment the network:

![](images/Pasted%20image%2020260622143645.png)

And to apply security in depth:

![](images/Pasted%20image%2020260622143840.png)

## 2. Stateless Packet Filtering (The "Basic" Firewall)

Stateless packet filters (often implemented as Access Control Lists, or ACLs, on screening routers) evaluate every single packet entirely independently, with no memory of what happened before. They filter purely based on the Network and Transport layer headers: Source/Destination IP, Source/Destination Port, and TCP Flags.

### 2.1 Three Step Process to Enforce a Policy:

1. Know your policy
2. Translate it in formal language
3. Rewrite it in terms of the firewall syntax

- Rules are evaluated from top to bottom
- The first matching rule is applied
- One implicit rule is assumed if no matching (block/allow everything)

### 2.2 Vulnerabilities of Stateless Filters

Because stateless filters lack context, their rules are often too rigid or dangerously broad:

- **The Direction/Port Problem:** If you write a rule to allow internal hosts to receive responses from external servers, a clever attacker can spoof their source port to match an allowed service (e.g., port 25) and freely push traffic into your internal network. To fix this, rules must strictly check for the `ACK` flag (indicating an established connection), but even this is flawed.
  
  ![](images/Pasted%20image%2020260622173608.png)

| **Action** | **SRC**     | **Port** | **Dest** | **Port** | **Flags** | Comment                  |
| ---------- | ----------- | -------- | -------- | -------- | --------- | ------------------------ |
| allow      | {our hosts} | *        | *        | 25       | *         | Connection to their SMTP |
| allow      | *           | 25       | *        | *        | ACK       | Their replies            |
| block      | *           | *        | *        | *        |           | Default                  |

- **Filter Rules for Network Firewalls (Complex Topologies):** When scaling up to a more complex network topology global stateless rules become inadequate. To properly enforce security, administrators must adapt their filtering approach:
  
  ![](images/Pasted%20image%2020260622175752.png)
	
	- **Interface-Specific Rules:** You cannot rely on a single global list. Rules must be written and separated into different tables for each specific physical interface on the firewall (e.g., Internet-facing, Internal Net 1, Internal Net 2).
	- **Ingress vs. Egress Filtering:** It is not enough to only consider where a packet is going (destination/egress). The rules must strictly verify where a packet is arriving from (source/ingress).
	- **Anti-Spoofing Defenses:** Applying rules per-interface is the primary way to block IP spoofing. For example, the external Internet-facing interface must contain explicit rules to block any incoming traffic that fraudulently claims to have a source IP address belonging to your internal networks.

- **IP Fragmentation Attacks:** Attackers can bypass stateless filters by intentionally fragmenting packets in abnormal ways. By manipulating the "fragment offset," an attacker can send overlapping fragments. The firewall might allow the first fragment because it looks harmless, but the second fragment mathematically overwrites the TCP header of the first fragment upon reassembly at the target host (e.g., maliciously inserting a `SYN` flag to establish a forbidden connection).

## 3. Stateful Packet Inspection (Dynamic Packet Filters)

Stateful firewalls solve the context problem by tracking the **state of connections**. Instead of evaluating every packet blindly, they maintain a real-time table of active sessions. Solve one major problem of simple packet filters, since they can check that incoming traffic for a high-numbered port is a genuine response to a previous outgoing request to set up a connection.

- **TCP Tracking:** The firewall actively monitors the TCP 3-way handshake. When an internal host sends a `SYN` packet, the firewall logs a `NEW` state. When the external server replies with `SYN/ACK`, the state upgrades to `ESTABLISHED`. Only packets matching an established state are allowed back in. The firewall then monitors for the `FIN` and `ACK` flags (from both parties) to tear down the connection state.
  
  ![](../../Pasted%20image%2020260623114950.png)

- **UDP "Statefulness":** How do you track state for a connectionless protocol like UDP or ICMP? When an internal host sends an outbound UDP request, the firewall dynamically "opens a hole" in its ruleset, anticipating a response from that specific external IP and port. It assigns a temporary timer to this expected session. If the response comes before the timer expires, it is allowed in; if not, the temporary rule vanishes.
  
  ![](../../Pasted%20image%2020260623115505.png)

## 4. Advanced Firewalls: Application & Circuit Gateways

If you need to inspect the actual payload of the traffic (big overhead), you must step up the OSI model to gateways (Proxies).

- **Application-Level Gateways (Proxies)** These firewalls understand specific application protocols (HTTP, FTP, SMTP). A Web Application Firewall (WAF), for instance, can inspect an HTTP `GET` or `POST` request to block malicious payloads like SQL injections.
	
	- Pros:
		- Logging capacity
		- Intelligent filtering
		- User-level authentication
		- Protection from wrong implementation
	- Cons:
		- Can introduce lag
		- Application-specific
		- Not always transparent
	
	- _The TLS Encryption Problem:_ Today, 95% of web traffic is encrypted via TLS. If the payload is encrypted, the proxy cannot read it. To perform Deep Packet Inspection, the proxy must operate as an intentional **Man-in-the-Middle (MitM)**. It dynamically generates a fake certificate for the destination website, decrypts the user's traffic locally, inspects it for viruses or policy violations, and then re-encrypts it with the external server's actual key to send it on its way. This causes massive processing overhead.

- **Circuit-Level Gateways (TCP Relays)** Operating at Layer 5 (Session Layer), these proxies do not care about the application content (no DPI overhead). Instead, they simply relay TCP connections.

	- **SOCKS Protocol:** The standard for this is SOCKS (used often with SSH tunneling or Tor). An internal client connects to the proxy, and the proxy connects to the external server on the client's behalf. To the external server, the traffic appears to be coming entirely from the proxy's IP address. This is highly useful for hiding internal network structures or bypassing geographic IP blocks.

- **Next-Generation Firewalls (NGFW)** Modern enterprise systems (such as OPNsense, which will be used in course labs) consolidate all these features into a single device. An NGFW acts as a traditional stateful firewall while simultaneously functioning as an Intrusion Detection System (e.g., Suricata), a VPN terminator (IPsec, WireGuard), and a traffic shaping tool to prioritize bandwidth for critical services.

## Summary of Firewall Types

- Host firewalls
- Stateless
- Stateful
- Application-gateway
- Circuit-gateway