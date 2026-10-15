Reinforcement Learning 2: The Multi-Armed Bandit Problem — Comprehensive Class Notes

---

1. Introduction to the k-Armed Bandit Problem

The $k$-armed bandit problem serves as the foundational, single-state abstraction of Reinforcement Learning (RL). In a full RL framework, an agent's actions alter environmental states, influencing both immediate outcomes and long-term cumulative returns ($G_t$). By stripping away state transitions and temporal credit assignment, the bandit framework isolates the core operational dilemma of decision-making under uncertainty: the exploration-exploitation tradeoff. In this non-associative setting, an agent receives an immediate numerical reward without altering future environmental conditions ($G_t = R_t$). Isolating value estimation from state dynamics provides a mathematically clean theoretical sandbox to analyze how autonomous systems acquire statistical knowledge while simultaneously optimizing operational performance.

1.1 The Exploration-Exploitation Dilemma

Decision-making agents in RL rely on evaluative feedback rather than instructive feedback:

- **Evaluative Feedback:** Evaluates how good the chosen action was, but provides zero information regarding whether another action would have been superior.
- **Instructive Feedback:** Indicates directly what the optimal action choice was, independently of the action actually selected by the agent.

Because evaluative feedback depends strictly on the action executed, an agent gathers statistical data exclusively for options it actively tests. This creates an operational imperative for **active exploration**:

- **Exploration:** Selecting non-optimal or poorly understood actions to gather statistical information about their underlying, hidden reward distributions.
- **Exploitation:** Selecting the action with the highest current value estimate to maximize immediate operational yield.

These two imperatives are inherently in conflict. Every exploratory action incurs an immediate **opportunity cost**, as the agent sacrifices short-term expected returns on an uncertain quest for long-term knowledge optimization.

**Crucial Conceptual Distinction: The True Goal of a Bandit Agent** A common misconception is that the goal of a bandit algorithm is simply to discover or identify the best arm. In online reinforcement learning, discovering the optimal action is merely a means to an end. The explicit mathematical objective is to **maximize the cumulative expected reward over a specific operational time horizon** (e.g., $T = 1000$ steps). Because rewards are collected _during_ the learning process, every exploratory choice directly impacts total performance. An algorithm that identifies the best arm quickly but incurs catastrophic losses during exploration is fundamentally inferior to one that balances knowledge acquisition with return realization.

1.2 Formal Problem Definition & Properties

Formally, in a $k$-armed bandit problem, a decision-maker repeatedly faces a choice among $k$ discrete options at time steps $t = 1, 2, \dots$. The environment is non-associative (single-state), meaning the background situation never changes. Consequently, there are no state transitions or delayed rewards, and the cumulative return equals the immediate reward ($G_t = R_t$). The agent's goal is to maximize the expected cumulative reward over $T$ steps. Rewards are drawn from unknown probability distributions associated with each action. In a _stationary_ bandit problem, these underlying distributions remain constant over time.

|Notation|Mathematical Definition & Operational Role|
|---|---|
|$\mathcal{A}$|Set of $k$ discrete actions: $\mathcal{A} = \{a_1, a_2, \dots, a_k\}$|
|$A_t$|Action selected at time step $t$ ($A_t \in \mathcal{A}$)|
|$R_t$|Numerical reward received at time step $t$, drawn from stationary distribution $P(R_t \mid A_t)$|
|$q^*(a)$|True (unknown) value of action $a$, defined as expected reward: $\mathbb{E}[R_t \mid A_t = a]$|
|$Q_t(a)$|Estimated value of action $a$ at time step $t$, approximating $q^*(a)$|
|$N_t(a)$|Number of times action $a$ was selected prior to time step $t$|
|$\epsilon$|Exploration probability parameter in $\epsilon$-greedy action selection ($\epsilon \in [0, 1]$)|
|$\alpha$|Step-size parameter / learning rate ($\alpha \in (0, 1]$)|

1.3 Practical Networking Applications

