Comprehensive Study Notes: Autonomous Networking & Reinforcement Learning — Smarter Exploration and Action-Selection Benchmarking

---

1. Foundations of Non-Stationary Tracking & Step-Size Dynamics

In reinforcement learning (RL), action-value estimation forms the mathematical foundation of decision-making under uncertainty. In a strictly stationary environment—where reward probability distributions remain fixed over time—taking simple sample averages (where every past reward receives an equal weight of $\frac{1}{n}$) guarantees convergence to the true expected value $\arg\max_a q^*(a)$ via the Law of Large Numbers. However, autonomous networking environments are inherently dynamic and non-stationary: wireless link capacities fluctuate, signal-to-noise ratios drift due to mobility, and background cross-traffic varies continuously. In such settings, relying on equal-weighted sample averages leads to learning stagnation because early historical observations exert an identical influence to the most recent network telemetry. To maintain continuous adaptability, an RL agent must transition from equal-weighted sample averages to recency-weighted update rules governed by a constant step-size parameter $\alpha \in (0, 1]$, effectively discounting historical data exponentially while prioritizing real-time observations.

Mathematical Derivation & Weight Analysis

The general incremental update rule for an action-value estimate using a constant step-size parameter $\alpha$ is defined as:

$$Q_{n+1} = Q_n + \alpha \big[ R_n - Q_n \big]$$

where $Q_n$ is the estimated value of the action prior to its $n$-th selection, $R_n$ is the actual reward received at step $n$, and $\alpha \in (0, 1]$ represents the constant step-size parameter.

This update equation can be algebraically rearranged into a convex combination of the newly observed reward $R_n$ and the prior estimate $Q_n$:

$$Q_{n+1} = \alpha R_n + (1 - \alpha) Q_n$$

To analyze how historical observations accumulate over time, we expand $Q_{n+1}$ recursively back to the initial value estimate $Q_1$:

$$\begin{aligned} Q_{n+1} &= \alpha R_n + (1 - \alpha) \Big[ \alpha R_{n-1} + (1 - \alpha) Q_{n-1} \Big] \\ &= \alpha R_n + \alpha(1 - \alpha) R_{n-1} + (1 - \alpha)^2 Q_{n-1} \\ &= \alpha R_n + \alpha(1 - \alpha) R_{n-1} + \alpha(1 - \alpha)^2 R_{n-2} + \dots + \alpha(1 - \alpha)^{n-i} R_i + \dots + (1 - \alpha)^n Q_1 \end{aligned} $$

Expressing this fully expanded sequence in summation notation yields:

$$Q_{n+1} = (1 - \alpha)^n Q_1 + \sum_{i=1}^{n} \alpha (1 - \alpha)^{n-i} R_i$$

Weight Analysis & Exponential Decay Mechanics

The algebraic expansion reveals two fundamental structural properties of constant step-size tracking:

1. **Sum of Weights Identity:** The total weight assigned to the initial value $Q_1$ and all $n$ past observed rewards $R_i$ sums exactly to $1$:

$$(1 - \alpha)^n + \sum_{i=1}^{n} \alpha (1 - \alpha)^{n-i} = 1$$

1. **Exponential Discounting Mechanics:** The weight assigned to reward observation $R_i$ is given by $\alpha(1 - \alpha)^{n-i}$. Because $\alpha \in (0, 1]$, the factor $(1 - \alpha)$ is strictly bounded within $[0, 1)$. Consequently, as the temporal lag $n - i$ increases, the weight factor $(1 - \alpha)^{n-i}$ decays exponentially toward zero.

Direct Pedagogical Link: Mapping General Indexing to Concrete Steps

To understand this decay concretely, consider an agent evaluating an action at step $n = 4$ to compute $Q_5$. Using the general summation index term $\alpha(1 - \alpha)^{n-i} R_i$:

- For the first observation ($i = 1$), the temporal lag is $n - i = 4 - 1 = 3$. The resulting weight assigned to $R_1$ is $\alpha(1 - \alpha)^3$. For $\alpha = 0.5$, this yields $0.5(0.5)^3 = 0.0625$.
- For the most recent observation ($i = 4$), the temporal lag is $n - i = 4 - 4 = 0$. The weight assigned to $R_4$ is $\alpha(1 - \alpha)^0 = \alpha = 0.5$.

The "So What?" Layer: Reactivity Under Non-Stationarity

This exponential decaying factor $(1 - \alpha)^{n-i}$ ensures that the agent's value estimates prioritize recency over historical longevity. While early rewards $R_1, R_2$ and initial baseline biases $Q_1$ exert influence during initial steps, their weight decays rapidly as $n$ grows. If a network link suddenly degrades at step $t$, a sample-average algorithm ($\frac{1}{n}$) would require an enormous volume of low-reward samples to overpower an extensive history of high-reward samples. Conversely, a constant step-size updater exponentially discounts pre-shift observations, allowing the value estimate $Q_{n+1}$ to track environmental changes dynamically and converge toward the new underlying distribution.

Having formalized the mathematical mechanics of value estimation updates, we now evaluate how these algorithmic choices perform empirically within standardized statistical benchmarks.

---

2. Empirical Methodology: Benchmarking on the 10-Armed Testbed

Evaluating reinforcement learning strategies requires isolation from confounding external variables. Because real-world and simulated environments involve continuous stochastic fluctuations, evaluating an agent's performance across a single trial yields noisy and misleading results. An inferior algorithm might achieve high cumulative reward purely through a sequence of fortunate random draws. To establish true statistical convergence and compare action-selection algorithms fairly, researchers utilize standardized multi-armed bandit testbeds executed over rigorous multi-run statistical protocols.

Deconstruction of the 10-Armed Testbed Architecture

The **10-armed testbed** provides a standard benchmark for evaluating action-selection policies under controlled stochastic conditions.

```
       True Value Distribution: q*(a) ~ N(0, 1)
                     |
  +------------------+------------------+
  |                  |                  |
Action 1          Action 2   ...    Action 10
q*(1) = 0.2       q*(2) = -0.8      q*(10) = 1.3
  |                  |                  |
  v                  v                  v
R ~ N(q*(1), 1)   R ~ N(q*(2), 1)    R ~ N(q*(10), 1)
```

The underlying architecture operates under three foundational definitions:

1. **Action Generation:** For each problem instance, $k = 10$ distinct actions are instantiated. The true expected reward value $q^*(a)$ for each action $a \in \{1, \dots, 10\}$ is independently sampled from a standard normal distribution with mean $0$ and variance $1$:

$$q^*(a) \sim \mathcal{N}(0, 1)$$

