# Firewall

**1. Traffic Regulation: Philosophy & Architecture**

While a router’s primary job is to ask "where should this packet go?" by checking routing tables against the destination IP, a firewall adds a crucial second question: **"Is this packet allowed to go there?"**. To enforce this, network administrators design architectures based on key security principles:

- **Choke Points:** You must force all traffic through a single, heavily monitored access point to ensure no traffic bypasses the firewall via "backdoors".
- **Defense in Depth:** Security should rely on overlapping systems. Instead of a single firewall, modern networks use multiple layers of protection (e.g., an external screening router, a perimeter network, an internal firewall, and individual host-based firewalls).
- **The DMZ (Demilitarized Zone):** A secure, neutral sub-network placed between the public internet and the private internal network. The DMZ hosts systems that need to be publicly accessible (like Web or Mail servers) on hardened machines called **Bastion Hosts**. If a Bastion Host in the DMZ is compromised, the attacker still faces another internal firewall before they can reach the private network.

**2. Stateless Packet Filtering (The "Basic" Firewall)**

Stateless packet filters (often implemented as Access Control Lists, or ACLs, on screening routers) evaluate every single packet entirely independently, with no memory of what happened before. They filter purely based on the Network and Transport layer headers: Source/Destination IP, Source/Destination Port, and TCP Flags.

**Deep Dive: Vulnerabilities of Stateless Filters** Because stateless filters lack context, their rules are often too rigid or dangerously broad:

- **The Direction/Port Problem:** If you write a rule to allow internal hosts to receive responses from external servers (which typically use high-numbered ports >1023), a clever attacker can spoof their source port to match an allowed service (e.g., port 25) and freely push traffic into your internal network. To fix this, rules must strictly check for the `ACK` flag (indicating an established connection), but even this is flawed.
- **IP Fragmentation Attacks:** Attackers can bypass stateless filters by intentionally fragmenting packets in abnormal ways. By manipulating the "fragment offset," an attacker can send overlapping fragments. The firewall might allow the first fragment because it looks harmless, but the second fragment mathematically overwrites the TCP header of the first fragment upon reassembly at the target host (e.g., maliciously inserting a `SYN` flag to establish a forbidden connection).

**3. Stateful Packet Inspection (Dynamic Packet Filters)**

Stateful firewalls solve the context problem by tracking the **state of connections**. Instead of evaluating every packet blindly, they maintain a real-time table of active sessions.

- **TCP Tracking:** The firewall actively monitors the TCP 3-way handshake. When an internal host sends a `SYN` packet, the firewall logs a `NEW` state. When the external server replies with `SYN/ACK`, the state upgrades to `ESTABLISHED`. Only packets matching an established state are allowed back in. The firewall then monitors for the `FIN` and `ACK` flags to tear down the connection state.
- **UDP "Statefulness":** How do you track state for a connectionless protocol like UDP or ICMP? When an internal host sends an outbound UDP request, the firewall dynamically "opens a hole" in its ruleset, anticipating a response from that specific external IP and port. It assigns a temporary timer to this expected session. If the response comes before the timer expires, it is allowed in; if not, the temporary rule vanishes.

**4. Advanced Firewalls: Application & Circuit Gateways**

If you need to inspect the actual payload of the traffic, you must step up the OSI model to gateways (Proxies).

**Application-Level Gateways (Proxies)** These firewalls understand specific application protocols (HTTP, FTP, SMTP). A Web Application Firewall (WAF), for instance, can inspect an HTTP `GET` or `POST` request to block malicious payloads like SQL injections.

- _The TLS Encryption Problem:_ Today, 95% of web traffic is encrypted via TLS. If the payload is encrypted, the proxy cannot read it. To perform Deep Packet Inspection, the proxy must operate as an intentional **Man-in-the-Middle (MitM)**. It dynamically generates a fake certificate for the destination website, decrypts the user's traffic locally, inspects it for viruses or policy violations, and then re-encrypts it with the external server's actual key to send it on its way. This causes massive processing overhead.

**Circuit-Level Gateways (TCP Relays)** Operating at Layer 5 (Session Layer), these proxies do not care about the application content (no DPI overhead). Instead, they simply relay TCP connections.

- **SOCKS Protocol:** The standard for this is SOCKS (used often with SSH tunneling or Tor). An internal client connects to the proxy, and the proxy connects to the external server on the client's behalf. To the external server, the traffic appears to be coming entirely from the proxy's IP address. This is highly useful for hiding internal network structures or bypassing geographic IP blocks.

**5. Next-Generation Firewalls (NGFW)**

Modern enterprise systems (such as OPNsense, which will be used in course labs) consolidate all these features into a single device. An NGFW acts as a traditional stateful firewall while simultaneously functioning as an Intrusion Detection System (e.g., Suricata), a VPN terminator (IPsec, WireGuard), and a traffic shaping tool to prioritize bandwidth for critical services.