Bandit algorithms are exceptionally well-suited for autonomous edge nodes constrained by tight microampere energy budgets, minimal processing power, and small RAM registers:

1. **Smart-Home RFID Tag Reader:** An autonomous reader queries one of $k$ battery-free tags per time slot. Actions correspond to querying tag $i$; rewards are $+1$ if new data is retrieved and $-1$ if no new data is present.
2. **Dynamic Channel Selection:** An autonomous wireless radio node selects among $k$ discrete frequency channels. Actions are channel choices, yielding a reward of $1$ for a successfully delivered packet and $0$ for a packet collision.
3. **Adaptive MAC Parameters:** Autonomous nodes dynamically adjust Medium Access Control (MAC) parameters (e.g., duty cycles, contention windows, or transmission power levels) treated as discrete arms to maximize overall network throughput per joule.

_Domain Limitation & Real-World Reality:_ Assuming a fixed probability $p$ of retrieving data from an RFID tag is a simplifying abstraction. In real-world sensor networks, a tag's probability of holding new data increases monotonically as a function of the time elapsed since its last query. This introduces explicit _state dependency_ ($S_t$), breaking the single-state bandit assumption and ultimately requiring full Reinforcement Learning frameworks (such as $Q$-learning), which will be addressed in subsequent modules.

Having established the formal definition and networking applications of the multi-armed bandit framework, attention turns to the quantitative estimation methods used to learn action values online.

---

2. Action-Value Estimation Methods

Action-value estimation forms the quantitative backbone of bandit decision-making. Action selection rules rely entirely on these numeric value estimates; a selection strategy can only be as effective as the underlying estimates it consumes.

2.1 Defining Action Values

The true value of an action $a$, denoted $q^*(a)$, is formally defined as the expected (mean) reward given that action $a$ is selected:

$$q^*(a) \doteq \mathbb{E}[R_t \mid A_t = a]$$

In full reinforcement learning, state-value functions are defined as $v_\pi(s) \doteq \mathbb{E}_\pi[G_t \mid S_t = s]$, incorporating discounted streams of future rewards across transitioning states. In the single-state bandit setting, because there are no future states or delayed returns, $G_t = R_t$. Because $q^*(a)$ is unknown to the agent, it must construct an empirical estimate, $Q_t(a)$, based on observed reward sequences.

Consider a medical trial domain where a doctor chooses among $k=3$ treatment options (Red, Yellow, Blue) to optimize patient health, measured as the numeric change in blood pressure:

- **Red Treatment (Bernoulli Distribution):** Yields rewards of $-11$ with probability $0.5$ and $+9$ with probability $0.5$. Its mean true value is $q^*(\text{Red}) = 0.5(-11) + 0.5(9) = -1$.
- **Yellow Treatment (Binomial Distribution):** Yields rewards over $\{-1, 0, 1, 2, 3\}$, symmetric around $1$. Its mean true value is $q^*(\text{Yellow}) = 1$.
- **Blue Treatment (Uniform Distribution):** Yields discrete rewards from $\{1, 3, 5\}$. Its mean true value is $q^*(\text{Blue}) = \frac{1+5}{2} = 3$.

If $q^*(a)$ were known, decision-making would be trivial: choose $\arg\max_a q^*(a)$ (Blue). Because true distribution parameters are unknown, $q^*(a)$ must be estimated from sample outcomes.

2.2 The Sample-Average Method

The natural approach to estimating action values is to compute the arithmetic sample average of the rewards received for each action prior to step $t$:

$$Q_t(a) \doteq \frac{\sum_{i=1}^{t-1} R_i \cdot \mathbb{1}[A_i = a]}{\sum_{i=1}^{t-1} \mathbb{1}[A_i = a]} = \frac{\text{sum of rewards taken for action } a \text{ before } t}{N_t(a)}$$

Here, $\mathbb{1}[\text{predicate}]$ is an indicator function returning $1$ if the predicate is true and $0$ otherwise, isolating time steps where action $a$ was selected. If an action has not yet been selected ($N_t(a) = 0$), $Q_t(a)$ is set to a default initial value (e.g., $Q_1(a) = 0$).