1. **Stochastic Reward Distribution:** When the agent selects action $a$ at step $t$, the environment generates an empirical reward $R_t$ sampled from a normal distribution centered at that action's true value $q^*(a)$ with unit variance:

$$R_t \sim \mathcal{N}(q^*(a), 1)$$

Because the reward standard deviation ($\sigma = 1$) is on par with the spread of true values across arms ($\sigma = 1$), an individual sample lies within $\pm 1$ of $q^*(a)$. Individual rewards are highly noisy, making it impossible for an agent to deduce $q^*(a)$ accurately from a single interaction.

1. **Maximum Performance Ceiling:** Across standard normal samples of $10$ actions, the expected maximum value across generated problems is:

$$\mathbb{E}\left[\max_a q^*(a)\right] \approx 1.54$$

This threshold ($1.54$) represents the absolute theoretical ceiling for optimal expected per-step reward performance.

```
Reward Distribution per Action (Violin Plot Architecture)
---------------------------------------------------------------------
 +4 |                                          ( )
 +3 |                     ( )                  / \
 +2 |         ( )         / \                  | |  <- Action 8 is optimal
 +1 |         / \         | |         ( )      \ /     q*(8) = 1.3
  0 |--------|---|--------|-|---------/ \-------v------------------- Mean
 -1 |        \ /          \ /         | |
 -2 |                     ( )         \ /
 -3 |                                 ( )
---------------------------------------------------------------------
Action:       a_1         a_2  ...    a_7      a_8     a_9    a_10
```

The Three Sources of Randomness

To evaluate testbed performance accurately, three distinct sources of stochastic variance must be isolated:

- **Problem Generation Noise:** Every run draws a unique set of true values $\{q^*(1), q^*(2), \dots, q^*(10)\}$. Some problem instances are inherently easy (e.g., one arm is exceptionally superior), while others are highly ambiguous (e.g., multiple arms have nearly identical means).
- **Reward Generation Noise:** Even when an action is selected repeatedly, its generated reward fluctuates around $q^*(a)$ according to $\mathcal{N}(q^*(a), 1)$.
- **Agent Operational Noise:** Exploration rules (e.g., randomized $\epsilon$-greedy decisions) and tie-breaking procedures (selecting randomly among equal $Q$-estimates) introduce stochastic variance into action trajectories.

Experimental Protocol: 2,000 Runs $\times$ 1,000 Steps

Because a single trial yields noisy performance metrics, benchmark protocols execute a multi-trial evaluation suite:

```
=====================================================================
                    10-ARMED TESTBED EXPERIMENTAL LOOP
=====================================================================
Outer Loop: 2,000 Independent Runs (Generates unique q*(a) distributions)
 │
 ├── Reset Value Estimates: Q(a) <- 0, Counts: N(a) <- 0
 │
 └── Inner Loop: t = 1 to 1000 Steps
      │
      ├── Select Action A_t using active policy (Greedy, ε-Greedy, UCB)
      ├── Execute A_t, receive Reward R_t ~ N(q*(A_t), 1)
      ├── Update estimate Q(A_t) and action count N(A_t)
      └── Log step statistics: Reward R_t and Optimal Action Flag [A_t == a*]
=====================================================================
Total Benchmark Computation: 2,000 Runs × 1,000 Steps = 2,000,000 Steps
=====================================================================
```

By averaging metrics across all 2,000 runs at each time step $t$, random variance shrinks proportionally to $1/\sqrt{\text{runs}}$. This transforms erratic single-run trajectories into smooth curves tracking two core metrics over time:

1. **Average Reward** at step $t$.
2. **% Optimal Action** selection accuracy at step $t$ ($\text{fraction of runs where } A_t = a^*$).

With a standardized benchmarking protocol established, we can systematically analyze classical action-selection policies.

---

3. Comparative Evaluation of Classical Action-Selection Policies

Action-selection policies must balance exploiting current estimates to maximize immediate payoff against exploring uncertain choices to discover higher-yielding actions. Classical policies apply distinct strategies to navigate this trade-off: pure exploitation (Greedy), persistent uniform random exploration ($\epsilon$-Greedy), and initial forced exploration driven by optimistic initial values.

Policy Analysis & Performance Metrics

Greedy Policy ($\epsilon = 0$)

The greedy policy purely exploits current action estimates, choosing $A_t = \arg\max_a Q_t(a)$ at every step. Ties are broken randomly.

- **Performance Trajectory:** The greedy algorithm exhibits a rapid initial rise within the first 10–20 steps, quickly plateauing at an average reward of $\approx 1.03$.
- **Suboptimal Trapping:** The greedy policy finds the optimal action $a^*$ in only $\approx 33\%$ to $36\%$ of benchmark runs. In the remaining two-thirds of trials, initial reward noise yields a deceptive lower sample estimate for $a^*$, or an inferior arm produces an initial sample above its true mean. Because a greedy agent never explores non-greedy options, these early bad impressions are never corrected, trapping the agent on suboptimal arms permanently.

```
       Average Reward Curve Comparison
       -------------------------------------------------------------
       1.54 | ................................... Best Possible
            |
        1.4 |                        /-------------- ε = 0.1
            |                       / /------------- ε = 0.01
        1.2 |                      / /
            |  /------------------/ /
        1.0 |==--------------------/---------------- Greedy (ε = 0)
            +-----------------------------------------------
            0        200        400        600  ...  1000 Steps
```

$\epsilon$-Greedy Policies ($\epsilon = 0.1$ vs. $\epsilon = 0.01$)

The $\epsilon$-greedy policy balances exploitation with continuous random exploration. With probability $1 - \epsilon$, the agent selects the greedy action; with probability $\epsilon$, it selects an action uniformly at random from all $k$ arms.

- **Speed vs. Asymptotic Trade-Off:**
    - $\epsilon = 0.1$ **(High Exploration):** Learns rapidly. It identifies superior arms early, reaching an average reward of $\approx 1.35$ and picking the optimal action $\approx 80\%$ of the time by step 1000. However, its performance caps out early due to continuous random choices.
    - $\epsilon = 0.01$ **(Low Exploration):** Learns much slower. By step 1000, it achieves an optimal action selection rate of only $\approx 60\%$ and an average reward of $\approx 1.30$. However, its trajectory continues climbing steadily beyond step 1000.
- **Theoretical Optimal Action Selection Ceiling:** When value estimates converge to true values ($Q(a) = q^*(a)$), an $\epsilon$-greedy agent selects the true optimal action $a^*$ in two distinct scenarios: during a greedy selection step, or when a random exploration step happens to select $a^*$.

