## 1. VPN Principles and Security Goals

A Virtual Private Network (VPN) is a virtual network built on top of an existing, insecure network infrastructure to provide a secure communication mechanism between endpoints.

- **Traditional Goals:** Confidentiality (preventing eavesdropping), Integrity (detecting data modification), and Peer Authentication.
- **Extended Goals:** Replay protection (using timestamps or nonces to prevent attackers from re-sending captured packets), Access Control (granular permissions based on authentication), and Traffic Analysis Protection (preventing attackers from identifying traffic patterns).
- **Usability:** A critical subsidiary goal. A VPN must be transparent to users and applications, flexible, and simple; poor usability ultimately leads to users bypassing security entirely.

This secure communication mechanism can be implemented Site-to-site (between two separated networks), Host-to-site (between a network and a remote host) or Host-to-host (directly between two remote hosts).

## 2. Network Layering and Encryption Strategies

Where you apply encryption in the OSI/TCP-IP stack model drastically changes the VPN's flexibility and transparency.

- **Physical / Datalink Layer:** Protects a single link. It is completely transparent but highly inflexible, as it requires the two endpoints to be directly connected or on the same local segment (e.g., military radio communications or modem-based PPTP/L2TP).
- **Network Layer (e.g., IPsec):** Protects end-to-end between hosts or sites. It encrypts the payload for the final destination, leaving the outer routing headers intact so intermediate routers can still forward the packet.
- **Transport Layer (e.g., SSL/TLS):** Protects end-to-end between processes. This is highly flexible because it operates below the application layer, meaning any application that relies on TCP can be secured without needing software modifications.
- **Application Layer (e.g., PGP, S/MIME):** Security is implemented by the application itself (e.g., encrypting an email payload before it hits the network). This is excellent for specific tasks but lacks transparency because it requires dedicated, specialized software.
  
Transport and network layers are the most popular choices for VPNs.

## 3. Tunneling Mechanics

**Tunneling** is the operation of running one network connection on top of another. The entire Protocol Data Unit (PDU)—including its original headers—is encapsulated within another PDU.

- **Secure Tunneling:** The original PDU is encrypted before being encapsulated. To intermediate routers on the internet, the communication simply appears as a connection between the two VPN endpoints; they cannot see the inner contents or the true final destination of the packet.
- **Split vs. Full Tunneling:**
    - **Split Tunneling:** Only specific traffic (e.g., traffic bound for the university/corporate network) is routed through the encrypted VPN tunnel, while regular internet traffic (like browsing Google) uses the host's default gateway. This is common in academic and corporate setups.
    - **Full Tunneling:** _All_ user network traffic is forced through the VPN tunnel. This is the standard approach for commercial privacy VPNs (e.g., NordVPN, ProtonVPN).

## **4. VPN Device Placement Strategies**

Deciding where to place the VPN terminator (the device that encrypts/decrypts the tunnel) impacts security, functionality, and performance.

- **VPN Functionality in the Firewall:** The firewall acts as the VPN terminator. It is simple to administer because all rules and VPN configurations are in one box. However, it exposes the firewall to external users (port 443 must be open) and limits you to the features provided by your firewall vendor.
- **VPN Device in the Internal Network:** The VPN is placed completely behind the firewall. While this protects the VPN device from external DMZ attacks, it introduces a massive risk: traffic exiting the VPN is already decrypted and bypasses the firewall's inspection, meaning a compromised VPN device compromises the entire internal network.
- **Single-Interface VPN Device in the DMZ:** The VPN device sits in the Demilitarized Zone. Unencrypted traffic leaving the VPN must pass back through the internal firewall to reach internal hosts, allowing the firewall and Intrusion Detection Systems (IDS) to analyze it. The downside is that it requires opening numerous firewall ports between the DMZ and the internal network.
- **Dual-Interface VPN Device in the DMZ:** The VPN device has one interface facing the internet/DMZ and a second interface wired directly to the internal network. This protects decrypted traffic from other hosts in the DMZ, but it introduces significant routing complexity.

## 5. SSL / TLS Tunneling

SSL (Secure Sockets Layer) and its modern successor TLS (Transport Layer Security) operate at the Transport layer to provide a secure byte stream for TCP-based protocols.

- **Protocol Architecture:**
    - **Record Protocol:** Handles the basic encryption, compression, and Message Authentication Code (MAC) integrity checks of the application data.
    - **Handshake Protocol:** A 4-phase process where the client and server establish security capabilities, authenticate the server (using X.509 digital certificates and Public Key Infrastructure), and securely agree on a symmetric master secret key for the session.