By the Law of Large Numbers, as an action is sampled indefinitely ($N_t(a) \to \infty$), the sample average $Q_t(a)$ converges almost surely to the true expected value $q^*(a)$.

2.3 Empirical Walkthrough & Noisy Estimation

To observe how sample averaging behaves in practice, consider a 10-patient clinical trial where treatment outcomes yield $R=1$ for success and $R=0$ for failure. Treatments are assigned completely at random across 10 steps:

|Treatment|Sampled Rewards (Patient Outcomes)|Total Reward / Selection Count|Sample Average Estimate $Q_t(a)$|
|---|---|---|---|
|**Red**|$1, 0, 0$|$1 / 3$|$Q(\text{Red}) = 0.33$|
|**Yellow**|$1, 0, 1$|$2 / 3$|$Q(\text{Yellow}) = 0.67$|
|**Blue**|$0, 1, 1, 0$|$2 / 4$|$Q(\text{Blue}) = 0.50$|

Each estimate uses only the patients who received that specific treatment. Due to small sample sizes, these estimates are highly noisy and deceptive: Yellow appears superior to Blue simply because Blue suffered an initial random failure.

While batch sample averaging is mathematically straightforward, storing historical reward sequences and re-summing them at every step is computationally prohibitive for edge hardware, necessitating an incremental update mechanism.

---

3. Incremental Implementation & Algorithmic Efficiency

In embedded autonomous networking systems (such as microcontrollers driving battery-free RFID tag readers or wireless sensor nodes), memory and processing speed are severely restricted. Hardware platforms often operate under microampere energy budgets with limited RAM registers. Storing all historical rewards and re-summing them at every step requires memory allocation and processing time that grow linearly with time ($O(t)$). Transitioning to incremental algorithms that run in $O(1)$ constant time and space per step is mandatory for real-world edge deployment.

3.1 Step-by-Step Mathematical Derivation

Let $R_i$ denote the reward received after the $i$-th selection of a given action, and let $Q_n$ denote the value estimate after it has been selected $n-1$ times:

$$Q_n = \frac{1}{n-1} \sum_{i=1}^{n-1} R_i \implies \sum_{i=1}^{n-1} R_i = (n-1) Q_n$$

The update rule for $Q_{n+1}$ after $n$ selections is derived algebraically as follows:

1. **Start with the sample average definition across** $n$ **rewards:** $$Q_{n+1} = \frac{1}{n} \sum_{i=1}^{n} R_i$$
2. **Isolate the** $n$**-th reward** $R_n$ **from the summation:** $$Q_{n+1} = \frac{1}{n} \left( R_n + \sum_{i=1}^{n-1} R_i \right)$$
3. **Multiply and divide the historical summation by** $(n-1)$**:** $$Q_{n+1} = \frac{1}{n} \left( R_n + (n-1) \cdot \frac{1}{n-1} \sum_{i=1}^{n-1} R_i \right)$$
4. **Substitute** $Q_n$ **into the expression:** $$Q_{n+1} = \frac{1}{n} \left( R_n + (n-1) Q_n \right)$$
5. **Expand the product term** $(n-1)Q_n = n Q_n - Q_n$**:** $$Q_{n+1} = \frac{1}{n} \left( R_n + n Q_n - Q_n \right)$$
6. **Separate the** $n Q_n$ **term from the remainder:** $$Q_{n+1} = \frac{1}{n} \cdot n Q_n + \frac{1}{n} [R_n - Q_n]$$
7. **Simplify** $\frac{n}{n} = 1$ **to reach the final incremental update rule:** $$Q_{n+1} = Q_n + \frac{1}{n} [R_n - Q_n]$$