Given $k$ total actions and exploration probability $\epsilon$, the maximum achievable theoretical optimal action probability is:

$$\text{Ceiling} = (1 - \epsilon) + \frac{\epsilon}{k}$$

Applying this theoretical identity to the testbed ($k = 10$):

- **For** $\epsilon = 0.1$**:**

$$\text{Ceiling} = (1 - 0.1) + \frac{0.1}{10} = 0.90 + 0.01 = 0.91 \quad (91\%)$$

The agent wastes $9\%$ of its operational life taking suboptimal random steps, limiting its long-term performance.

- **For** $\epsilon = 0.01$**:**

$$\text{Ceiling} = (1 - 0.01) + \frac{0.01}{10} = 0.99 + 0.001 = 0.991 \quad (99.1\%)$$

Given sufficient time horizon, $\epsilon = 0.01$ achieves near-perfect asymptotic efficiency, significantly outperforming $\epsilon = 0.1$.

Expected Reward in Long-Run Limits

Given optimal true value $q^*(a^*)\approx 1.54$ and non-optimal mean average $\approx 0$:

- **For** $\epsilon = 0.1$**:**

$$\mathbb{E}[R] = 0.91 \times 1.54 + 0.09 \times 0 \approx 1.40$$

- **For** $\epsilon = 0.01$**:**

$$\mathbb{E}[R] = 0.991 \times 1.54 + 0.009 \times 0 \approx 1.526$$

Optimistic Initial Values ($Q_1(a) = 5, \alpha = 0.1$)

Rather than relying on explicit random action choices ($\epsilon > 0$), initial value estimates can be set to an artificially high baseline (e.g., $Q_1(a) = 5$ for all $a$, where true values average $0$).

```
       % Optimal Action Dynamics: Optimistic vs. Realized
       -------------------------------------------------------------
       100% |
        80% |                               /------- Optimistic (Q1=5, ε=0)
        60% |                              /
        40% |  /\ Step 11 Spike           /--------- Realistic (Q1=0, ε=0.1)
        20% | /  \                       /
          0 |/----\---------------------/--------------------
            0    11  100   200   300   400  ...  1000 Steps
```

- **Mechanism — "Exploration Pushed by Disappointment":** Because initial estimates ($Q_1 = 5$) vastly exceed any realistic reward sampled from $\mathcal{N}(q^*(a), 1)$, whichever action the agent chooses yields an empirical reward lower than its current estimate ($R_t < Q_t(a)$). Under constant step-size updates ($Q_{n+1} = Q_n + \alpha[R_n - Q_n]$), this negative error forces the action estimate to drop. As a result, the unchosen arms retain higher estimates, prompting a greedy policy to sample a new unchosen action on the next step.
- **Stationary vs. Non-Stationary Breakdown:**
    - **In Stationary Environments:** Optimistic initialization performs exceptionally well ($\approx 86\%$ optimal action accuracy at step 1000 versus $\approx 76\%$ for $\epsilon=0.1$), because initial exploration naturally dies out as value estimates converge to true means, eliminating persistent exploration overhead.
    - **In Non-Stationary Environments:** Optimistic initialization fails completely. Because "the start happens only once," initial optimistic values decay permanently. If the environment changes later in the run, the agent retains low $Q$-estimates for non-selected arms and lacks a mechanism to re-explore them.

---

Real-World & Safety-Critical Risk Perspective

In theoretical benchmarks, setting aggressive hyperparameters like $Q_1 = 5$ merely results in transient performance dips during early exploration steps. However, in physical, medical, or network-engineering deployments, uncalibrated optimism carries severe real-world risks:

- **Medical Decision Systems:** In clinical trial optimization, setting artificially optimistic expectations on untested drug treatments forces the system to administer potentially toxic or ineffective therapies repeatedly to patient populations until their estimates decay down to realistic baseline levels.
- **Autonomous Network Routing:** In safety-critical network infrastructure, an overly optimistic agent deployed on core routing switches will blindly route production traffic down degraded or saturated links repeatedly to confirm that their throughput is low. This degrades network Quality of Service (QoS), breaches Service Level Agreements (SLAs), and risks widespread packet drop cascades.

Consequently, pure optimism cannot be blindly deployed in live autonomous environments without explicit safety bounds or uncertainty-aware exploration constraints.

---

While classical strategies force exploration through random noise or initial disappointment, they lack a principled mechanism to track uncertainty continuously. This necessitates smarter action selection methods.

---

4. Smarter Exploration: Upper Confidence Bound (UCB) Action Selection

While $\epsilon$-greedy policies explore continuously, they do so blindly: a random step is just as likely to re-evaluate a known poor action as it is to test a highly uncertain one. Optimistic initialization drives early systematic exploration, but lacks a mechanism to re-explore later. To achieve higher sample efficiency, an agent must select actions based on uncertainty, quantifying an action's potential value rather than relying solely on its sample mean.

```
                   UNCERTAINTY & CONFIDENCE INTERVALS
  
  Possible Value q*(a) Distribution Curves:
  
           Action 3 (High N_t -> Narrow Interval, Certain)
                |  /\
                | /  \
                |/    \
  --------------+------+----------------------------------- Q_t(a3) = 2.0
  
           Action 1 (Low N_t -> Wide Interval, Highly Uncertain)
          /------------\
         /              \
        /                \
  -----+------------------+-------------------------------- Q_t(a1) = 1.0
  
  Principle: Select Action 1 because its upper bound potential reaches higher!
```

Theoretical & Mathematical Anatomy of UCB

Upper Confidence Bound (UCB) action selection operationalizes the principle of **"optimism in the face of uncertainty."** The agent quantifies an upper confidence limit for each action's true value and deterministically selects the arm with the highest upper bound:

$$A_t = \underset{a}{\arg\max} \left[ Q_t(a) + c \sqrt{\frac{\ln t}{N_t(a)}} \right]$$

|Formula Component|Core Function|Structural & Mathematical Role|
|---|---|---|
|$Q_t(a)$|Exploitation Component|Current sample-average estimate of action $a$. Measures expected immediate reward.|
|$c$|Exploration Hyperparameter|Constant parameter ($c > 0$) controlling the relative weight assigned to uncertainty bounds. Setting $c = 0$ reduces the policy to pure greedy selection.|
|$\ln t$|Natural Logarithm of Time|Natural logarithm of total elapsed steps $t$. Grows monotonically but slowly, ensuring that the uncertainty bonus grows continuously for unchosen arms so no action is abandoned indefinitely.|
|$N_t(a)$|Action Selection Count|The number of times action $a$ has been chosen prior to step $t$. Appears in the denominator under a square root to model shrinking variance.|
|$\sqrt{\frac{\ln t}{N_t(a)}}$|Variance Uncertainty Bound|Represents a upper confidence variance term. The bonus shrinks asymptotically proportional to $\frac{1}{\sqrt{N_t(a)}}$.|

