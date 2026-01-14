> [!tip] Tip
*Don't write too much. Complete but not deep: main mechanisms, sometimes examples*

> [!question]- Propose an implementation of a failure detector and discuss its properties (completeness and accuracy) in a synchronous distributed system

> [!question]- Show how to make Paxos live with a perfect failure detector
leader election!

> [!question]- Describe what forks are handled in Bitcoin blockchain, when they happen and how they are handled

> [!question]- Describe how miners get consensus on the next blockchain
> 
>
> tip: spend two words to say what is a miner.

> [!question]- Describe the notion of proof of work in Bitcoin
> The proof of work is a mechanism with which miners poof that they performed a vast amount of computational work. Only with this proof a block of transaction is considered valid and the system reaches consensus on these transactions. The actual PoW consists of finding a valid hash for a block of transaction, where valid indicate meet a specific difficult requirement such as produce a hash digest with the last k bit equal to zero. To obtain such digest, miners repeatedly change a variable contained in the block header called nonce

> [!check]- Briefly describe TOR
> Tor is a protocol introduced to provide good privacy for internet browsing without relying on trusting an external service provider. It achieves anonymity by routing packets through a circuit of servers called relays, of which thousands exists globally. 
> 
> A user that wants to initiate a Tor communication randomly select a number of relays (minimum three) to create a circuit. In this circuit the relays have these roles:
> - Guard: the first relay with which the user communicate. It know's the user's location and the next relay, but not the final destination.
> - Middle relay: it can be one or more. Each one knows only the previous and the next relay location.
> - exit point: the last relay that knows the previous one and the final destination of the message.
>
>Since the user and the relays initially do not share secret information, they must negotiate shared keys before sending the actual messages. To do that the authenticated Diffie-Hellman protocol is used: the user sends its contribution to the shared key encrypting it in the relay public key, that guarantees that only the correct server can decrypt the message with its private key and generate the same shared key, with which the two nodes will perform symmetric cryptography. Due to the fact that only the correct relay can decrypt the message, this protocol guarantees authentication mitigating MITM attacks.
> 
> After the circuit connection is established, the user create an "onion" encryption to send a message: the packet is encrypted multiple time starting with the key of the exit point, then with the key of the second-to-last relay and finally with the key of the guard. The reverse order is because  the packet are decrypted from the outer to the inner layer: each relay receive the message, it decrypts it with its own key and than forward the packet (that is still encrypted with the following relay key) to the next packet, that does the same. Eventually the message arrives to the exit point that decrypt the last layer of encryption, sees the original message and sends it to the destination website. Doing so each relay knows only about the previous and next relay, not the content of the message.
>
> This circuit struct impacts the performance, since the latency can be high and the bandwitdth limited. In fact, the total latency it is the sum of the latency across all the link between the nodes of the circuit, potentially making the communications very slow. Regarding the bandwidth instead, it is limited by the minimum bandwidth of the slowest relay in the circuit that can create a bottleneck, so it can be low.
> 
> Concerning the security of the system, the most powerful attack against Tor is the traffic analysis: an adversary can control both the guard and the exit point in a single circuit and, even without full decryption, can breaking anonymity only by correlating the timing of packets entering the guard with those exiting the exit point. To minimize this attack, users can select an high number of relay and choose the guard and exit point from different autonomous systems.

> [!done]- Describe how Tor and the Dark Web work
> Tor: previous question.
>
>The term Dark Web refers to a collection of hidden services on the web. They uses Tor along with the users who access them, which ensure that both the user and web server are hidden.
>
>To expose their content on the dark web a hidden service selects some Tor nodes ad introductory points, then publishes information about them (IP address) and its public key on a distributed hash table. All this communication are performed using Tor protocol.
>
>On the other end, a user that want to use one of these services firstly consult the DHT to know the introductory point of the desired service. Then it selects a separate Tor node as rendezvous point and communicate the address of this node to the hidden service via an introductory point. Eventually the user and the hidden service can communicate without revealing their location each other. Also all these steps are performed using Tor protocol.

> [!question]- Briefly describe bit-torrent

> [!done]- BONUS Briefly describe Akamai
> Akamai is a Content Delivery Network, that is a distributed system that act as a cache of some content, for example images of a website, movies from a streaming platform etc. Akamai is a network of server used to hold copies of this media files that usually takes time to download.
>
> To do that the url of this media in the webpages is replaced with an url containing the akamai domain (e.g. `uniroma1.it.akamai.com/pictures/22.jpg`), so when the url is resolved by a DNS resolver it is sent to an akamai dns server. This server analyzes the request and, based on the origin IP address, it replies with the address of the nearest Akamai server (called point of presence) that holds the requested resource. So the HTTP connection is made with this closer server than the source and it speed up the download of the content.
>
> This mechanism can be also exploited to perform load balancing, routing the user to less-loaded server that are slightly farther away.
>
> Distributed systems like Akamai have the purpose of prioritising availability despite losing a little bit of consistency (maybe the closest server is not update as the origin), and are used for content-driven application like social media, news, streaming services etc. in contrast to application like banking systems or tickets sale that require high consistency.

> [!question]- Describe how consistent hash and Chord work
what it is, how files are located, how you make lookup, what is a finger table