This incremental formula holds even for $n=1$: $Q_2 = Q_1 + 1 \cdot [R_1 - Q_1] = R_1$. The estimate $Q_2$ becomes equal to the first observed reward $R_1$, completely overwriting whatever initial default value $Q_1$ was set. Per action, the node stores only two numeric scalars: estimate $Q_n$ and selection counter $n$.

3.2 Anatomy of the General Update Rule

The derived incremental formula follows a universal algorithmic architecture fundamental to all Reinforcement Learning updates:

$$\text{NewEstimate} \leftarrow \text{OldEstimate} + \text{StepSize} \cdot [\text{Target} - \text{OldEstimate}]$$

- **Target (**$R_n$**):** Represents the newly observed, noisy sample reward toward which the current estimate is adjusted.
- **Error Term (**$[R_n - Q_n]$**):** Quantifies the prediction error between outcome $R_n$ and previous estimate $Q_n$. A positive error raises the estimate; a negative error lowers it.
- **StepSize (**$\frac{1}{n}$ **or** $\alpha$**):** Determines the magnitude of the correction. For sample averages, the step size is $\frac{1}{n}$, which monotonically decreases as sample counter $n$ increases.

3.3 Resource Efficiency & Algorithmic Design

Maintaining only $Q(a)$ and $N(a)$ arrays for $k$ actions yields $O(1)$ spatial complexity and $O(1)$ temporal complexity per execution step. The procedural pseudocode for a complete $\epsilon$-greedy bandit algorithm using incremental sample averages is structured as follows:

```
Initialize, for a = 1 to k:
    Q(a) <- 0
    N(a) <- 0

Loop forever:
    A <- argmax_a Q(a) with probability 1 - epsilon
         random action among all k actions with probability epsilon
    R <- bandit(A)
    N(A) <- N(A) + 1
    Q(A) <- Q(A) + (1 / N(A)) * [R - Q(A)]
```

With an efficient, $O(1)$ constant-memory estimation mechanism established, system performance depends entirely on the action selection rule used to balance exploration against exploitation.

---

4. Action Selection Rules & Exploration Strategies

Action selection rules govern how value estimates $Q_t(a)$ translate into actual choices $A_t$, balancing immediate yield realization against continuous knowledge acquisition.

4.1 Greedy Action Selection

A purely greedy selection strategy selects the action with the highest estimated value at step $t$:

$$A_t \doteq \arg\max_a Q_t(a)$$

Ties are broken arbitrarily (e.g., uniformly at random). Purely greedy selection exploits current knowledge to maximize short-term yield, but performs zero active exploration. If an optimal action yields an initial low reward due to random noise, its value estimate drops, causing a greedy agent to avoid it permanently.

_Example of Suboptimal Greedy Lock-In:_ Suppose true action values are $q^*(\text{Red}) = 0.25$, $q^*(\text{Yellow}) = 0.50$, and $q^*(\text{Blue}) = 0.75$. If Blue's first trial fails ($R=0 \implies Q(\text{Blue})=0$) while Yellow gets a lucky initial success ($R=1 \implies Q(\text{Yellow})=0.75$), the greedy rule will select Yellow continuously from step 2 onward. Blue's estimate is never corrected, locking the agent into a suboptimal action indefinitely.

4.2 $\epsilon$-Greedy Selection Mechanism

The $\epsilon$-greedy rule prevents premature lock-in by mixing high-probability exploitation with occasional random exploration:

- With probability $1 - \epsilon$, select the greedy action: $\arg\max_a Q_t(a)$.
- With probability $\epsilon$, select an action uniformly at random from all $k$ actions.

**Mathematical Probability Analysis:** The exploratory choice can also select the greedy action by chance. For a bandit with $k$ actions, the total probability of selecting the greedy action is:

$$P(A_t = \text{Greedy Action}) = (1 - \epsilon) + \frac{\epsilon}{k}$$

**Concept Check 4.1: Selection Probability in 2-Armed Bandits** _Question:_ In a 2-armed bandit ($k=2$) using $\epsilon$-greedy selection with $\epsilon = 0.5$, what is the exact probability that the greedy action is selected?