Quantitative Behavior of the Uncertainty Bonus

To illustrate how uncertainty bounds evolve, consider a system at step $t = 10,000$, where $\ln(10,000) \approx 9.21$. The table below shows how the bonus term scales relative to parameter $c$ as an action is sampled more frequently:

|Action Count $N_t(a)$|Computation Factor $\sqrt{9.21 / N_t(a)}$|Effective Bonus Scale|Interpretation & Action Priority|
|---|---|---|---|
|$N_t(a) = 1$|$\sqrt{9.21 / 1} \approx 3.034$|$3.03 \cdot c$|**Very High Uncertainty:** Action chosen very few times; receives massive priority boost.|
|$N_t(a) = 10$|$\sqrt{9.21 / 10} \approx 0.960$|$0.96 \cdot c$|**Uncertain:** Moderate exploration incentive.|
|$N_t(a) = 100$|$\sqrt{9.21 / 100} \approx 0.303$|$0.303 \cdot c$|**Moderately Known:** Bonus reduced substantially relative to $N=1$.|
|$N_t(a) = 5,000$|$\sqrt{9.21 / 5000} \approx 0.043$|$0.043 \cdot c$|**Well Known:** Uncertainty nearly zero; choice governed almost purely by $Q_t(a)$.|

Key Proportional Insight: Square-Root Decay Scaling

Comparing $N = 5,000$ to $N = 100$ represents a $50\times$ **reduction in total action selections** ($5000 / 100 = 50$). Because the uncertainty term is governed by a square root in the denominator ($\frac{1}{\sqrt{N}}$), this $50\times$ reduction in samples produces a $\sqrt{50} \approx 7.07\times$ **increase in bonus scale** ($0.043c \to 0.303c$). Halving an uncertainty bonus requires a fourfold ($4\times$) increase in action selection trials.

UCB Algorithmic Execution & Testbed Dynamics

```
# UCB Algorithmic Execution Structure
Initialize for all a in 1..k:
    Q(a) = 0
    N(a) = 0

Loop for t = 1, 2, ...:
    # Forced initialization rule for unselected actions
    if any action a has N(a) == 0:
        A_t = pick_first_action_with_N_zero()
    else:
        A_t = argmax_a [ Q(a) + c * sqrt( ln(t) / N(a) ) ]
    
    R_t = execute_action(A_t)
    N(A_t) = N(A_t) + 1
    Q(A_t) = Q(A_t) + (1 / N(A_t)) * [ R_t - Q(A_t) ]
```

---

Unified Synthesis: Deconstructing the Step-11 Performance Artifacts

Both **Optimistic Initial Values (**$Q_1 = 5$**)** and **UCB (**$c = 2$**)** display a distinct, sharp performance spike at **Step 11** on the 10-armed testbed before settling into their long-term trajectories. While visually similar on benchmark plots, the underlying algorithmic drivers differ fundamentally:

```
Side-by-Side Comparison of Step-11 Artifact Mechanisms

OPTIMISTIC INITIAL VALUES (Q1 = 5)          UCB ACTION SELECTION (c = 2)
----------------------------------          ----------------------------
• Driver: Disappointment                    • Driver: Zero-Count Override
• Steps 1–10: High Q1=5 causes greedy       • Steps 1–10: Rule N(a)=0 forces sampling
  agent to try every arm once. Every          every arm once sequentially to initialize 
  reward disappoints (R < 5), depressing      counts.
   estimates sequentially.                  
• At Step 11: All N(a) = 1. The agent       • At Step 11: All N(a) = 1. Consequently,
  briefly exploits whichever arm gave the     the uncertainty bonuses c*sqrt(ln 11 / 1) 
  highest single reward in steps 1–10,        are identical across all arms. UCB is 
  producing a performance spike.             forced to pick the arm with the highest 
                                              single reward observed so far.
• Subsequent Steps: As that arm is re-       • Subsequent Steps: Once an arm is chosen 
  sampled, its estimate drops further,        again, its N(a) becomes 2, reducing its 
  forcing the agent to re-explore all arms    bonus and allowing other arms' uncertainty
  extensively over steps 12–200.              terms to drive exploration systematically.
```

Having analyzed individual algorithmic mechanics, we now construct a unified framework comparing all four policies across dynamic operational environments.

---

5. Comprehensive Policy Matrix & Environmental Adaptability

Selecting an appropriate action-selection policy requires matching algorithm mechanics to environmental characteristics, state space scales, and stationarity constraints.

Comparative Synthesis Matrix

|Feature / Dimension|Greedy Policy ($\epsilon = 0$)|$\epsilon$-Greedy Policy|Optimistic Initial Values|Upper Confidence Bound (UCB)|
|---|---|---|---|---|
|**Exploration Mechanism**|None. Pure exploitation.|Uniform random choice with probability $\epsilon$.|Greedy selection driven by disappointing initial expectations.|Optimism in the face of uncertainty via upper confidence bounds.|
|**When Exploration Occurs**|Never.|Continuously across the agent's lifespan.|Concentrated heavily during initial operational steps.|High early on; decays asymptotically as action counts grow.|
|**Key Parameters to Tune**|None.|$\epsilon$ (Exploration rate).|$Q_1$ (Initial baseline value).|$c$ (Exploration weight).|
|**Stationary Performance**|**Poor:** Trapped on suboptimal arms in $\approx 66\%$ of trials; average reward $\approx 1.03$.|**Good:** Reliable. $\epsilon=0.1$ converges fast; $\epsilon=0.01$ reaches high asymptotic reward ($99.1\%$).|**Very Good:** Reaches $\approx 86\%$ optimal action accuracy. Exploration decays naturally.|**Best:** Top performance on benchmark testbeds. High sample efficiency.|
|**Non-Stationary Performance**|**Poor:** Cannot adapt to distribution shifts.|**Good:** Persistent exploration ($\epsilon$) detects environmental changes continuously.|**Poor:** Fails completely after early steps; cannot re-explore later changes.|**Poor (Unmodified):** Action counts $N_t(a)$ grow monotonically, suppressing future uncertainty bonuses.|
|**Scalability & Computational Overhead**|Minimal $O(k)$ max calculation.|Minimal $O(k)$ calculation; random number draw.|Minimal $O(k)$ calculation.|Higher $O(k)$ overhead in bandits. Fails in large MDP state spaces ($S \times A$).|

