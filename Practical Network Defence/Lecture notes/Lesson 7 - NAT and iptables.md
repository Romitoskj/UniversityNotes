## 1. Network Address Translation (NAT) Fundamentals

- **The Addressing Problem:** NAT was originally introduced to combat the exhaustion of IPv4 addresses by allowing many private addresses to share a few public addresses.
- **NAT Goal:** Translate the addresses for connecting to the Internet a LAN using un-routable in-house LAN addresses. It allows a whole network to connect to the internet by using only one IP addresses provided by the ISP.
- **Address Types:** 
	- **Routable (public) addresses** must be globally unique on the internet
	- **Non-routable (private) addresses** (such as `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`) are reserved by RFC 1918 and cannot be routed over the public internet, allowing any organization to use them internally.
- **Key Benefits:** Beyond conserving addresses, NAT hides the internal network topology from the outside world, allows organizations to dynamically alter their internal IP scheme without external impact, and makes it simple to switch Internet Service Providers (ISPs) without changing private IP addresses.

## 2. Core Types of NAT

- **Source NAT (SNAT):** Translates the source IP address of packets leaving the private network (LAN to WAN), masquerading the entire session as originating from the firewall or router that perform the NATting process.
	
	![](images/Pasted%20image%2020260624111429.png)
	
	- **Network Address Port Translation (NAPT):** The most common form of SNAT multiplexes traffic of a number of private host into a single public IP address by translating transport-level identifiers (TCP/UDP source ports). The router maintains a **NAT table** to map which internal host/port initiated the request to a unique external port, ensuring returning traffic reaches the right device. By default block all incoming requests.
	  ![](../../Pasted%20image%2020260624112531.png)
	  
- **Destination NAT (DNAT):** Also known as Port Forwarding or a Virtual Server, this enables external clients to access servers located inside the private LAN. The firewall intercepts requests addressed to its public IP and translates the destination IP to the internal server's private address. This process allows for port multiplexing, forwarding different ports to entirely different internal machines.

## 3. Limitations of NAT & Mitigation

- **The Broken Applications Problem:** NAT breaks protocols that embed IP addresses directly inside the packet payload (like FTP and SIP), P2P connections, and encrypted protocols like IPsec, where modifying the packet invalidates the signature.
- **Mitigation Strategies:**
    - **Application Level Gateways (ALGs):** Allow firewalls to inspect specific application traffic and rewrite the IP/port information hidden in the payload.
    - **STUN and TURN:** STUN discovery servers help NAT-bound hosts discover their public IPs for direct P2P connections, while TURN servers act as relays forwarding data if direct communication fails.
- **UDP Hole Punching:** A STUN-assisted technique where hosts exchange their public endpoints to establish outbound NAT states (a "hole") on their respective firewalls, allowing direct return traffic.

## 4. Implementing NAT and Traffic Regulation with `iptables`

The Linux kernel regulates network traffic using `iptables`, a packet filtering firewall implementation that operates directly within the kernel space to filter at both the Network layer (IPs) and Transport layer (TCP/UDP ports). It evolved from older tools like `ipchains` and `ipfw`, with `nftables` planned as its modern successor.

### 4.1**Architecture: Tables and Chains**
The framework structures operations into **Tables** (categories of operations) and **Chains** (stages in the routing process). Packets are evaluated against rules from top to bottom, and the packet's fate is dictated by the **first matching rule**. If the packet reaches the end of the chain without a match, the chain's default policy (e.g., DROP or ACCEPT) applies.

- **The Four Tables (In Priority Order):**
    1. **RAW:** Has the highest priority, operates in PREROUTING and OUTPUT, and is used to create exceptions to connection tracking.
    2. **MANGLE:** Used strictly for manipulating bits in the IP/TCP headers (like Time to Live) and must not be used for filtering or NAT.
    3. **NAT:** Dedicated entirely to Network Address Translation.
    4. **FILTER:** The default table used for standard packet filtering and enforcing security policies.

**Understanding Chains & Packet Flow**

- **PREROUTING:** Applied immediately when a packet arrives, _before_ a routing decision is made. **DNAT** occurs here because the destination IP must change before the router decides where to send it.
- **INPUT / FORWARD / OUTPUT (Filter Chains):** Packets destined for the host machine itself are evaluated by `INPUT`, packets passing through the machine to another network are evaluated by `FORWARD`, and packets generated internally by the machine are evaluated by `OUTPUT`.
- **POSTROUTING:** Applied _after_ a routing decision, right before the packet exits the interface. **SNAT** is performed here to assign the correct outgoing public IP.

### 4.2 Command Syntax, Targets, and Management

- **Switches:** Rules are built using switches such as `-t` (Table), `-A` (Append), `-F` (Flush), `-P` (Change policy), `-p` (Protocol), `-s` / `-d` (Source/Destination IP), `--sport` / `--dport` (Ports), and `-i` / `-o` (Interfaces).
- **Rule Management:** Rules can be saved and restored persistently using `iptables-save > file.bk` and `iptables-restore < file.bk`.
- **Targets (Packet Fates via `-j`):**
    - `ACCEPT`: Permits the packet.
    - `DROP`: Silently blocks the packet.
    - `REJECT`: Blocks the packet but actively sends an ICMP error message to the sender.
    - `LOG`: A **non-terminating target** that sends packet info to the syslog daemon (customizable via `--log-level` and `--log-prefix`) and continues processing the next rule. To log and drop, you must write a `LOG` rule immediately followed by a `DROP` rule.

### 4.3 Stateful Inspection & Connection Tracking**

To avoid rigid, stateless setups, administrators use the `conntrack` module (`-m state --state`).

- **Connection States:** Include **NEW** (start of a connection), **ESTABLISHED** (part of an existing connection), **RELATED** (new connection related to an existing one), and **INVALID** (unidentifiable packet).
- **Mechanics:** Using internal logic, iptables tracks "connections" even for connectionless protocols like UDP and ICMP. By enforcing that inbound traffic is only allowed if the state is `ESTABLISHED`, firewalls can also mitigate "half-open" attacks that attempt to consume a server's TCP stack memory.

### 4.4 NAT Mechanics & Targets in iptables

- **The "First Packet" Rule:** In the NAT table, **only the very first packet in a stream is evaluated** against the rules. Once translated, a state is created, and all subsequent packets in that connection automatically receive the identical action.
- **Targets:**
    - **SNAT:** Static source translation.
    - **MASQUERADE:** A dynamic form of SNAT used when the firewall's WAN IP is dynamically assigned (e.g., via DHCP).
    - **DNAT:** Destination translation (Port Forwarding).
    - **REDIRECT:** Redirects the packet entirely to the firewall machine itself.
- **User-Defined Chains:** Systems like Docker heavily utilize custom chains (like `DOCKER-ISOLATION`) within the FILTER and NAT tables to isolate containers safely and route port-forwarded traffic.