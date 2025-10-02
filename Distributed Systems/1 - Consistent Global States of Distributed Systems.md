## Introduction
A **distributed system** is a kind of computer system consisting of a set of interconnected processes whose communications between them happen only by message exchange.

A large class of problem in distributed computing (monitoring, detection, load balancing) can be cast as executing some notification or reaction when the state of the system satisfies certain conditions. Thus, the ability to construct a **global state** and evaluate a predicate over such a state constitutes the core of solutions to many problems in distributed computing.

The **global state** of a distributed system is the _union of the states_ of the individual processes, which do not shares memory but communicate solely through exchange of messages. 

>The fundamental problem is to ensure that a global state constructed in this manner is meaningful.

---
## Asynchronous Distributed Systems
Distributed system is composed of:
- a collection of **processes** $p_1, p_2, ..., p_n$
- **communication channels** between pair of them for message exchange
Two models:
- **Asynchronous** (realistic for actual systems):
	- no bound on processes speeds and on message delay
	- synchronized to local clock
	- communications only mechanism for synchronization
- **Synchronous**:
	- processes speeds and message delays bounded

---
## Distributed Computations
#### Events
The execution of a distributed program by a collection of processes. Each process executes a sequence of ***events***, that can be internal or involve communications.

A communication event can be:
- $send(m)$ (enqueuing message for transmission to destination process)
- $receive(m)$ (dequeing message at destination process)
where  $m$ is the message identifier.
#### History
- *local history* of process $p_i$ is a sequence of event $h_i = e_i^1 e_i^2 ... e_i^n$, 
- history of the process $p_i$ containing the first $k$ events $h_i^k = e_i^1 e_i^2 ... e_i^k$
- *global history* is a set $H=h_1 \cup ... \cup h_n$
#### Happened-before
Global history does not specify timing between events, they can only be ordered based on *"cause-and-effect"* relationship: two events are considered to occur in a certain order only if the first affect the outcome of the second, either because they are from the same process or they are from different processes and they correspond to the exchange of a message.

The binary relation *happened-before* $\rightarrow$ is defined over events such that:
1. $e_i^k, e_i^\ell \in h_i \wedge k<\ell \Rightarrow e_i^k \rightarrow e_i^\ell$ 
2. $e_i = send(m) \wedge e_j = receive(m) \Rightarrow e_i \rightarrow e_j$ 
3. $e \rightarrow e' \wedge e' \rightarrow e'' \Rightarrow e \rightarrow e''$ (transitivity)
It denotes that if $e \rightarrow e'$ the outcome of $e'$ *may* have been influenced by $e$. 

It is possible that for some event $e$ and $e'$ neither $e \rightarrow e'$ nor $e' \rightarrow e$ so they are *concurrent* $e||e'$.

#### Space-time diagram
![[figure_1.png]]
If a path can be traced from one event to the other left to right along the horizontal lines and in the sense of the arrow the are related, otherwise they are concurrent.
For example in the [[#Space-time diagram|figure]] $e_2^1 \rightarrow e_3^6$ but $e_2^2 || e_3^6$ .

---
## Global States, Cuts and Runs
### Local State
$\sigma_i^k$ or $a_i^k$ denote the *local* state of process $p_i$ after the event $e_i^k$ is executed, so $\sigma_i^0$ is the initial state. The local state of a process may include information such as the values of local variables and the sequence of messages sent and received.
#### Global State
The *global* state of a distributed computation is an n-tuple of local states $\Sigma=(\sigma_1, \sigma_2, ..., \sigma_n)$ one for each process
#### Cut
![[figure_2.png]]
a *cut* of a distributed computation is a subset $C$ of its global history $H$ and contains an initial prefix of each of the local histories.