Failure Modes & Adaptations in Non-Stationary Settings

Failure of Optimistic Initialization

Optimistic Initial Values depend on high initial expectations ($Q_1 \gg q^*(a)$). Once these initial values decay through sampling, the mechanism disappears. If a network route degrades at step $t = 2000$, the agent's updated estimates remain low, leaving it unable to re-explore alternative paths that may now be optimal.

Failure of Unmodified UCB

In standard UCB, total step count $t$ and action counts $N_t(a)$ accumulate monotonically over time. As $N_t(a)$ grows large (e.g., $N = 50,000$), the bonus term $c\sqrt{\ln t / N_t(a)}$ shrinks close to zero. If an arm's underlying reward distribution shifts later in execution, its high historic count $N_t(a)$ suppresses its uncertainty bonus, preventing the agent from recognizing the change.

Non-Stationary Modifications for UCB

To restore adaptability in dynamic environments, UCB must be augmented with mechanisms that forget distant history:

1. **Sliding-Window UCB:** Restricts step count $t$ and action counts $N_t(a)$ to a local temporal window $\tau$ (e.g., the last 1,000 steps).
2. **Discounted UCB:** Applies a discount factor $\gamma \in (0, 1)$ to past selection counts, weighting recent action trials higher than older ones.

Architectural Bottleneck: The State-Space Explosion in UCB

While UCB excels in multi-armed bandits, scaling UCB to full Reinforcement Learning—specifically Markov Decision Processes (MDPs) with large state spaces $S$—reveals a major structural bottleneck. In full RL, count tracking requires maintaining selection counts $N(s, a)$ for every state-action pair. In continuous or high-dimensional state spaces (such as autonomous network packet scheduling across millions of dynamic flow states), the size of $S \times A$ approaches infinity. Tracking explicit counts $N(s,a)$ becomes memory-intractable, and individual state-action visits become so sparse that $N(s,a)$ remains near zero, causing UCB uncertainty bounds to break down.

To confirm mastery of these mathematical and algorithmic concepts, we turn to step-by-step practical exercises.

---

6. Step-by-Step Practical Exercises & Exam Solutions

---

Exercise 1: Trace of $\epsilon$-Greedy 3-Armed Bandit (Sample Average Updates)

Problem Statement

A 3-armed bandit agent uses $\epsilon$-greedy action selection with sample-average value updates and $Q_1(a) = 0$ for all $a \in \{1, 2, 3\}$. Ties are broken randomly. The trace below shows chosen actions $A_t$ and observed rewards $R_t$ over the first 5 steps:

\begin{array}{c|c|c} t & A_t & R_t \ \hline 1 & 1 & 2 \ 2 & 2 & 1 \ 3 & 1 & 0 \ 4 & 3 & 3 \ 5 & 3 & 1 \ \end{array}

1. Compute value estimates $Q(a)$ after each step.
2. Identify which steps were definitively exploration choices versus ambiguous steps (which could have been greedy or exploratory).
3. Assuming $\epsilon = 0.1$, calculate the probability of selecting Action 3 at step $t = 6$.

---

Solution & Step-by-Step Execution

1. Sample Average Update Tracking Table

Using sample average update rule $Q_{n+1}(a) = \frac{\sum \text{Rewards for } a}{N(a)}$:

- **Step 1 (**$t=1$**):** Select $A_1 = 1$, receive $R_1 = 2$.
    - $N(1) = 1 \implies Q(1) = \frac{2}{1} = 2.0$
    - State Estimates: $Q = [2.0, 0.0, 0.0]$
- **Step 2 (**$t=2$**):** Select $A_2 = 2$, receive $R_2 = 1$.
    - $N(2) = 1 \implies Q(2) = \frac{1}{1} = 1.0$
    - State Estimates: $Q = [2.0, 1.0, 0.0]$
- **Step 3 (**$t=3$**):** Select $A_3 = 1$, receive $R_3 = 0$.
    - $N(1) = 2 \implies Q(1) = \frac{2 + 0}{2} = 1.0$
    - State Estimates: $Q = [1.0, 1.0, 0.0]$
- **Step 4 (**$t=4$**):** Select $A_4 = 3$, receive $R_4 = 3$.
    - $N(3) = 1 \implies Q(3) = \frac{3}{1} = 3.0$
    - State Estimates: $Q = [1.0, 1.0, 3.0]$
- **Step 5 (**$t=5$**):** Select $A_5 = 3$, receive $R_5 = 1$.
    - $N(3) = 2 \implies Q(3) = \frac{3 + 1}{2} = 2.0$
    - State Estimates: $Q = [1.0, 1.0, 2.0]$

|Step $t$|Selected $A_t$|Observed $R_t$|Updated $Q(1)$|Updated $Q(2)$|Updated $Q(3)$|Action Counts $[N_1, N_2, N_3]$|
|---|---|---|---|---|---|---|
|**0**|—|—|$0.0$|$0.0$|$0.0$|$[0, 0, 0]$|
|**1**|$1$|$2$|$2.0$|$0.0$|$0.0$|$[1, 0, 0]$|
|**2**|$2$|$1$|$2.0$|$1.0$|$0.0$|$[1, 1, 0]$|
|**3**|$1$|$0$|$1.0$|$1.0$|$0.0$|$[2, 1, 0]$|
|**4**|$3$|$3$|$1.0$|$1.0$|$3.0$|$[2, 1, 1]$|
|**5**|$3$|$1$|$1.0$|$1.0$|$2.0$|$[2, 1, 2]$|

---

2. Step Classification (Exploration vs. Ambiguous)

- **Step 1:** Ambiguous. Prior estimates were tied at $[0, 0, 0]$. Action 1 was optimal under tie-breaking, so this step could reflect either a greedy tie-break or a random choice.
- **Step 2:** **Definitively Exploration.** Prior estimates were $Q = [2.0, 0.0, 0.0]$. Action 1 was the unique greedy choice ($Q=2.0$). The agent chose Action 2 ($Q=0.0$), which must have been a random exploration step.
- **Step 3:** Ambiguous. Prior estimates were $Q = [2.0, 1.0, 0.0]$. The unique greedy choice was Action 1 ($Q=2.0$). The agent chose Action 1, which could be a greedy selection or a random exploration step that happened to land on Action 1.
- **Step 4:** **Definitively Exploration.** Prior estimates were $Q = [1.0, 1.0, 0.0]$. Greedy actions were Action 1 or 2 ($Q=1.0$). The agent selected Action 3 ($Q=0.0$), which must have been a random exploration choice.
- **Step 5:** Ambiguous. Prior estimates were $Q = [1.0, 1.0, 3.0]$. Action 3 was the unique greedy choice ($Q=3.0$). The agent chose Action 3.

