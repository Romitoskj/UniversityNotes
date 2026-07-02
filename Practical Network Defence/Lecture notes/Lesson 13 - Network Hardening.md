## **1. Introduction to Network Hardening**

Network hardening is the process of reducing a system's vulnerability surface by shutting down unused services, removing unnecessary accounts, and strengthening access controls. Because networks are composed of multiple interconnected devices, a breach in just one component can compromise the entire infrastructure.

To effectively protect a network, administrators must secure devices across their three functional planes:

![](../../Pasted%20image%2020260702153736.png)

- **Management Plane:** 
	- How administrators monitor, configure, and access the device.
	- Breaches caused by overly simple passwords or insecure protocol, resulting in unauthorized access or loss of access to the device
- **Control Plane:**
	- The protocols and mechanisms the devices use to perform their tasks (e.g., routing protocols).
	- Violations caused by unauthorized data exchange with the device resulting in loss of performance or DoS.
- **Data Plane:** 
	- The actual traffic forwarded by the network devices.
	- Violations caused by external events, malicious interventions and failures resulting in alteration of packet paths and a block of network services.

Anomalies in one plane easily reflect into the others, making comprehensive protection across all three mandatory.

## **2. Management Plane Protection**

Protecting the management plane ensures that attackers cannot gain unauthorized control of network appliances.

- **Secure Remote Access:** Cleartext protocols like Telnet and HTTP expose credentials to interception. Administrators must rely strictly on encrypted tunnels like SSH (preferably SSHv2) and HTTPS.
- **Password Policies & Lockouts:** Devices should enforce strong passphrases (combining uppercase, lowercase, numbers, and symbols) to mathematically defeat brute-force attacks. Passwords should be stored securely (encrypted), and systems should temporarily lock accounts after multiple failed attempts. They also should change frequently.
- **The AAA Framework:**
    - **Authentication:** Verifying the user's identity.
    - **Authorization:** Restricting access to authorized components, commonly using Role-Based Access Control (RBAC) which associate roles to users each one with a set of permission on system objects.
    - **Accounting/Auditing:** Keeping an unalterable log of user actions.
- **Centralized Management:** Instead of managing local databases on every device, organizations should use centralized Access Control Servers (ACS) running protocols like RADIUS (for end-users) or TACACS+ (for administrators). This allows you to have a centralized point from which to manage all the devices on the network.
- **Time Synchronization (NTP):** To perform reliable auditing and cross-device log correlation (crucial for SIEMs), all devices must synchronize their clocks. The Network Time Protocol version 3 (NTPv3) is preferred as it includes authentication and integrity verification.
- **Syslog Centralization:** Log messages should be securely forwarded to a centralized syslog server. Syslog uses 8 severity levels (from _0 Emergency_ to _7 Debugging_). Administrators must carefully configure these levels, relying heavily on lower severity levels like debugging can quickly exhaust storage capacity.

## **3. Control Plane Protection**

The control plane is vulnerable to Denial of Service (DoS) attacks and the injection of fraudulent routing data to change the way traffic moves through the network.

- **Control Plane Policing/Protection:** Standard packet forwarding is handled in the device's cache and has near-zero CPU impact. However, packets directed _at_ the router itself (e.g., routing table updates, ICMP, DHCP, management traffic) require heavy CPU processing. Administrators must set specific thresholds (e.g., limiting packets per second for certain protocols) to prevent the CPU from being overwhelmed by flooding.
- **Filtering Dangerous ICMP:** While ICMP is useful, it can be weaponized. To protect the CPU and network topology, devices should filter out:
    - **ICMP Redirects:** Attackers use these to suggest alternate gateways, facilitating Man-in-the-Middle (MitM) attacks.
    - **ICMP Unreachable:** Attackers trigger these messages to map out the internal structure of a network (by addresses that do not respond unreachable).
- **Authenticated Routing Protocols:** To prevent malicious routers from injecting fake routes and hijacking traffic, routing protocols must require cryptographic authentication.

## **4. Data Plane Protection**

Data plane protection ensures packets flow according to strict governance policies without being altered, spoofed, or hijacked. It operates across multiple OSI layers:

**Layer 2 Protection (Switch Configurations):**

- **ARP Security:** Disable gratuitous ARP and Proxy ARP to mitigate MitM spoofing. Enable Dynamic ARP Inspection (DAI) to validate ARP packets.
- **Port Security & Snooping:** Limit the number of MAC addresses allowed per physical port (preventing CAM overflow attacks). Enable DHCP Snooping to ensure responses only come from trusted DHCP servers.
- **IP Source Guard:** Block traffic with abnormal IP-to-MAC associations.

**Layer 3 & 4 Protection (Router/Firewall Configurations):**
Network traffic is restricted using Access Control Lists (ACLs), either Standard (IP only) or Extended (IP + Layer 4 headers). As covered in your earlier firewall lectures, ACLs must be written per-interface to be effective.

- **Anti-Spoofing Rules:** The external/WAN interface must have strict ingress rules blocking any incoming traffic that fraudulently claims to have a source IP address belonging to the internal network.
- **Block Network Mapping:** Utilize ACLs to filter out external UDP or ICMP probes attempting to discover open ports.
- **Principle of Least Privilege (Default Deny):** Only allow traffic explicitly expected on the network. Furthermore, restrict access to critical infrastructure services—rules should dictate that protocols like SSH, NTP, or SNMP only accept connections from a trusted list of administrator IPs.