_Answer:_ It is **not** $0.5$ ($50\%$). Applying the formal probability formula: $$P(A_t = \text{Greedy}) = (1 - 0.5) + \frac{0.5}{2} = 0.5 + 0.25 = 0.75 \quad (75\%)$$ The agent exploits with $50\%$ probability, but the remaining $50\%$ random exploration slot splits evenly across both arms ($25\%$ each). Thus, the greedy arm is chosen in $75\%$ of all time steps.

**Asymptotic Performance Guarantees:** Because every action maintains a minimum non-zero selection probability ($\frac{\epsilon}{k}$), $N_t(a) \to \infty$ as $t \to \infty$ for all $a$. Consequently, $Q_t(a) \to q^*(a)$ for all actions, guaranteeing convergence to true values.

4.3 Deducing Exploration: Sutton & Barto Exercise 2.2 Walkthrough

To master the interaction between $Q$-value states, selection rules, and observed rewards, consider a 4-armed bandit ($k=4$, actions $\{1, 2, 3, 4\}$) utilizing $\epsilon$-greedy selection, sample-average updates, and $Q_1(a) = 0$ for all $a$. We analyze an empirical 5-step sequence to determine on which steps exploration _definitely_ or _possibly_ occurred:

|Time Step ($t$)|Selected Action ($A_t$)|Observed Reward ($R_t$)|$Q_t$ Values Prior to Action Selection|Greedy Set $\arg\max_a Q_t(a)$|Classification|
|---|---|---|---|---|---|
|$t = 1$|$A_1 = 1$|$R_1 = 1$|$Q_1 = [0, 0, 0, 0]$|$\{1, 2, 3, 4\}$ (All tied)|**Possibly Exploratory**|
|$t = 2$|$A_2 = 2$|$R_2 = 1$|$Q_2 = [1, 0, 0, 0]$|$\{1\}$|**Definitely Exploratory**|
|$t = 3$|$A_3 = 2$|$R_3 = 2$|$Q_3 = [1, 1, 0, 0]$|$\{1, 2\}$ (Tied)|**Possibly Exploratory**|
|$t = 4$|$A_4 = 2$|$R_4 = 2$|$Q_4 = [1, 1.5, 0, 0]$|$\{2\}$|**Possibly Exploitative**|
|$t = 5$|$A_5 = 3$|$R_5 = 0$|$Q_5 = [1, 1.67, 0, 0]$|$\{2\}$|**Definitely Exploratory**|

_Detailed Analysis:_

- **Step** $t=1$**:** All initial estimates are tied at $0$. Any choice is in the greedy set. Action $1$ could have been selected via exploitation (tie-breaking) or exploration.
- **Step** $t=2$**:** After $R_1=1$, $Q_2(1)=1$ while $Q_2(2)=Q_2(3)=Q_2(4)=0$. The unique greedy action is $1$. The agent chose $A_2=2$. This selection violates the greedy rule and was **definitely exploratory**.
- **Step** $t=3$**:** After $R_2=1$, $Q_3(2)=1$. The greedy set is tied between $\{1, 2\}$. The agent chose $A_3=2$, which belonged to the greedy set. Hence, it could be exploitation or exploration landing on action $2$.
- **Step** $t=4$**:** After $R_3=2$, $Q_4(2) = \frac{1+2}{2} = 1.5$. The unique greedy action is $2$. The agent selected $A_4=2$. This choice matches the greedy choice, so it is **possibly exploitative** (or exploratory landing on $2$).
- **Step** $t=5$**:** After $R_4=2$, $Q_5(2) = \frac{1+2+2}{3} = 1.67$. The unique greedy choice is $2$. The agent selected $A_5=3$. This violates the greedy rule and was **definitely exploratory**.

4.4 Comparative Evaluation of Parameter Choice

