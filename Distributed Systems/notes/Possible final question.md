> [!tip] Tip
*Don't write too much. Complete but not deep: main mechanisms, sometimes examples*

> [!question]- Propose an implementation of a failure detector and discuss its properties (completeness and accuracy) in a synchronous distributed system

> [!question]- Show how to make Paxos live with a perfect failure detector
leader election!

> [!question]- Describe what forks are handled in Bitcoin blockchain, when they happen and how they are handled

> [!question]- Describe how miners get consensus on the next blockchain
tip: spend two words to say what is a miner.

> [!question]- Describe the notion of proof of work in Bitcoin

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

> [!question]- Describe how Tor and the Dark Web work
> Tor: previous question.
>
>The term Dark Web refers 

> [!question]- Briefly describe bit-torrent

> [!question]- BONUS Briefly describe Akamai

> [!question]- Describe how consistent hash and Chord work
what it is, how files are located, how you make lookup, what is a finger table
