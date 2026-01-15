> [!tip] Tip
*Don't write too much. Complete but not deep: main mechanisms, sometimes examples*

> [!question]- Propose an implementation of a failure detector and discuss its properties (completeness and accuracy) in a synchronous distributed system
> An implementation of a failure detector can be a FD that works by sending a ping message to all the other processes in the systems repeatedly, with an interval of time $\Delta t$. A process $p$ will suspect another process $q$ to be dead if no reply is received within a specific time delay $\Delta d$.
> 
> This implementation in a synchronous system guarantees both strong completeness and strong accuracy:
> - if $q$ crashes it cannot reply to a ping sent by $p$ so the latter will suspect $p$ as dead. Thus, the detector has strong completeness.
> - if the delay $\Delta d$ is set large enough, larger than the maximum round trip time of the system, we can be certain that if a process reply is not arrived after waiting $\Delta d$ it will never arrive, so the process is surely dead and the detector has strong accuracy.
>   
> In an asynchronous system this detector will lack of accuracy because message delays are unbounded and it's not possible to be certain 

> [!done]- Show how to make Paxos live with a perfect failure detector
> If the existence of a perfect failure detector is assumed, it would be simple to solve leader election even in an asynchronous system: process could query the detector, discard dead process and elect the living process with the minimum ID, achieving consensus on the same leader. Since Paxos with a leader that is the only one allowed to propose value, resolving the leader election problem make the protocol live. 
> 
> Moreover, making Paxos both safe and live will solve consensus in an asynchronous system will contradict the FLP theorem. However, achieving a perfect FD is not possible in an asynchronous system.

> [!done]- Describe what forks are handled in Bitcoin blockchain, when they happen and how they are handled
> When two miners simultaneously mine the next block, the blockchain forks in two different branches. After a block is mined, it is broadcasted with the gossip protocol so roughly half of the total miner will work on one of the two branches, while the other half on the other one. Eventually one branch will grow faster than the other and the shortest one will be discarded, not before copying its transaction on the survived branch. Due to forks, a branch is actually confirmed to be the right one after other six blocks are added to it (approximately one hour after its creation).

> [!DONE]- Describe how miners get consensus on the next blockchain 
> BECAME Who are miners in bitcoin? (look at the following question to know how they get consensus)
> Miners are bitcoin nodes that perform a vast amount of computational work by repeatedly change the nonce of a block to obtain a valid hash (the proof of work) and reach consensus on a block of transactions. Moreover, they receive a sum of newly minted bitcoin as an incentive to perform this process.

> [!done]- Describe the notion of proof of work in Bitcoin
> The proof of work is a mechanism with which miners poof that they performed a vast amount of computational work. Only with this proof a block of transaction is considered valid and the system reaches consensus on these transactions. The actual PoW consists of finding a valid hash for a block of transaction, where valid indicate meet a specific difficult requirement such as produce a hash digest with the last k bit equal to zero. To obtain such digest, miners repeatedly change a variable contained in the block header called nonce until and re-hash the block until a valid digest is found. Also the first transaction in every block grants the miner newly minted Bitcoin, as an incentive for mining. This reward halves approximately every four years converging toward a total of 21 million bitcoin.
>
> Moreover this mechanism guarantee security to the system since mine a block is an hard task (the probability of computing a correct hash is $\frac{1}{2^k}$ that for $k=30$ is $\frac{1}{1B}$). Hence, if an attacker wants to modify an old transaction to get consensus on different transactions he would need to re-mine all the previous blocks and this is an infeasible task.
 
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

> [!done]- Describe how consistent hash and Chord work
> Chord is a Distributed Hash Table protocol used to locate resources in peer-to-peer systems such as files in IPFS. In this protocol files are identified by the hash of their content called CID (Content Identifier). Also the IP addresses of the servers in the system are hashed and, along with the files CIDs, are treated as numbers in a circular space or ring where they are projected based on their hash. For example, if the hash algorithm used is SHA-256 the identifiers range from $0$ to $2^{256}-1$.
> 
> *Here draw the circle with files on the left and servers to the right projected on the circle*
> 
> Based on these projections a file is stored on the next server encountered while moving clockwise around the ring from the file CID position, i.e. the first server with greater hash. Doing so file and server are distributed uniformly, because of the hash that acts like a random function generating totally different hash for two input with small differences. That guarantees load balancing in the system.
> 
> Moreover, to avoid contacting almost all the servers one by one to find a file, Chord uses a routing mechanism called finger table: each server maintains a table of "fingers" pointing to other servers at exponential distances ($p + 2^0, p + 2^1, p + 2^2, \dots$ , where $p$ is the hash of the IP address of the server containing the table). The table doesn't contains all the servers but only does at this distances. When a user request a file to a server it can be stored by the server itself or not. If it is not present the server check the finger table to get the IP of the server with the largest preceding hash than the CID of the file (the first encountered going counterclockwise in the ring) and then relay the request to it, that will do the same thing. This process continues until a node discovers that the file’s CID falls between itself and its immediate successor. At that point, the node knows for certain that its successor is the one responsible for the file and directs the request there to complete the search. This lookup result in a cost of $O(\log n)$, since the distance to the target files is reduced by half at every request "hop".
> 
> Finally, when a new server joins the system, it must contact its successor to transfer the files for which it is now responsible. It then builds its own finger table, which takes approximately $O(\log^2 n)$ time. And if a server leaves abruptly without its files being pinned elsewhere, those files are lost to the system.
