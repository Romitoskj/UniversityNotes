### I. Privacy Tools Comparison

First it is needed to understand **what kind of privacy** is sought and **against whom** (stakeholders include the government, internet providers, and other users of the same system).

Browsers privacy tools:
1. **Incognito/Private Browsing Mode:** This provides a little privacy, primarily against **other users of the same system** by preventing the browser from storing cookies or history. It offers **zero privacy** against internet profiling, internet providers, or the government.
2. **VPN (Virtual Private Network):** Using a VPN gives you privacy against your **internet provider** because the communication is encrypted and goes through the VPN provider. However, you place significant trust in the VPN provider. It is typically not designed specifically for privacy but for creating the illusion of being on a private network. Privacy against the government is questionable, as authorities could request data from the VPN provider.

### II. Tor (The Onion Router) Protocol

Tor was introduced as a system designed to provide good privacy against the internet provider and the government, without relying on trusting an external service.

#### A. The Onion Circuit and Routing

Tor achieves anonymity by routing packets through a circuit of servers called **relays** (thousands exist globally). A user randomly selects a number of relays (typically five, or three for illustrative purposes).

![](../assets/Pasted%20image%2020251211110006.png)

1. **Circuit Server Roles:**
	
    - **Guard (Entry Point):** The first relay. It knows the user's location (e.g., Rome) and the next relay (e.g., Sydney) but not the final destination (e.g., CNN website).
    - **Middle Relays (e.g., Paris):** Know only the previous relay (e.g., Sydney) and the next relay (e.g., Rio).
    - **Exit Point:** The last relay. It knows the last relay (e.g., Paris) and the final destination (e.g., CNN website in New York City) but nothing about the user's origin.
	
	The user downloads the list of all relays with their IPs and keys and select at least 3 of them  to create the circuit.

2. **Onion Encryption:** The user creates an "onion" by encrypting the packet multiple times, starting with the key for the **last relay (K3)**, then the key for the second-to-last relay (K2), and finally the key for the **Guard (K1)**. The reverse order is because the packet are decrypted from the outer to the inner layer.

3. **Layered Decryption:** As the packet travels, each relay decrypts one layer of the onion using its respective key. 
	
    - The Guard decrypts K1, sees the next layer (K2 encrypted), and forwards it to the middle relay.
    - The middle relay decrypts K2, sees the next layer (K3 encrypted), and forwards it to the exit point.
    - The exit point decrypts K3, sees the original packet, and sends it to the destination website.
	
	Doing so each relay knows only about the previous and next relay, not the content of the message.

#### B. Key Negotiation and Authentication

Since the user and the relays initially share no secret information, they must negotiate shared keys (K1, K2, K3).

##### Diffie-Hellman (DH) Protocol

This protocol is used to establish a shared secret key over an insecure channel.  It relies on the computational difficulty of the **Discrete Logarithm Problem** —it is easy to compute $g^x$ from $x$, but impossible (infeasible) to compute $x$ from $g^x$, were $g \in \mathbb{Z}_p$  is a generator for the field $\mathbb{Z}_p$.

![](../assets/Pasted%20image%2020251211175844.png)

As seen in the image the key K is computed without it being actually shared over the channel.

###### Man-in-the-Middle Attack
The basic DH protocol is weak against a MITM attack, where an adversary intercepts messages and negotiates separate secret keys with both parties, allowing the adversary to decrypt all traffic.

##### Authenticated DH
Tor uses an **authenticated DH protocol** to prevent MITM attacks. The user achieves this by encrypting their DH contribution ($g^X$) using the relay's **public key**. Since only the intended relay can decrypt the message, the user is guaranteed to be talking to the correct server. After that they both have the same key to perform symmetric cryptography.
###### Negotiation Improvement
Modern Tor avoids direct communication when negotiating keys with the middle relay and exit point, instead negotiating them **through the already established circuit** (starting with the guard). This prevents the user's internet provider from knowing which relays were chosen, minimizing the risk of provider/server collusion.

#### C. Attacks and Performance
##### Attacks 

1. The most potent attack against Tor is when an adversary controls both the **Guard and the Exit Point** in a single circuit.
2. **Traffic Analysis:** Even without full decryption, an attacker controlling both points can perform **traffic analysis**, correlating the timing of packets entering the Guard with those exiting the Exit Point to deduce that they belong to the same flow, thereby breaking anonymity.
3. **Mitigation:** To minimize this attack, users select a higher number of relays and choose the Guard and Exit Point from **different continents or autonomous systems**.
##### Performance Metrics

Tor's circuit structure impacts performance:
-  **Latency:** Is **critical/high** because the total latency is the **sum** of the latency across all relays of the circuit, making it potentially very slow.
- **Bandwidth:** Is limited by the **minimum** bandwidth of the slowest relay in the circuit (bottleneck), so it is not necessarily low but it can be.

### III. The Dark Web (Hidden Services)

**Dark Web** refers to hidden services using Tor to ensure that **both the user and the web server are hidden**.

> ***All communications occur using Tor connections***

#### Hidden Service

1. The hidden web server selects "**introductory points**".
	![](../assets/Pasted%20image%2020251211184825.png)
2. Than publishes information about these points and its public key on a **Distributed Hash Table** (DHT).
	![](../assets/Pasted%20image%2020251211184834.png)

#### Connection

1. A user consults the DHT to know the introductory point of the desired service.
	![](../assets/Pasted%20image%2020251211184915.png)
2. Then the user selects a separate Tor node as **meeting point** (rendezvous).
	![](../assets/Pasted%20image%2020251211184924.png)
3. Then communicates information about the meeting point to the hidden service via **introductory points**.
	![](../assets/Pasted%20image%2020251211185025.png)
	![](../assets/Pasted%20image%2020251211185043.png)
4. Now the user and the server can communicate without revealing their locations each other.
	![](../assets/Pasted%20image%2020251211185057.png)![](../assets/Pasted%20image%2020251211185121.png)

Offloading the service to a meeting point instead use it on the introductory points is done to perform **load balancing** and also the rendezvous changes through time while the introductory points remain the same. 