# IPv6 security

## 1. IPv6 Security Myths vs. Reality

- **Myth: No NAT means no security.** _Reality:_ While IPv6 emphasizes end-to-end reachability without NAT, this simply means network administrators must rely on proper filtering and firewalls to block unwanted inbound traffic.
- **Myth: IPv6 networks are too big to scan.** _Reality:_ While brute-forcing a `/64` subnet is mathematically impossible, attackers use clever enumeration. They guess statically assigned IPs (like gateway `::1` or server `::80`), derive addresses from MAC addresses (EUI-64), or observe IPv4 dual-stack traffic to discover active hosts.
- **Myth: IPv6 is just IPv4 with longer addresses.** _Reality:_ IPv6 introduces an entirely new addressing architecture and new associated protocols (like NDP and ICMPv6) which bring unique security vulnerabilities.

## 2. Addressing & Interface ID (IID) Security

- **EUI-64:** This deterministic auto-configuration method embeds the host's MAC address directly into the IP address, making the host easy to track and discover.
- **Best Practices:** To protect host privacy and prevent scanning, networks should use **RFC 7217** (which generates stable, semantically opaque identifiers) or temporary randomized SLAAC addresses.

## 3. IPv6 Header Threats & Evasion Techniques

- **Covert Channels:** Attackers can secretly hide data inside the **Traffic Class** or **Flow Label** fields to bypass security monitors. While intrusion detection systems (IDS) can normalize these fields by setting them to zero, doing so breaks IPv6 Quality of Service (QoS) features.
- **Routing Header 0 (RH0):** This deprecated Extension Header allowed users to specify the exact path a packet should take. Attackers exploited this to bounce packets back and forth between two routers until the hop limit expired, causing severe resource exhaustion. Firewalls must be configured to drop RH0 packets.
- **Fragmentation Attacks:** Firewalls struggle to process fragmented Extension Headers properly. Attackers bypass filters using:
    - **Overlapping Fragments:** Maliciously crafting fragment offsets so that subsequent fragments overwrite packet data, tricking the firewall.
    - **Atomic Fragments:** Sending a packet with a fragment header that indicates no further fragments are coming. This pushes the TCP header further down the packet, often causing poorly configured firewalls to miss the TCP filtering rules entirely.

## 4. ICMPv6 and Neighbor Discovery Protocol (NDP) Threats
IPv6 relies heavily on ICMPv6 and NDP for network operation (replacing IPv4 ARP), making them prime targets for attack.

- **Smurf Attacks (ICMPv6):** An attacker sends an Echo Request to a multicast group using a spoofed source IP (the victim's IP). Every host on the network replies to the victim, flooding them with traffic. Multicast Echo Requests should be filtered.
- **Neighbor Solicitation/Advertisement (NS/NA) Spoofing:** An attacker sends spoofed NS/NA messages to alter the MAC-to-IP mappings in a victim's neighbor cache. This allows the attacker to intercept traffic, acting as a Man-in-the-Middle (MitM).
- **Rogue Router Advertisements (RA):** An attacker broadcasts malicious RAs, tricking hosts into adopting the attacker's machine as their default gateway.
- **Redirect Spoofing:** Attackers send spoofed ICMPv6 Redirect messages to maliciously alter a host's routing table, redirecting specific traffic (e.g., traffic bound for Facebook) to the attacker.
- **Duplicate Address Detection (DAD) DoS:** When a new host tries to verify its newly generated IP using DAD, the attacker immediately replies that the IP is "in use." The new host will fail to acquire an IP address and be locked out of the network.

## 5. Defenses & First Hop Security (FHS)

While standards like IPsec and Secure Neighbor Discovery (SEND) exist to secure these protocols, they are rarely implemented in the real world. Instead, defense relies on **First Hop Security (FHS) features implemented directly on network switches**.

- **RA-Guard & DHCPv6 Guard:** The switch only permits Router Advertisements and DHCPv6 replies on explicitly trusted ports (where the actual router/server is plugged in), blocking rogue devices.
- **IPv6 Snooping & Source Guard:** The switch monitors legitimate NDP traffic to build a binding table of valid IP-to-MAC addresses. It then blocks any spoofed packets originating from unauthorized IPs or MACs.
- **Egress Filtering:** To prevent IP spoofing attacks from leaving your network, edge routers must employ egress filtering and Reverse Path Forwarding (RPF) to drop outbound packets that do not contain a legitimate internal source IP.

## 6. Security Tools for IPv6

During the course labs, the primary tool used will be **Scapy**, a powerful Python library that allows you to build, manipulate, and send custom IPv6 packets layer-by-layer to test these vulnerabilities. Other standard tools include Wireshark/tcpdump for packet analysis, THC-IPV6 and Ettercap for launching attacks, OpenVAS for vulnerability scanning, and Snort/Zeek for intrusion detection.