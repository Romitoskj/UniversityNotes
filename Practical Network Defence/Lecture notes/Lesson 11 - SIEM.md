## **1. Introduction to SIEM**

**SIEM (Security Information and Event Management)** is a cybersecurity approach that centralizes the monitoring and analysis of a network. It is not a single application, but rather a combination of building blocks designed to help organizations detect and mitigate threats by providing a comprehensive picture of the system's health.

It combines two distinct disciplines:

- **SIM (Security Information Management):** Focuses on collecting, storing, and analyzing log data, and alerting responsible individuals to threats.
- **SEM (Security Event Management):** Focuses on real-time system monitoring, establishing event correlations, and notifying network administrators of active issues.

While there is no single established standard or protocol for SIEM, most systems function by automatically collecting information from distributed sources (either via installed agents or agentless mechanisms), storing it centrally, correlating events, and producing alerts or compliance reports.

## **2. Log Management vs. State Data**

Logs are the fundamental building blocks of a SIEM; without them, security management is impossible. However, it is important to distinguish between event data and state data:

- **Event Data (Logs):** A specific string or record telling you exactly _what_ happened and _when_. Because logs come from vastly different sources (syslog of servers and computers, firewalls, domain controllers, IDSs, proxies), the SIEM must first **parse and normalize** these heterogeneous strings into a common, structured syntax.
- **State Data:** This provides a picture of the overall system at a given moment (e.g., current configurations, active users, open processes). As the professor noted, "state" is mathematically derived by combining events—for instance, matching a process "start" log and ruling out a "terminate" log to determine what is currently running.

## **3. The SIEM Architecture Stack**

The operational flow of a SIEM can be visualized as a hierarchical stack:

1. **Event Layer:** The raw data collected from all nodes on the network.
2. **Normalization Layer:** The SIEM extracts meaningful information (e.g., timestamp, event name, process, source IP) from the raw logs so that different formats are standardized.
3. **Correlation Layer:** The engine uses rules or Machine Learning to monitor the normalized logs for logical sequences, relationships, or patterns.
4. **Reporting Layer:** The presentation of findings for human review or compliance validation.

**Data Enrichment (Supporting Data):** To make the correlation layer more effective, SIEMs use "supporting data" to enrich the logs. For example, an isolated IP address in a log might not mean much on its own. However, by enriching it with geolocation or asset management data, the SIEM can recognize that the IP belongs to a known malicious actor (e.g., an APT group in Russia) and immediately escalate the alert's priority.

## **4. Event Correlation and Active Response**

The core selling point of a SIEM is its **Correlation Engine**. Instead of evaluating a single event in a vacuum, it links non-homogeneous events from different hosts to uncover attacks that are invisible to individual systems. For example, the SIEM might correlate a strange interface error with a concurrent SSH login on that host to identify an ongoing intrusion.

When a threat is correlated, the SIEM can trigger an **Active Response**. This means moving beyond passive detection to automate protective procedures. Examples include dynamically adding IP filters to a router's ACL, disabling a compromised user account, pushing an OS update to a vulnerable endpoint, or adjusting a misconfigured host firewall.

## **5. IT Regulatory Compliance and Forensics**

A major function of a SIEM is proving organizational diligence.

- **The "Jury" Concept:** As the professor explained, compliance asks the fundamental question: _If you had to defend your actions to a jury, could you prove you used best practices to protect the data?_.
- **Auditing & Reporting:** A SIEM automatically monitors requirements like the frequency of password changes or the successful installation of OS patches. It generates the required reports (e.g., for GDPR) to prove the organization wasn't negligent.
- **Log Integrity:** To be legally defensible for digital forensics, the log server itself must be highly reliable. Best practices require using reliable transport (TCP), encrypted storage, and digital signatures to guarantee the authenticity and integrity of the logs.

## **6. Threat Intelligence Integration**

Modern SIEMs incorporate open-source or commercial **Threat Intelligence feeds**. These feeds provide a continuously updated stream of intelligence regarding active cyber campaigns, newly discovered malware signatures, or malicious IPs.

By importing this data, organizations shift their security posture from **reactive** to **proactive**. Instead of waiting for an alarm to trigger, security officers use the intelligence feeds to proactively hunt through their SIEM data for specific indicators of compromise (IOCs) to see if their network is already targeted.

## **7. Operational Interfaces**

Administrators interact with the SIEM through two primary interfaces:

- **Dashboards and Maps (Pull of Information):** A highly organized, graphical representation of events. They allow administrators to log in, identify trends, and understand the system's health at a glance.
- **Alerts (Push of Information):** Automated notifications sent to administrators when a specific correlation rule triggers. They do not require human diligence to notice that something important is happening.

## **8. Prominent SIEM Solutions**

The course highlights several major players in the SIEM market:

- **Splunk:** A leading commercial solution known for its highly expressive query syntax and advanced search capabilities.
- **Elastic Stack (Logstash / Kibana):** Commonly used for log collection (Logstash) and powerful dashboard visualization (Kibana).
- **Google Chronicle:** A cloud-based solution where an organization sends all its logs to Google's infrastructure for processing, eliminating the need to maintain on-premise hardware.
- **Others:** Syslog-ng and Graylog, which are often used for robust, open-source logging and analysis.