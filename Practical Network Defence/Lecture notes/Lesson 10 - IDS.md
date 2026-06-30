
## **1. Introduction to IDS and IPS**

An **Intrusion Detection System (IDS)** aims to detect the presence of intruders before serious damage is done, such as preventing legitimate users from accessing the system, revealing confidential information, or using the host as a stepping stone for further attacks.

To detect these threats, an IDS must perform **Deep Packet Inspection (DPI)**, exploring the inner payload of packets rather than just looking at headers like a standard firewall.

- **IDS (Passive):** Operates "out of band" (e.g., using a network tap or mirrored SPAN port). It detects anomalies and raises alarms for an analyst to investigate, but does not interfere with the traffic directly.
- **IPS (Active):** Intrusion Prevention Systems operate "in line" (in band) with the traffic. When an intrusion is detected, it actively reacts by dropping connections, blocking access, or dynamically modifying firewall rules. The industry standard term (NIST) combining both concepts is IDP.
  
### 1.1 IDS Architecture

- **Network Sensors:** Detect and send data to the system.
- **Central Monitoring System:** A server that processes and analyzes data sent from the sensors.
- **Database and Storage Component:** Repository for event information.
  ![](../../Pasted%20image%2020260630173206.png)

## **2. Alarms and Accuracy Metrics**

Because an IDS evaluates ambiguous behavior, its alarms are categorized into four outcomes:

- **True Positive:** A real attack occurred, and the IDS correctly raised an alarm.
- **True Negative:** Normal traffic occurred, and the IDS correctly stayed silent.
- **False Positive:** Harmless, legitimate behavior was incorrectly classified as an attack, wasting a security analyst's time.
- **False Negative:** A real attack occurred, but the IDS failed to detect it, leaving the network vulnerable. A successful IDS must balance detecting a substantial percentage of intrusions while keeping false positives at a manageable level.

## **3. Types of Intrusion Detection Systems**

- **Host-based (HIDS):** Monitors a single specific system (typically critical servers). It looks for unauthorized access, configuration changes, or anomalous process executions. A prime example of what HIDS monitors is ransomware activity, which can be detected when a process rapidly opens, encrypts, and overwrites an unusually high number of files.
- **Endpoint Detection and Response (EDR):** The modern evolution of HIDS. It integrates host firewalls, anti-malware, and process-level monitoring, frequently leveraging cloud-based global threat intelligence and machine learning to proactively hunt and respond to threats.
- **Network-based (NIDS):** Operates in promiscuous mode (usually connected to a switch's mirrored/SPAN Switch Port ANalizer port) to analyze network, transport, and application protocol activity protecting many hosts. All the traffic generated within all the ports of the switches are replicated on the mirrored port where the NIDS is placed. A single NIDS can monitor multiple network segments simultaneously, though it struggles with encrypted traffic.
- **Wireless (WIDS):** Analyzes wireless networking protocols (e.g., signal power, MAC sequence numbers).
- **File Integrity Monitors:** Tools like Tripwire that monitor cryptographic hashes of key system configuration files to detect unauthorized changes.

## **4. Detection Approaches: Behavior vs. Signature**

### 4.1 **Behavior-based (Anomaly Detection)**

This approach defines what "normal" behavior looks like and raises an alarm for any significant deviation.

![](images/Pasted%20image%2020260630155159.png)

- **Feature Extraction:** The system must be trained using data mining to extract relevant features. A classic example is the Lee & Stolfo feature set, classifying features into **Intrinsic** (about particular connection e.g., connection lifetime), **Traffic** (statistical information about connections e.g., percentage of SYN errors), and **Content** (application-related statistic e.g., failed login attempts).
- **Measurement:** Anomalies are measured using distance formulas (Hamming, Mahalanobis, Kolmogorov) or probability models (Markov chains, Neural Networks).
- **Pros & Cons:** It is excellent for catching zero-day exploits, but it suffers from high false-positive rates because dynamically defining "normal" in a changing network is extremely difficult.

### 4.2 **Signature-based (Misuse Detection)**

This approach defines the specific characteristics (signatures) of known abnormal activities or attacks. It uses pattern matching against a massive database of known signatures to spot malicious payloads. To extract these signatures invariant characteristics of known attacks must be used (body of known viruses and worms, port numbers of application with known buffer overflows, etc.)

![](images/Pasted%20image%2020260630155218.png)

- **Evasion Techniques:** Attackers actively try to bypass signature detection. They might use **TCP Insertion or TTL attacks**, where they send overlapping fragments or packets with intentionally short Time-to-Live (TTL) or bad checksums. The NIDS might reassemble the payload incorrectly and miss the signature (e.g., the string "USER root"), while the target host drops the garbage packets and executes the hidden attack.
- **Honeypots:** To gather new signatures, researchers use honeypots, namely deceptive security resources (single host or network) with no legitimate purpose. Any interaction with a honeypot is inherently malicious, allowing administrators to safely observe the attacker's methods and extract new signatures.
- **Pros & Cons:** It is fast, highly accurate for known threats, and produces very few false positives. However, it cannot detect new (zero-day) attacks, and it struggles with polymorphic/metamorphic malware that constantly changes its signature.

_Note: Modern robust defense requires combining both approaches to minimize both false positives and false negatives._

## **5. Prominent IDS/IPS Tools**

- **Snort:** The most famous open-source NIDS/NIPS (now owned by Cisco). It uses a highly expressive rule-driven language. Its architecture consists of a packet decoder, preprocessors (for defragmentation and stream reassembly), a detection engine, and an alerting system.
    - _Rule Syntax:_ Rules contain a **Header** (Action, Protocol, Source IP/Port, Direction `->`, Dest IP/Port) and **Options** enclosed in parentheses (content matching, offset, depth, PCRE regular expressions, and flags).
- **Suricata:** A modern, high-performance alternative developed by the OISF. It overcomes Snort's historical limitations by natively supporting multi-threading and GPU acceleration. It includes deep application-layer decoding (HTTP, TLS, DNS, etc.) and integrates the Lua scripting language to handle complex conditions that a simple legacy Snort rule cannot match.
- **Zeek (formerly Bro):** Distinct from Snort/Suricata, Zeek uses a Turing-complete scripting language (Bro Script, structurally similar to C++) instead of traditional signatures. It acts as a powerful network security monitor, but its extensive deep-packet inspection capabilities make it highly resource-intensive and challenging to maintain.
- **Fail2ban:** A host-based intrusion prevention framework designed to stop brute-force attacks. It is essentially a log parser that reads service logs (e.g., SSH, Apache). When a defined filter detects too many failed attempts (a `jail` configuration), it executes an action to dynamically add an `iptables` firewall rule to ban the offending IP address.
    - _Limitations:_ Because it relies on reading logs, it has a reaction time delay. Administrators must disable syslog buffering to make it faster. It is also vulnerable to local DoS attacks if an insider uses the `logger` command to forge fake syslog entries.