|Exploration Parameter|Operational Mechanics|Performance Advantages & Disadvantages|
|---|---|---|
|$\epsilon = 0$ **(Pure Greedy)**|Never explores; always selects $A_t = \arg\max_a Q_t(a)$.|Maximizes immediate yield early, but risks permanent lock-in on suboptimal actions.|
|$\epsilon = 0.01$ **(Low Exploration)**|Explores in $1\%$ of steps; exploits in $99\%$.|Slowly discovers optimal actions, but yields superior long-term cumulative return by minimizing exploratory overhead.|
|$\epsilon = 0.1$ **(Moderate Exploration)**|Explores in $10\%$ of steps; exploits in $90\%$.|Rapidly identifies optimal actions, but continuously sacrifices $\sim 10\%$ of performance to unnecessary random exploration.|
|$\epsilon = 1.0$ **(Pure Random)**|Selects actions uniformly at random every step.|Accurately estimates all action values $q^*(a)$, but never exploits learned knowledge to optimize cumulative reward.|

**Decaying** $\epsilon$ **Heuristic:** A common hybrid strategy starts with a high exploration rate (e.g., $\epsilon = 1.0$) to gather data quickly, gradually decaying $\epsilon$ over time toward zero to exploit established estimates later ("explore early, exploit later").

4.5 Optimistic Initial Values

Optimistic initialization forces early exploration without requiring explicit random choices. By initializing estimates well above achievable reward bounds (e.g., setting $Q_1(a) = 2$ when maximum possible reward is $1$), the agent experiences initial disappointment regardless of which action it takes.

_Mechanism:_ Suppose $Q_1(a) = 2$ for all $a$, and updates use step size $\alpha = 0.5$. On step 1, the agent chooses Red greedily and receives $R=1$. Its estimate drops to $Q_2(\text{Red}) = 2 + 0.5(1 - 2) = 1.5$. On step 2, Yellow and Blue still have estimates of $2.0$, making them appear strictly superior. The agent is forced greedily to rotate through untried actions until all estimates decay toward realistic values.

**Mathematical Synthesis: Initial Values vs. Step-Size Rules** A critical mathematical interaction exists between initial estimates $Q_1(a)$ and the choice of step-size parameter: Under standard sample averaging ($\frac{1}{n}$), setting an optimistic $Q_1(a)$ is erased after a single trial! On step 1 ($n=1$), the update yields $Q_2(a) = Q_1(a) + 1 \cdot [R_1 - Q_1(a)] = R_1$. The initial estimate $Q_1(a)$ is completely overwritten by $R_1$. Consequently, **optimistic initial values do not drive sustained exploration under sample averages**. They strictly require a constant step size $\alpha$, where historical memory of $Q_1(a)$ decays exponentially via $(1-\alpha)^n Q_1(a)$, preserving the optimistic bias across multiple initial pulls.

_Limitations:_ Optimistic initialization drives exploration only at the beginning of learning. Once initial values decay, exploration ceases completely, making it ineffective for non-stationary environments. Furthermore, it requires prior domain knowledge of reward bounds to set an appropriately "optimistic" $Q_1$.

4.6 Step-by-Step Execution Trace (Smart-Home RFID Reader)

Consider an 8-query execution trace using an $\epsilon$-greedy reader ($\epsilon = 0.2$) querying 3 tags ($T_1, T_2, T_3$). Rewards are $+1$ (new data) or $-1$ (no new data). All $Q_1(a) = 0$, updated via sample averages. (True hidden data probabilities: $p_1=0.5, p_2=0.2, p_3=0.8 \implies q^*(a) = 2p - 1 \implies q^*(T_1)=0, q^*(T_2)=-0.6, q^*(T_3)=+0.6$).

