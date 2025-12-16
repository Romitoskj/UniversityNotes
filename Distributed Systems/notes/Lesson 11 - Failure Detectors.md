### 1. The Function and Impossibility of Perfect Detectors

A Failure Detector is a **local software module** residing within a process $p$ that can be queried to determine the fate of another process $q$, specifically whether $q$ has crashed or stalled. If the process $p$ dies, its local detector also dies.

If one assumes a **magically perfect** failure detector—one that is **always right**—it would be simple to solve leader election, even in an asynchronous system. Processes could query the detector, disregard processes identified as dead, and elect the living process with the minimum ID.

However, achieving such a perfect module is **not possible** in an asynchronous system. If it were possible, it could be used to make Paxos both safe and live, thereby solving consensus, which is known to be impossible in asynchronous systems due to the FLP result.

### 2. Formal Properties of Failure Detectors

Failure detectors are classified based on two formal properties:
#### A. Completeness

Completeness means that **if a process has crashed, the detector can see it** (i.e., detect the crash and respond that the process has crashed to queries).

Given that $\sigma$ is a run, $crashed(\sigma$) the set of process crashed during the run, $up(\sigma)$ the set of live process during the run and $D_q(t, \sigma)$ the set of process believed to be dead at time $t$ in the run $\sigma$ by process $q$, completeness can be:

- **Strong Completeness:** If a process $p$ crashes, **every** other up process $q$ will eventually suspect $p$ as crashed:
$$

\forall\sigma, \quad \forall p\in crashed(\sigma),\quad  \forall q\in up(\sigma) \quad  \exists t : \forall t'>t \quad  p\in D_q(t', \sigma)
$$
- **Weak Completeness:** If a process $p$ crashes, there exists **at least one** up process $q$ that will eventually suspect $p$ as crashed:
$$

\forall\sigma, \quad \forall p\in crashed(\sigma),\quad  \exists q\in up(\sigma) \quad  \exists t : \forall t'>t \quad  p\in D_q(t', \sigma)
$$

It is possible to transform any failure detector with **weak completeness** into one with **strong completeness**. This is achieved by having processes broadcast the list of dead processes they suspect and taking the **union** of these lists.

#### B. Accuracy

Accuracy means that **if the module tells a process that Q is dead, it must be true** (if a process is alive, the detector see it alive). Accuracy pertains to processes that are alive.

- **Strong Accuracy:** If two processes ($p$ and $q$) are up (alive), $q$ does not think $p$ is dead:
$$
\forall\sigma\quad\forall t \quad\forall p,q\in up(t, \sigma): p \notin D_q (t, \sigma)
$$
- **Weak Accuracy:** There exists at least one process $p$ that remains up and is **never suspected** to be dead by any other up process $q$:
$$
\forall\sigma\quad\forall t \quad\exists p\in up(\sigma) \quad\forall q\in up(t, \sigma): p \notin D_q (t, \sigma)
$$
- **Eventual Strong Accuracy ($\diamond$S):** The detector may be inaccurate initially, but from some **fixed time $t$ onward**, the strong accuracy property holds:
$$
\forall\sigma\quad\forall t'>t \quad\forall p,q\in up(t', \sigma): p \notin D_q (t', \sigma)
$$
- **Eventual Weak Accuracy ($\diamond$W):** The detector may be inaccurate initially, but from some **fixed time $t$ onward**, the weak accuracy property holds:
$$
\forall\sigma\quad\forall t'>t \quad\exists p\in up(\sigma) \quad\forall q\in up(t', \sigma): p \notin D_q (t', \sigma)
$$

> [!Info] Failure Detector Taxonomy
>
> 
> | **C\A** | **S**    | **W** | **$\diamond$S** | **$\diamond$W** |
| ------- | -------- | ----- | --------------- | --------------- |
| **S**   | P        | S     | $\diamond$P     | $\diamond$S     |
| **W**   | $\theta$ | W     | $\theta$        | $\diamond$W     |
Where:
> - P = perfect
> - S = strong
> - W = weak
> - $\diamond$ = eventual
> 
> A detector with **strong completeness and strong accuracy** is called **Perfect (P)**. The "magical" detector described earlier is even stronger than P because it implies completeness holds at all times, not just eventually.

### 3. Example of Failure detector

A practical example can be a failure detector that works by having a process $p$ send a "ping" message to $q$ repeatedly (every $\Delta t$) and suspecting $q$ to be dead if no reply is received within a specific time delay ($\Delta d$).

- **Completeness:** In any system, if $q$ crashes, it cannot reply, so $p$ will suspect it. Thus, the detector has **strong completeness**.
- **Accuracy (Asynchronous System):** In an asynchronous system (where message delays are unbounded), the detector lacks of accuracy because a slow message reply might lead the detector to **falsely suspect** a process is dead when it is merely running slowly.
- **Accuracy (Synchronous System):** If the system is **synchronous** (messages deliver within a known time bound), the delay ($\Delta d$) can be set large enough (e.g., larger than the round trip time) to guarantee **strong accuracy**.

### 4. Leader Election and Paxos Liveness

Using a leader election protocol based on a detector with **strong completeness and eventual strong accuracy** (like the one described before in a partially synchronous system) can lead to eventual consensus.

While this detector can eventually ensure only one leader exists, during the initial "messy" phase (before the detector becomes accurate), two problems may temporarily arise:

1. **Two Leaders:** A live process $p_1$ might be wrongly suspected of being dead due to the initial lack of accuracy, causing another process $p_2$ to declare itself the leader.
2. **Zero Leaders:** When an actual leader dies, there is a delay (waiting for the $\Delta d$ timeout) before other processes detect the crash (a completeness issue), resulting in a temporary period with no leader.

Despite these temporary issues, the eventual strong accuracy guarantees that eventually, the system converges on a single leader, which is sufficient to **make Paxos live**. However, a detector with these properties (strong completeness and eventual strong accuracy) is only possible if the system is synchronous or partially synchronous; it is **not possible** in a strictly asynchronous system.