---

3. Probability Calculation for Action 3 at Step $t = 6$

Prior to step 6, action value estimates are $Q = [1.0, 1.0, 2.0]$. Action 3 is the unique greedy choice ($a^* = 3$).

Using the theoretical optimal probability formula derived in Section 3 for $k = 3$ arms and $\epsilon = 0.1$:

$$P(A_6 = 3) = (1 - \epsilon) + \frac{\epsilon}{k} = (1 - 0.1) + \frac{0.1}{3} = 0.90 + 0.03333 = 0.93333 \quad (93.33\%)$$

**Derivation Mapping Note:** The term $0.90$ ($1 - \epsilon$) represents the probability of executing a greedy choice, while $0.03333$ ($\frac{\epsilon}{k} = \frac{0.1}{3}$) represents the probability of selecting Action 3 during a uniform random exploration step.

---

Summary Answer Footer: Exercise 1

- **Final Value Estimates at** $t=5$**:** $Q(1) = 1.0, Q(2) = 1.0, Q(3) = 2.0$
- **Definitively Exploration Steps:** Step 2 and Step 4
- **Selection Probability** $P(A_6 = 3)$**:** $93.33\%$ ($0.93333$)

---

Exercise 2: Non-Stationary Exponential Decay Update ($\alpha = 0.5$)

Problem Statement

A single-arm system uses a constant step size $\alpha = 0.5$ with an initial estimate $Q_1 = 0$. The observed sequence of rewards over four steps is $R_1 = 4, R_2 = 0, R_3 = 2, R_4 = 2$.

1. Compute estimates $Q_2, Q_3, Q_4, Q_5$ using $Q_{n+1} = Q_n + \alpha[R_n - Q_n]$.
2. Express $Q_5$ as an expanded weighted sum of $Q_1, R_1, R_2, R_3, R_4$ and verify that the weights sum to 1. Direct link to Section 1 indexing required.
3. Contrast this result with a standard sample average and justify why constant step-size updates are superior in non-stationary environments.

---

Solution & Step-by-Step Execution

1. Numerical Estimates Step Computation

Using $Q_{n+1} = \alpha R_n + (1 - \alpha) Q_n$ with $\alpha = 0.5$ and $(1 - \alpha) = 0.5$:

- $Q_2 = 0.5(R_1) + 0.5(Q_1) = 0.5(4) + 0.5(0) = \mathbf{2.0}$
- $Q_3 = 0.5(R_2) + 0.5(Q_2) = 0.5(0) + 0.5(2.0) = \mathbf{1.0}$
- $Q_4 = 0.5(R_3) + 0.5(Q_3) = 0.5(2) + 0.5(1.0) = \mathbf{1.5}$
- $Q_5 = 0.5(R_4) + 0.5(Q_4) = 0.5(2) + 0.5(1.5) = \mathbf{1.75}$

---

2. Algebraic Expansion of $Q_5$ & Direct Link to Section 1 Indexing

Recall from Section 1 the general expanded summation formula for $n$ steps:

$$Q_{n+1} = (1 - \alpha)^n Q_1 + \sum_{i=1}^{n} \alpha (1 - \alpha)^{n-i} R_i$$

For $Q_5$, we set $n = 4$ steps. Expanding each term individually using $\alpha = 0.5$ and $(1 - \alpha) = 0.5$:

- For $i = 4$ ($R_4$): $\alpha(1 - \alpha)^{4-4} R_4 = 0.5(0.5)^0 R_4 = \mathbf{0.5 R_4}$
- For $i = 3$ ($R_3$): $\alpha(1 - \alpha)^{4-3} R_3 = 0.5(0.5)^1 R_3 = \mathbf{0.25 R_3}$
- For $i = 2$ ($R_2$): $\alpha(1 - \alpha)^{4-2} R_2 = 0.5(0.5)^2 R_2 = \mathbf{0.125 R_2}$
- For $i = 1$ ($R_1$): $\alpha(1 - \alpha)^{4-1} R_1 = 0.5(0.5)^3 R_1 = \mathbf{0.0625 R_1}$
- Initial Value $Q_1$: $(1 - \alpha)^4 Q_1 = (0.5)^4 Q_1 = \mathbf{0.0625 Q_1}$

Combining these terms yields the complete weighted sum:

$$Q_5 = 0.5 R_4 + 0.25 R_3 + 0.125 R_2 + 0.0625 R_1 + 0.0625 Q_1$$

**Checking Weight Sum Identity:**

$$\text{Sum} = 0.5 + 0.25 + 0.125 + 0.0625 + 0.0625 = \mathbf{1.0}$$

**Numerical Verification:**

$$Q_5 = 0.5(2) + 0.25(2) + 0.125(0) + 0.0625(4) + 0.0625(0) = 1.0 + 0.5 + 0 + 0.25 + 0 = \mathbf{1.75}$$

---

3. Sample Average Comparison & Non-Stationary Justification

A standard sample average gives:

$$Q_5^{\text{sample}} = \frac{R_1 + R_2 + R_3 + R_4}{4} = \frac{4 + 0 + 2 + 2}{4} = \frac{8}{4} = \mathbf{2.0}$$

**Justification:** The constant step-size estimate ($1.75$) places $50\%$ of its total weight on the most recent observation ($R_4 = 2$), assigning only $6.25\%$ to the initial observation ($R_1 = 4$). Conversely, the sample average ($2.0$) weights all past outcomes equally at $25\%$. In non-stationary environments where reward distributions change over time, older samples no longer reflect current expected values. Constant step-size updates exponentially discount older data, allowing value estimates to adapt to structural changes dynamically.

---

Summary Answer Footer: Exercise 2

- **Computed Estimates:** $Q_2 = 2.0, Q_3 = 1.0, Q_4 = 1.5, \mathbf{Q_5 = 1.75}$
- **Expanded Weights:** $Q_5 = 0.5 R_4 + 0.25 R_3 + 0.125 R_2 + 0.0625 R_1 + 0.0625 Q_1$ ($\text{Sum} = 1.0$)
- **Sample Average Comparison:** $Q_5^{\text{sample}} = 2.0$ vs. $Q_5^{\text{exponential}} = 1.75$

---

Exercise 3: Optimistic Initial Values Hand Trace (Deterministic Rewards)

Problem Statement

Consider a two-armed bandit with deterministic rewards: Action $a_1$ always pays $R = 1$, and Action $a_2$ always pays $R = 2$. The agent is purely greedy ($\epsilon = 0$) using a step-size $\alpha = 0.5$. Ties are broken in favor of $a_1$.

