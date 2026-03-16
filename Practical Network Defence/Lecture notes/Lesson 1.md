# Networking 101

**1. Internet Architecture & Infrastructure**

- **The Internet** is an interconnected "network of networks" operating on a hierarchical structure.
- **Network Hierarchy:**
    - **Network Edge:** End hosts (clients, servers, P2P) and applications (HTTP, email, etc.).
    - **Access Networks:** Wired or wireless links connecting hosts to the edge router.
    - **Network Core:** A mesh of routers and optical fiber links that determine the most efficient path for packets. It is managed by global (Tier-1) and regional (Tier-2) ISPs.
- **Peering:** Tier-1 ISPs connect their backbones at global access points called **Internet eXchange Points (IXPs)** to directly exchange internet traffic.
- **Standards:** Maintained by the **IETF** (Internet Engineering Task Force) through **RFCs** (Request for Comments).

**2. Network Protocols**

- Protocols specify the rules for network communication, including message formats, syntax, semantics, flow control, and timing.
- Network architectures use **modularization (layers)** to hide implementation details, making system updates, maintenance, and development much easier.

**3. The Access Layer & Ethernet (Physical Addressing)**

- The access layer provides connectivity among end-points under the same local management (LANs).
- **Ethernet (IEEE 802.3)** is the standard protocol for this layer.
- **MAC Addresses:** Every Network Interface Card (NIC) has a fixed, **48-bit (6-byte) physical MAC address** that uniquely identifies "who" the host is.
- **Switches & Broadcast Domains:** An Ethernet network forms a broadcast domain. **Switches** help segment this network by learning source MAC addresses and storing them in **CAM tables**. Switches only replicate traffic to the specific segment where the destination MAC resides, limiting unnecessary packet explosion (broadcast messages are an exception and get replicated to all).

**4. Distribution/Core Layers & IP (Logical Addressing)**

- Large networks cannot rely solely on Ethernet because heavy broadcast traffic is inefficient. We need a logical division to reach remote hosts.
- **IP Addresses:** Provide **logical addressing** ("where you are"), much like a home address that changes if you move.
    - **IPv4:** 32 bits, divided into 4 octets separated by dots (e.g., 69.58.201.25).
    - **IPv6:** 128 bits, separated by colons.
- **Routers:** Routers act as **Default Gateways** connecting local subnets to the internet distribution/core layers. They determine if an IP is local (directly connected) or remote.

**5. IP Address Types & Subnetting**

- IP addresses are divided into a **Network portion** and a **Host portion**, defined by a **Subnet Mask**.
- **Address Types:**
    - **Unicast:** One-to-one communication.
    - **Broadcast:** One-to-all hosts on a subnet.
    - **Multicast:** One-to-many (a specific group of IP addresses).
- **Classful vs. Classless Addressing:**
    - **Classful:** Historically divided into Classes A, B, C (standard hosts), D (multicast), and E (reserved). This resulted in poor flexibility and wasted addresses.
    - **Classless Inter-Domain Routing (CIDR):** Introduced **Variable Length Subnet Masks (VLSM)** (e.g., `/24`, `/26`). This notation tells you exactly how many bits specify the network address.
- **Calculating Hosts:** In any CIDR network, the first address is reserved for the network, and the last address is reserved for the broadcast. The formula for usable hosts is **2^(32 - netmask bits) - 2**.
- **Conserving IP Addresses:** To avoid wasting IPv4 addresses, strategies include using `/31` masks for point-to-point links (RFC 3021), NAT, and transitioning to IPv6.
- **Routeable vs. Non-Routeable (Private):** Non-routeable IPs cannot be routed on the public internet. These include RFC1918 private networks (10.x.x.x, 172.16.x.x, 192.168.x.x), loopback addresses (127.0.0.0/8), and link-local addresses.

**6. Next Steps & Homework**

- **Subnetting Practice:** Practice calculating the network part, host part, network size, network address, and broadcast address for various IPs (e.g., `10.11.12.0/21`, `192.168.4.32/27`).
- **Local vs Remote identification:** Practice determining if a destination IP falls within the same subnet as a source IP.
- **Linux Practice:** Continue working on the **OverTheWire Bandit wargame** via the command line. Aim to reach level 33 or 34, taking careful notes on the passwords and how you solved each challenge.