|$t$|Prior Estimates $Q(T_1), Q(T_2), Q(T_3)$|Greedy Set|$\epsilon$-Dice Outcome|Queried Tag|Reward $R$|Updated Value of Queried Tag|
|---|---|---|---|---|---|---|
|**1**|$0, 0, 0$|$\{T_1, T_2, T_3\}$|Exploit (Tie $\to T_1$)|$T_1$|$+1$|$Q(T_1) = 1/1 = \mathbf{1.0}$|
|**2**|$1.0, 0, 0$|$\{T_1\}$|Exploit|$T_1$|$+1$|$Q(T_1) = (1+1)/2 = \mathbf{1.0}$|
|**3**|$1.0, 0, 0$|$\{T_1\}$|**Explore (**$\to T_3$**)**|$T_3$|$-1$|$Q(T_3) = -1/1 = \mathbf{-1.0}$|
|**4**|$1.0, 0, -1.0$|$\{T_1\}$|Exploit|$T_1$|$-1$|$Q(T_1) = (1+1-1)/3 = \mathbf{0.33}$|
|**5**|$0.33, 0, -1.0$|$\{T_1\}$|**Explore (**$\to T_3$**)**|$T_3$|$+1$|$Q(T_3) = (-1+1)/2 = \mathbf{0.0}$|
|**6**|$0.33, 0, 0$|$\{T_1\}$|Exploit|$T_1$|$-1$|$Q(T_1) = (1+1-1-1)/4 = \mathbf{0.0}$|
|**7**|$0, 0, 0$|$\{T_1, T_2, T_3\}$|Exploit (Tie $\to T_3$)|$T_3$|$+1$|$Q(T_3) = (-1+1+1)/3 = \mathbf{0.33}$|
|**8**|$0, 0, 0.33$|$\{T_3\}$|Exploit|$T_3$|$+1$|$Q(T_3) = (-1+1+1+1)/4 = \mathbf{0.50}$|

_Trace Analysis:_ At step 3, optimal tag $T_3$ suffered an initial random failure ($R=-1$), dropping $Q(T_3)$ to $-1.0$. A purely greedy reader would have abandoned $T_3$ forever. Exploration at step 5 gave $T_3$ a second chance, raising $Q(T_3)$ back to $0$. By step 7, $T_3$ entered the greedy set, reaching $Q(T_3) = 0.50$ by step 8 (approaching true value $q^*(T_3) = +0.6$). Tag $T_2$ was never queried, leaving $Q(T_2) = 0$.

While stationary action selection rules ensure convergence when reward distributions are static, real-world wireless environments exhibit dynamic, non-stationary behavior that requires specialized tracking mechanisms.

---

5. Tracking Non-Stationary Environments

In modern autonomous networking, reward distributions fluctuate dynamically over time. Physical wireless phenomena such as shadow fading, temporal channel interference, node mobility, and varying application traffic patterns constantly alter link performance. Stationary assumptions break down entirely under these non-stationary operational conditions.

5.1 Breakdown of Sample-Average Methods

In non-stationary environments, historical rewards reflect past physical states that no longer exist. Sample-average estimation uses step size $\frac{1}{n}$. As trial counter $n$ increases, $\frac{1}{n} \to 0$. The update rule effectively freezes, rendering $Q_t(a)$ insensitive to dynamic changes in the environment.

5.2 Constant Step-Size Tracking ($\alpha$)

To track shifting parameters in non-stationary environments, sample-average step sizes ($\frac{1}{n}$) are replaced with a constant step-size parameter $\alpha \in (0, 1]$:

$$Q_{n+1} \doteq Q_n + \alpha [R_n - Q_n]$$

Expanding this update equation recursively demonstrates how historical rewards are weighted over time:

1. **First algebraic expansion:** $$Q_{n+1} = \alpha R_n + (1-\alpha) Q_n$$
2. **Substitute** $Q_n = \alpha R_{n-1} + (1-\alpha) Q_{n-1}$**:** $$Q_{n+1} = \alpha R_n + (1-\alpha) [\alpha R_{n-1} + (1-\alpha) Q_{n-1}]$$ $$Q_{n+1} = \alpha R_n + \alpha(1-\alpha) R_{n-1} + (1-\alpha)^2 Q_{n-1}$$
3. **Substitute** $Q_{n-1} = \alpha R_{n-2} + (1-\alpha) Q_{n-2}$**:** $$Q_{n+1} = \alpha R_n + \alpha(1-\alpha) R_{n-1} + \alpha(1-\alpha)^2 R_{n-2} + (1-\alpha)^3 Q_{n-2}$$
4. **Generalizing recursively back to initial estimate** $Q_1$**:** $$Q_{n+1} = (1-\alpha)^n Q_1 + \sum_{i=1}^{n} \alpha (1-\alpha)^{n-i} R_i$$

