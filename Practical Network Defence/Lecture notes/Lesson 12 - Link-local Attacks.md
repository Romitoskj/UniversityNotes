## 1. Network Sniffing and Eavesdropping
Network eavesdropping (or sniffing) involves capturing packets transmitted by other nodes to read sensitive data like passwords or session tokens. To achieve this, a network interface card (NIC) is placed in **promiscuous mode**, allowing it to pass all intercepted traffic to the user level for analysis, rather than dropping packets not destined for its own MAC address.

For sniffing to be successful, the attacker must be on the traffic's path or in the same broadcasting domain. While legacy non-switched LANs (using hubs) natively broadcast all traffic, modern switched LANs isolate traffic. To sniff on a switched network, an attacker must break the switch's segmentation mechanisms:

### 1.1 Breaking Switch Segmentation: CAM Overflow
Switches learn and store the MAC addresses of connected hosts in a fixed-size Content Addressable Memory (CAM) table. When a switch receives a frame destined for an unknown MAC address, it "floods" the frame to all ports.

- **The Attack:** Attackers exploit the finite size of the CAM table using tools like `macof`, which floods the switch with thousands of fake, randomly generated MAC addresses.
- **The Impact:** Once the CAM table is full, legitimate MAC addresses cannot be stored, forcing the switch to flood all incoming frames to every port, effectively turning the switch into a hub and allowing the attacker to sniff all traffic. This can also result in a Denial of Service (DoS) if the switch freezes or crashes.
- **Mitigation:** The primary defense is **Port Security**, which limits the number of MAC addresses allowed per physical switch port or restricts ports to specific known MAC addresses.

### 1.2 ARP Poisoning and Man-in-the-Middle (MITM)
The Address Resolution Protocol (ARP) translates IP addresses to MAC addresses dynamically, it works like that:
1. An ARP request message is placed in a frame and broadcasted to all computers on the network;
2. Each computer receives the request and examines the IP address;
3. The computer mentioned in the request send a response with its MAC address and all other computer discard the request;
4. The computer that made the request store the address in a dynamic table that holds the IP-MAC pairs called ARP table, which starts empty and is filled as the MAC address are collected while unused addresses are removed after a timeout.

But it lacks any built-in security or ownership validation:
- **Gratuitous ARP Response:** Hosts normally broadcast Gratuitous ARP messages (without request) to announce their IP-to-MAC pairing to the local network and prevent duplicate IPs.
- **The Attack:** An attacker can misuse Gratuitous ARP by repeatedly broadcasting forged messages claiming that their MAC address corresponds to a victim's IP address (often the default gateway).
- **The Impact:** Because ARP tables cache these replies blindly, the network hosts will overwrite their legitimate routing entries with the attacker's MAC. The attacker can then perform a **Man-in-the-Middle (MITM)** attack, silently intercepting, altering, or dropping traffic between hosts and the gateway.

## 2. IPv6 Neighbor Discovery (ND) Threats
IPv6 eliminates ARP and relies on the **Neighbor Discovery Protocol (NDP)** via ICMPv6. Address resolution is handled using Neighbor Solicitation (NS) multicast requests and Neighbor Advertisement (NA) unicast replies. However, this introduces new link-local vulnerabilities:

- **Duplicate Address Detection (DAD) DoS:** DAD is used by hosts to verify that their intended IPv6 address is unique on the link. In this attack, every time a new host sends a solicitation asking if an IP is available, the attacker immediately sends a forged reply claiming the address is already in use. The victim host is repeatedly blocked and can never acquire an IPv6 address.
- **Rogue Router Advertisements (RA):** In IPv6, routers periodically send RAs to provide hosts with network prefixes and default gateway information for Stateless Address Autoconfiguration (SLAAC). A "malicious last hop router" can send rogue RAs to trick hosts into using the attacker as their default gateway. This enables VPN bypass, MITM attacks, and credential theft.
- **RA Flooding:** Attackers can overwhelm the network by flooding hosts with thousands of bogus Router Advertisements containing fake prefixes. This forces the hosts to process excessive routing information, leading to frozen operating systems (especially older OSs like Windows 7/8) or severe connectivity loss.

## 3. DHCP Starvation and Rogue DHCP
An attacker can perform a DoS attack by requesting and exhausting all available DHCP addresses in a network. Once the legitimate address pool is empty, the attacker deploys a **Rogue DHCP Server**. Because DHCP responses automatically provide clients with DNS server and default gateway configurations, the attacker can redirect all routing and name resolution to malicious endpoints, facilitating complete MITM compromises.

## 4. ICMPv6 Redirect Attacks
ICMP redirects are normally used by a router to inform a host that there is a better, more direct route on the local link to reach a specific destination.

- **The Attack:** An attacker sends a spoofed redirect message to a victim, tricking the victim's routing table into believing the attacker is the optimal next-hop for a target destination.
- **Mitigation:** The primary defense against this at the host level is to configure the operating system kernel to reject all ICMP redirects by default (e.g., setting `accept_redirects` to FALSE).

## 5. First Hop Security (Switch-Level Mitigations)
Defending against link-local and IPv6 ND attacks requires enforcing security at the Data Link layer (Layer 2) within the network switches. Key switch-level mitigations include:

- **RA Guard:** The switch is configured to only allow Router Advertisements on explicitly authorized ports (where the legitimate router is connected), dropping rogue RAs coming from user ports.
- **DHCPv6 Guard:** Similar to RA Guard, the switch blocks DHCP server responses coming from unauthorized switch ports.
- **IPv6 Snooping & ND Inspection:** The switch inspects Neighbor Discovery traffic to ensure hosts are not spoofing IP addresses or claiming MAC addresses that do not belong to their physical port.
- **802.1X Authentication:** Enforcing port-level network access control to ensure only authenticated devices can transmit on the local link.