1. Trace the first 6 steps with optimistic initial values $Q_1(a_1) = 5, Q_1(a_2) = 5$.
2. Trace the first 6 steps with realistic initial values $Q_1(a_1) = 0, Q_1(a_2) = 0$.
3. Compare which run successfully discovers the optimal arm $a_2$, and explain the underlying mechanism.

---

Solution & Step-by-Step Execution

1. Trace A: Optimistic Values ($Q_1 = 5$)

Update formula: $Q_{n+1}(a) = Q_n(a) + 0.5 \big[ R - Q_n(a) \big]$.

- **Step 1:** $Q = [5.0, 5.0]$. Tie breaks in favor of $a_1$. Select $A_1 = a_1$. Receive $R = 1$.
    - $Q_2(a_1) = 5 + 0.5(1 - 5) = 3.0$. Estimates: $Q = [3.0, 5.0]$.
- **Step 2:** $Q = [3.0, 5.0]$. Greedy choice is $a_2$ ($5.0 > 3.0$). Select $A_2 = a_2$. Receive $R = 2$.
    - $Q_3(a_2) = 5 + 0.5(2 - 5) = 3.5$. Estimates: $Q = [3.0, 3.5]$.
- **Step 3:** $Q = [3.0, 3.5]$. Greedy choice is $a_2$ ($3.5 > 3.0$). Select $A_3 = a_2$. Receive $R = 2$.
    - $Q_4(a_2) = 3.5 + 0.5(2 - 3.5) = 2.75$. Estimates: $Q = [3.0, 2.75]$.
- **Step 4:** $Q = [3.0, 2.75]$. Greedy choice is $a_1$ ($3.0 > 2.75$). Select $A_4 = a_1$. Receive $R = 1$.
    - $Q_5(a_1) = 3.0 + 0.5(1 - 3.0) = 2.0$. Estimates: $Q = [2.0, 2.75]$.
- **Step 5:** $Q = [2.0, 2.75]$. Greedy choice is $a_2$ ($2.75 > 2.0$). Select $A_5 = a_2$. Receive $R = 2$.
    - $Q_6(a_2) = 2.75 + 0.5(2 - 2.75) = 2.375$. Estimates: $Q = [2.0, 2.375]$.
- **Step 6:** $Q = [2.0, 2.375]$. Greedy choice is $a_2$ ($2.375 > 2.0$). Select $A_6 = a_2$. Receive $R = 2$.
    - $Q_7(a_2) = 2.375 + 0.5(2 - 2.375) = 2.1875$. Estimates: $Q = [2.0, 2.1875]$.

---

2. Trace B: Realistic Values ($Q_1 = 0$)

- **Step 1:** $Q = [0.0, 0.0]$. Tie breaks in favor of $a_1$. Select $A_1 = a_1$. Receive $R = 1$.
    - $Q_2(a_1) = 0 + 0.5(1 - 0) = 0.5$. Estimates: $Q = [0.5, 0.0]$.
- **Step 2:** $Q = [0.5, 0.0]$. Greedy choice is $a_1$ ($0.5 > 0.0$). Select $A_2 = a_1$. Receive $R = 1$.
    - $Q_3(a_1) = 0.5 + 0.5(1 - 0.5) = 0.75$. Estimates: $Q = [0.75, 0.0]$.
- **Step 3:** $Q = [0.75, 0.0]$. Select $a_1 \implies Q_4(a_1) = 0.875$. Estimates: $Q = [0.875, 0.0]$.
- **Step 4:** $Q = [0.875, 0.0]$. Select $a_1 \implies Q_5(a_1) = 0.9375$. Estimates: $Q = [0.9375, 0.0]$.
- **Step 5:** $Q = [0.9375, 0.0]$. Select $a_1 \implies Q_6(a_1) = 0.96875$. Estimates: $Q = [0.96875, 0.0]$.
- **Step 6:** $Q = [0.96875, 0.0]$. Select $a_1 \implies Q_7(a_1) = 0.984375$. Estimates: $Q = [0.984375, 0.0]$.

---

3. Run Comparison & Underlying Discovery Mechanism

- **Comparison Result:** The run initialized with optimistic values ($Q_1 = 5$) successfully discovers the optimal action $a_2$ by Step 2 and locks onto it permanently from Step 5 onward. The realistic run ($Q_1 = 0$) gets trapped on suboptimal action $a_1$ indefinitely and never samples $a_2$.
- **Underlying Mechanism:** Optimistic initialization forces exploration through disappointed expectations. Because $Q_1(a) = 5$ exceeds all actual rewards ($1$ or $2$), every real outcome produces a negative update error ($R_t - Q_t < 0$), driving that action's estimate down. This disappointment forces a greedy selection policy to switch arms systematically until all estimates drop to near their true expected means. Conversely, under zero initialization ($Q_1 = 0$), sampling $a_1$ yields $R=1 > 0$, raising $Q(a_1)$ above $Q(a_2) = 0$. This permanently locks a greedy agent into exploiting $a_1$ without ever trying $a_2$.

---

Summary Answer Footer: Exercise 3

- **Optimistic Trace (**$Q_1=5$**):** Discovers $a_2$ at Step 2; converges permanently to $a_2$ at Step 5.
- **Realistic Trace (**$Q_1=0$**):** Trapped permanently on suboptimal arm $a_1$; never explores $a_2$.

---

Exercise 4: UCB Action Score Calculation ($t = 11$ Benchmark Trace)

Problem Statement

At time step $t = 11$, an agent using UCB action selection has accumulated the estimates and action counts shown below ($\ln(11) \approx 2.40$):

\begin{array}{c|c|c} \text{Action } a & Q_t(a) & N_t(a) \ \hline a_1 & 1.0 & 4 \ a_2 & 1.5 & 5 \ a_3 & 0.8 & 1 \ \end{array}

1. Determine which action is selected by a pure greedy policy ($\epsilon = 0$).
2. Calculate UCB action selection scores for all arms using $c = 2$ and identify the winning action.
3. Calculate UCB action selection scores for all arms using $c = 0.5$ and identify the winning action.
4. Explain why the winning action changes as $c$ varies.

---

Solution & Step-by-Step Execution

1. Pure Greedy Selection ($\epsilon = 0$)

$$\text{Greedy Selection} = \underset{a}{\arg\max} \, Q_t(a) = \arg\max(1.0, 1.5, 0.8) = \mathbf{a_2}$$

---

2. Base Uncertainty Bonus Computation