**Properties of Exponential Recency-Weighted Averaging:**

1. **Sum of Weights:** The weights sum strictly to $1$: $$(1-\alpha)^n + \sum_{i=1}^{n} \alpha(1-\alpha)^{n-i} = 1$$
2. **Exponential Memory Decay:** The weight assigned to historical reward $R_i$ is $\alpha(1-\alpha)^{n-i}$. Because $(1-\alpha) < 1$, older rewards decay exponentially at rate $(1-\alpha)$ per step. Recent rewards maintain significantly higher weights, enabling the value estimate to continuously track ongoing environmental shifts.

5.3 Comparative Exercise Walkthrough (Sutton & Barto Exercise 2.3 / Exercise 3)

Consider a non-stationary shift where Blue treatment effectiveness increases during winter months. After 20 patients, $Q(\text{Blue}) = 0.5$. The next 4 patients are all successfully cured ($R=1$). We compare sample averaging ($\frac{1}{n}$) against constant step size ($\alpha = 0.2$):

|Patient Outcome Step|Applicable Step Size|Sample Average Formula ($\frac{1}{n}$)|Sample Average Result|Constant Step Size Formula ($\alpha = 0.2$)|Constant Step Size Result|
|---|---|---|---|---|---|
|**21st Patient**|$\frac{1}{21} \text{ vs } 0.2$|$0.5 + \frac{1}{21}(1 - 0.5)$|**0.52**|$0.5 + 0.2(1 - 0.5)$|**0.60**|
|**22nd Patient**|$\frac{1}{22} \text{ vs } 0.2$|$0.52 + \frac{1}{22}(1 - 0.52)$|**0.55**|$0.60 + 0.2(1 - 0.60)$|**0.68**|
|**23rd Patient**|$\frac{1}{23} \text{ vs } 0.2$|$0.55 + \frac{1}{23}(1 - 0.55)$|**0.57**|$0.68 + 0.2(1 - 0.68)$|**0.74**|
|**24th Patient**|$\frac{1}{24} \text{ vs } 0.2$|$0.57 + \frac{1}{24}(1 - 0.57)$|**0.58**|$0.74 + 0.2(1 - 0.74)$|**0.80**|

_Analysis:_ Under sample averaging, 20 outdated historical records dominate the estimate; reaching $Q = 0.80$ would require 30 consecutive successes. With constant step size $\alpha = 0.2$, past memory decays by factor $0.8$ per step, enabling the node to track the non-stationary shift from $0.50$ to $0.80$ in just 4 steps.

5.4 Synthesis: The Limits of Bandits & Transition to Full RL

While multi-armed bandits effectively solve single-state decision problems under uncertainty, real-world autonomous networks frequently violate the non-associative assumption. As highlighted in smart-home RFID tag harvesting, radio channel selection, and adaptive MAC protocols, immediate reward distributions often depend explicitly on system history (e.g., elapsed time since last poll, buffer occupancy, or node mobility states). When action choices alter environmental states ($S_t \to S_{t+1}$) and influence future reward possibilities, single-state bandit formulations become insufficient. Modern autonomous networking must therefore extend these value-estimation and exploration principles to full Markov Decision Processes (MDPs) and temporal-difference algorithms such as $Q$-learning.

In summary, multi-armed bandits balance exploration and exploitation in $O(1)$ memory by pairing incremental value updates with stochastic selection rules. Selecting constant step sizes ($\alpha$) allows these resource-efficient agents to continuously adapt to dynamic, non-stationary network environments.