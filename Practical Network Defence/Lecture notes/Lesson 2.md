# Traffic Monitoring

**1. Network Layering Models & Encapsulation**

- **Packet-Switched Networks:** The internet was designed as a packet-switched network during the Cold War to survive node failures. Every packet is independent, meaning packets from the same connection can take different routes to their destination.
- **Architectural Models:**
    - **OSI Model:** A 7-layer reference model (Application, Presentation, Session, Transport, Network, Data Link, Physical).
    - **TCP/IP Model:** A simpler 4-layer model (Application, Transport, Internet, Network Access) created by the IETF.
- **Encapsulation (The "Matryoshka" Concept):** As data moves down the layers from the application to the physical layer, each protocol adds its own header (control information) to the data.
- **Layer Terminology:** Data is referred to differently depending on its current layer:
    - **Application:** Data / Payload.
    - **Transport:** **Segments** (TCP) or **Datagrams** (UDP).
    - **Network (Internet):** **Packets** (e.g., IP packets).
    - **Data Link:** **Frames** (e.g., Ethernet frames).
    - **Physical:** **Bits** (transformed into analog signals like radio waves, light pulses, or electrical tension).

**2. Addressing Across Layers**

Each layer utilizes a different addressing mechanism to route information:

- **Application Layer:** Domain names (e.g., _www.sapienza.it_).
- **Transport Layer:** **Ports** (range 0 to 65535). Port numbers identify the specific process or service receiving the data. Ports 0-1023 are "well-known" standard ports.
- **Internet/Network Layer:** **IP Addresses**. These represent logical addressing indicating **"where you are"** geographically and logically in the network.
- **Data Link Layer:** **MAC Addresses**. These represent physical addressing indicating **"who you are"** (the specific network interface card).

**3. Deep Dive: Network Layer & IP**

- **Unreliability:** IP is a completely unreliable protocol ("like the pigeon of protocols"). It sends packets without guaranteeing delivery, expecting the upper layers to handle reliability if needed.
- **Time to Live (TTL):** To prevent packets from circulating infinitely in routing loops, the IP header includes a TTL field. Every router that forwards the packet decrements the TTL by 1. If it reaches 0, the packet is dropped, and an **ICMP Time Exceeded** message is sent back to the source (this mechanism is how `traceroute` maps network paths).
- **IPv4 vs. IPv6:** The IPv6 header drops the fragmentation fields and flags found in IPv4 to improve efficiency, offloading fragmentation to "extension headers." Additionally, IPv6 uses 128-bit addresses compared to IPv4's 32-bit addresses.

**4. Deep Dive: Transport Layer (TCP vs. UDP)**

- **TCP (Transmission Control Protocol):** Connection-oriented and highly reliable. It guarantees the correct sequence and delivery of packets.
    - Features a complex header (Sequence Number, Acknowledgement Number, Window Size, Flags).
    - Establishes connections via a **3-way handshake:** SYN -> SYN+ACK -> ACK.
    - Uses: HTTP (80), HTTPS/SSL (443), SSH (22), FTP (20/21).
- **UDP (User Datagram Protocol):** Connectionless and unreliable. It does not control data exchange flow or guarantee delivery, making it faster.
    - Features a very simple header (Source Port, Destination Port, Length, Checksum).
    - Uses: DNS (53), DHCP (67/68), TFTP (69).
- _Note on QUIC:_ Modern web browsing is increasingly replacing TCP with QUIC (built over UDP) for more efficient reliable communication.

**5. Key Application Services**

- **DNS (Domain Name System):** Resolves human-friendly domain names to IP addresses. It operates on a hierarchical system (Root -> Top-Level Domain -> Domain -> Host) so that no single server has to hold the entire internet's address book.
- **TLS (Transport Layer Security):** A general-purpose mechanism that sits between the application and transport layers to encrypt communication (e.g., turning HTTP into HTTPS).

**6. Network Traffic Monitoring & Sniffing**

- **Capture Tools:** Network traffic is monitored using tools built on the `pcap` (or `winpcap`) library, such as **tcpdump** (command line), **dumpcap**, and **Wireshark/tshark**. Zeek and Netflow are used for broader traffic inspection and statistics.
- **Promiscuous vs. Monitor Mode:**
    - **Promiscuous Mode:** Required for Wireshark to capture packets on a wired network. It disables the network interface card's default filter, forcing it to process _all_ packets on the wire, not just those destined for its own MAC address.
    - **Monitor Mode:** Specific to Wi-Fi. It allows the capture of wireless traffic across different networks and channels in the area, not just the access point you are connected to.
- **Wireshark Filters:**
    - **Capture Filters:** Use BPF (Berkeley Packet Filter) syntax to limit what data is actually recorded and processed (discarded packets are lost forever).
    - **Display Filters:** Hide packets in the UI without deleting them from the capture file. They use comparison operators and can be built interactively.

**7. Sniffing Techniques, Attacks, & Defenses**

- **How to Capture Traffic on Switched Networks:** Because switches only send traffic to the specific port where the destination MAC resides, sniffing requires physical taps, or **Port Mirroring (SPAN/RAP)** on a managed switch.
- **Aggressive Sniffing Attacks:**
    - **ARP Cache Poisoning:** Sending fake ARP replies to steal IP addresses and intercept traffic (e.g., using Ettercap).
    - **MAC Flooding:** Flooding the switch's CAM table so it fails open and acts like a hub, broadcasting all traffic to all ports.
    - **DHCP Redirection:** Creating a rogue DHCP server to exhaust legitimate IP addresses and set the attacker as the default gateway.
- **Defenses:** Modern switches prevent these attacks using **Dynamic Address Resolution Inspection (DAI)** to validate ARP packets, and **DHCP Snooping** to block rogue untrusted DHCP servers.
- **Wi-Fi Security Note:** Avoid "Open" networks (sniffable by anyone) and "WEP" (easily cracked with a single packet). Use "WPA/WPA2" (like Eduroam) because it negotiates a unique per-session encryption key for every user, making it impossible to sniff other users' traffic even if you are on the same network.