For each action, calculate $\text{Bonus Factor} = \sqrt{\frac{\ln t}{N_t(a)}} = \sqrt{\frac{2.40}{N_t(a)}}$:

- **Action 1 (**$a_1$**):** $\sqrt{\frac{2.40}{4}} = \sqrt{0.60} \approx 0.7746$
- **Action 2 (**$a_2$**):** $\sqrt{\frac{2.40}{5}} = \sqrt{0.48} \approx 0.6928$
- **Action 3 (**$a_3$**):** $\sqrt{\frac{2.40}{1}} = \sqrt{2.40} \approx 1.5492$

---

3. UCB Score Evaluation for $c = 2$

$$\text{Score}(a) = Q_t(a) + 2 \cdot \sqrt{\frac{2.40}{N_t(a)}}$$

- $\text{Score}(a_1) = 1.0 + 2(0.7746) = 1.0 + 1.5492 = \mathbf{2.5492}$
- $\text{Score}(a_2) = 1.5 + 2(0.6928) = 1.5 + 1.3856 = \mathbf{2.8856}$
- $\text{Score}(a_3) = 0.8 + 2(1.5492) = 0.8 + 3.0984 = \mathbf{3.8984}$

$$\text{Winning Action } (c = 2) = \mathbf{a_3} \quad (\text{Score } 3.8984)$$

---

4. UCB Score Evaluation for $c = 0.5$

$$\text{Score}(a) = Q_t(a) + 0.5 \cdot \sqrt{\frac{2.40}{N_t(a)}}$$

- $\text{Score}(a_1) = 1.0 + 0.5(0.7746) = 1.0 + 0.3873 = \mathbf{1.3873}$
- $\text{Score}(a_2) = 1.5 + 0.5(0.6928) = 1.5 + 0.3464 = \mathbf{1.8464}$
- $\text{Score}(a_3) = 0.8 + 0.5(1.5492) = 0.8 + 0.7746 = \mathbf{1.5746}$

$$\text{Winning Action } (c = 0.5) = \mathbf{a_2} \quad (\text{Score } 1.8464)$$

---

5. Analysis: Why Parameter $c$ Changes the Selection Decision

The exploration parameter $c$ scales the agent's appetite for uncertainty relative to expected reward estimates:

- **When** $c = 2$ **(High Exploration Appetite):** The bonus term dominates selection decisions. Because Action 3 has been sampled only once ($N = 1$), its large uncertainty term ($\approx 1.5492$) receives a heavy weighting ($2 \times 1.5492 = 3.0984$). This propels Action 3's total score ($3.8984$) past Action 2 ($2.8856$), driving the agent to explore Action 3 to reduce its uncertainty.
- **When** $c = 0.5$ **(Low Exploration Appetite / High Exploitation Preference):** The weight assigned to uncertainty bonuses is reduced fourfold. As a result, Action 2's superior sample estimate ($Q = 1.5$) outweighs the small uncertainty bonus added to Action 3 ($0.7746$). Consequently, the policy shifts back to exploiting the highest estimated mean.

---

Summary Answer Footer: Exercise 4 ($t = 11$ Main Solution)

- **Greedy Choice (**$\epsilon = 0$**):** **Action** $a_2$ ($Q = 1.5$)
- **UCB Choice (**$c = 2.0$**):** **Action** $a_3$ ($\text{Score} = 3.8984$)
- **UCB Choice (**$c = 0.5$**):** **Action** $a_2$ ($\text{Score} = 1.8464$)

---

Parallel Exam Variant Callout: Step $t = 21$ Analysis (Slide 46 Variant)

**Problem Statement:** At time step $t = 21$, an agent has accumulated the following estimates and counts ($\ln(21) \approx 3.04$):

\begin{array}{c|c|c} \text{Action } a & Q_t(a) & N_t(a) \ \hline a_1 & 0.6 & 10 \ a_2 & 0.9 & 8 \ a_3 & 0.4 & 2 \ \end{array}

1. Determine the pure greedy action selection ($\epsilon = 0$).
2. Calculate UCB scores for $c = 1.0$ and identify the winning action.
3. Calculate UCB scores for $c = 0.5$ and identify the winning action.

**Step-by-Step Calculation:**

1. **Greedy Choice (**$\epsilon = 0$**):** $$\arg\max_a Q_t(a) = \arg\max(0.6, 0.9, 0.4) = \mathbf{a_2}$$
2. **Base Uncertainty Factors** $\sqrt{\frac{\ln 21}{N_t(a)}} = \sqrt{\frac{3.04}{N_t(a)}}$**:**
    - Action $a_1$: $\sqrt{3.04 / 10} = \sqrt{0.304} \approx \mathbf{0.5514}$
    - Action $a_2$: $\sqrt{3.04 / 8} = \sqrt{0.38} \approx \mathbf{0.6164}$
    - Action $a_3$: $\sqrt{3.04 / 2} = \sqrt{1.52} \approx \mathbf{1.2329}$
3. **UCB Scores for** $c = 1.0$**:**
    - $\text{Score}(a_1) = 0.6 + 1.0(0.5514) = \mathbf{1.1514}$
    - $\text{Score}(a_2) = 0.9 + 1.0(0.6164) = \mathbf{1.5164}$
    - $\text{Score}(a_3) = 0.4 + 1.0(1.2329) = \mathbf{1.6329} \implies \mathbf{\text{Winning Action: } a_3}$
4. **UCB Scores for** $c = 0.5$**:**
    - $\text{Score}(a_1) = 0.6 + 0.5(0.5514) = 0.6 + 0.2757 = \mathbf{0.8757}$
    - $\text{Score}(a_2) = 0.9 + 0.5(0.6164) = 0.9 + 0.3082 = \mathbf{1.2082} \implies \mathbf{\text{Winning Action: } a_2}$
    - $\text{Score}(a_3) = 0.4 + 0.5(1.2329) = 0.4 + 0.6165 = \mathbf{1.0165}$

**Parallel Exam Variant Summary Footer (**$t=21$**):**

- **Greedy Choice (**$\epsilon = 0$**):** **Action** $a_2$ ($Q = 0.9$)
- **UCB Choice (**$c = 1.0$**):** **Action** $a_3$ ($\text{Score} = 1.6329$)
- **UCB Choice (**$c = 0.5$**):** **Action** $a_2$ ($\text{Score} = 1.2082$)

---

Pedagogical Synthesis Confirmation

These worked solutions confirm that tracking exponential updates provides non-stationary reactivity, optimistic initial values drive systematic early exploration via disappointment, and UCB balances exploitation and uncertainty bounds efficiently across stationary benchma