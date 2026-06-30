## **1. Introduction to Proxies**

A proxy acts as an intermediary relay system that takes the role of one of the two parties in a network communication. Originally conceived at CERN in 1994, an application-level proxy was designed to make a firewall safely permeable for internal users without creating a direct security hole for attackers. Namely: *one single host handling requests from several users*.

Unlike VPNs, which provide a secure encrypted tunnel for device-level routing, a proxy explicitly handles and evaluates requests on behalf of a client or server. Operating at higher layers, proxies can perform highly intelligent filtering (e.g., blocking specific SQL injections) but introduce significant processing overhead because they must intercept, reassemble, and inspect application data.

## **2. Forward Proxies**

A **Forward Proxy** operates strictly on behalf of the client. Instead of connecting directly to the internet, the client sends an absolute-form HTTP or FTP request to the proxy, which then forwards the request to the final destination.

![](images/Pasted%20image%2020260630100110.png)

- **Key Benefits:**
    - **Authentication & Content Filtering:** The proxy can enforce Authentication, Authorization, and Auditing (AAA). It acts as a content filter (e.g., in schools) to enforce blacklists, scan for malware, or block executable downloads.
    - **Caching:** Proxies store retrieved documents locally to save bandwidth on subsequent requests, relying on HTTP headers like `If-Modified-Since` to verify if the cache is up-to-date.
    - **Anonymization:** A proxy shields the client's privacy because the destination server only sees the proxy's IP address, rather than the originating client's IP. Typically used for accessing geo-restricted content.
- **HTTP Tunneling (`CONNECT` Method):** For non-standard HTTP and FTP requests or TLS/SSL traffic (or any protocol that use TCP), the proxy uses the `HTTP CONNECT` method. The proxy establishes a two-way TCP connection on behalf of the client and then blindly forwards the TCP stream, allowing protocols like HTTPS to traverse the proxy. Often limited to port 443 (HTTPS) only or not supported.
  ![](../../Pasted%20image%2020260630104528.png)

## **3. Reverse Proxies**

A **Reverse Proxy** operates on behalf of the server. It receives requests from the outside world as if it were the actual server, and then routes those requests to the appropriate internal origin servers.

- **Internal Server Protection:** Because external users never establish a direct connection to the internal servers, reverse proxies naturally defend against Denial of Service (DoS) attacks. They are also used to protect vulnerable legacy software or IoT devices that cannot be natively secured or updated.
- **Application Layer Firewall (WAF):** A reverse proxy can act as a Web Application Firewall (e.g., ModSecurity for Apache). It deeply inspects HTTP traffic to block malformed communication, unwanted protocols, or application-level attacks like Cross-Site Scripting (XSS) and file inclusions.
- **Other Functions:** Load balancing, caching static content, data compression, and centralizing access control.

## **4. Proxies and the HTTPS Challenge**

Because roughly 95% of modern web traffic is encrypted with SSL/TLS, an application proxy cannot natively read the payload to perform inspections.

- **SSL Forward Proxy (SSL Bump):** To inspect encrypted traffic, the proxy must perform a controlled Man-in-the-Middle (MitM) attack. The proxy intercepts the client's request, dynamically generates a forged certificate mimicking the requested server, and presents it to the client. The proxy decrypts the traffic for inspection, applies security policies, and then re-encrypts it before sending it to the destination.
- **The SNI Dilemma:** During a TLS handshake, the proxy needs to know which certificate to forge before the encrypted HTTP request is sent. It relies on **Server Name Indication (SNI)**, an extension where the client requests the hostname in cleartext during the initial `Client Hello`. Because SNI is unencrypted, it allows proxies (and eavesdroppers) to filter domains. An experimental protocol, Encrypted SNI (ESNI), aims to close this privacy loophole.
- **TLS Acceleration & Offloading:** Public key asymmetric encryption is highly processor-intensive. Reverse proxies alleviate this bottleneck using dedicated hardware.
    - _SSL Termination:_ The proxy decrypts the traffic and forwards it internally in cleartext, allowing Intrusion Detection Systems (IDS) to inspect it.
    - _SSL Forwarding:_ The proxy decrypts, inspects for malicious code, and securely re-encrypts the traffic before sending it internally.

## **5. Circuit-Level Gateways (SOCKS Proxies)**

SOCKS proxies (e.g., SOCKS5) operate at the session layer as a "shim" between the application and transport layers. They are generic TCP/UDP relays that do not inspect application payloads.

- **Versatility:** They support IPv6, multiple authentication mechanisms, and are natively integrated into tools like SSH and Tor.
- **Practical Use Case:** SOCKS proxies are excellent for bypassing IP-based authentication. For example, establishing a SOCKS tunnel into a university server allows a user to download academic papers from home, as the academic database will only see the university's approved proxy IP address.

## **6. Transparent (Intercepting) Proxies**

While forward proxies require client-side configuration, **Transparent Proxies** are designed to be invisible to the user. The client genuinely believes it is talking to the final internet gateway.

- **Mechanics:** The proxy's TCP/IP stack forcefully intercepts packets destined for external servers and pretends to be the destination.
- **Policy-Based Routing (PBR):** Because traditional network routing is purely destination-driven, implementing a transparent proxy requires PBR. Administrators must configure the router (e.g., using `iptables` port forwarding rules) to specifically detour all traffic matching a certain port (like TCP 80) into the proxy machine before it leaves the network.

## **7. ICAP (Internet Content Adaptation Protocol)**

ICAP (RFC-3507) is a lightweight protocol that allows proxies to execute "remote procedure calls" on HTTP messages.

Instead of heavily burdening the proxy server with intense processing, ICAP offloads the payload to dedicated "surrogate" servers. These surrogate servers can perform expensive operations like real-time virus scanning, translating languages, formatting content for mobile devices, or dynamically inserting advertisements before passing the adapted message back to the proxy.