- **SSL VPN Architectures:**
    - **SSL Portal VPN:** Users access specific protected services directly through a web browser interface.
    - **SSL Tunnel VPN:** The VPN acts as a true network extension, allowing broader access to internal network services.
- **The Heartbleed Bug:** TLS includes a "Heartbeat" extension (RFC 6520) to keep idle sessions alive. The infamous "Heartbleed" vulnerability occurred because the OpenSSL library failed to verify the payload length specified by the sender. An attacker could send a tiny payload but declare it as the maximum size, forcing the server's memory buffer to return up to 64KB of adjacent memory, leaking private keys and passwords.

## **6. IPsec (IP Security)**

IPsec (standardized in RFC 4301) is a Network Layer protocol suite for providing secure communication over IP networks. It is natively built into IPv6 and available as an add-on for IPv4. Because it operates at the Network Layer, it is completely transparent to upper-layer applications, though its architecture is more complex to configure than TLS.

It guarantees several core security fundamentals: data origin authentication, connectionless data integrity (detecting transit modifications), payload confidentiality, and replay protection.

**Core Protocols** IPsec relies on three primary (sub)protocols to function:

- **AH (Authentication Header):** Provides data integrity and origin authentication for the IP payload and immutable IP header fields (fields that do not change in transit, like the source address). It provides _no_ encryption.
- **ESP (Encapsulating Security Payload):** Provides payload encryption for confidentiality, and can optionally provide authentication.
- **IKEv2 (Internet Key Exchange version 2):** Standardized in RFC 5996, IKEv2 operates over UDP ports 500 and 4500 to dynamically negotiate cryptographic suites, exchange keys, and mutually authenticate endpoints. The IKEv2 process requires two main exchanges:
    - **Phase 1 (IKE_SA_INIT):** Negotiates encryption, integrity protection algorithms, and Diffie-Hellman values to create an `IKE_SA`, which encrypts and protects all subsequent IKE communications.
    - **Phase 2 (IKE_AUTH):** Authenticates the previous messages (often using X.509 Public Key Certificates, Pre-Shared Keys, EAP, or Xauth) and creates the first `CHILD_SA`. The `CHILD_SA` is the actual IPsec tunnel that protects the IP traffic with AH or ESP.

### 6.1 Architecture: Policies and Associations

Within the operating system kernel, IPsec relies on two critical databases to evaluate traffic:

- **Security Policy Database (SPD):** Stores Security Policies (SPs) set by the administrator. An SP dictates the security requirements for a specific IP stream. Its actions can be configured to **Discard** the packet, **Bypass** IPsec (send in cleartext), or **Secure** the packet using IPsec.
- **Security Association Database (SAD):** Stores Security Associations (SAs). An SA is a simplex (unidirectional) channel detailing the specific encryption/authentication algorithms, modes, and keys to be applied. Because it is simplex, bidirectional communication requires at least two SAs. SAs are uniquely identified by a 32-bit **Security Parameters Index (SPI)** alongside the destination IP.

## 6.2 Packet Processing Flow

- **Outgoing Traffic:** The kernel intercepts an outbound packet and checks the SPD. If the policy requires the packet to be secured, the kernel looks for a corresponding SA in the SAD. If no SA exists, it triggers the IKE daemon to negotiate one. Once the SA is found, the kernel applies the AH/ESP transformations and sends the packet.
- **Incoming Traffic:** The kernel extracts the SPI from the incoming IPsec header and looks up the corresponding SA in the SAD to decrypt and authenticate the packet. Before handing the data up to the transport layer, the kernel strictly checks the SPD to ensure the packet's protection perfectly matched the required security policy (dropping it if it fails).

### 6.3 IPsec Modes

- **Transport Mode:** Provides protection for a Transport-layer payload (e.g., the TCP segment) embedded within an IP packet. The original IP header remains intact, visible, and is used for routing.
- **Tunnel Mode:** Takes the entire original IP packet, encrypts it, and encapsulates it inside a brand-new outer IP header. This provides traffic flow confidentiality, as intermediate internet routers only see the new outer IP header.

### 6.4 Linux Implementation
Historically, Linux systems configured IPsec using the `ipsec-tools` framework, which included `setkey` for manually manipulating the SAD/SPD databases and `racoon` as the IKE daemon. However, manually setting up SAs is highly error-prone and can lead to inconsistent policies. Modern enterprise deployments and courses rely on automated keying frameworks like **strongSwan** (utilizing the `charon` IKE daemon and the `swanctl` command-line tool) to robustly and securely establish IPsec connections.