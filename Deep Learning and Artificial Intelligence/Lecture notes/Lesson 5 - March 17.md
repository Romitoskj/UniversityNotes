# Stochastic Grandient Descent

## 1. Gradient Descent & Loss Landscapes

- **The Goal:** In deep learning, finding optimal parameters to train a network means minimizing a loss function. Because models like logistic regression and multi-layer perceptrons are non-convex and lack closed-form solutions, we must use iterative, first-order optimization algorithms like Gradient Descent (GD).

- **The Intuition:** The loss function $\ell_\Theta:\mathbb{R}^n\rightarrow\mathbb{R}$ can be visualized as a mountainous landscape or surface  (if $n=2$), where the domain is the parameter space and the height is the loss value. Optimizing is like a skier or a ball trying to roll down to the lowest point as fast as possible, in fact optimization process is represented as a trajectory on the loss function:

![](../../Pasted%20image%2020260416113837.png)

- **Algorithm:** 
	1. Start from some point $\Theta^{(0)}\in\mathbb{R}^n$
	2. Iteratively compute (update step):$$\Theta^{(t+1)}=\Theta^{(t)}-\alpha\nabla\ell_{\Theta^{(t)}};$$
	3. Stop when a minimum is reached, *e.g.*:$$\Vert\nabla\ell_{\Theta^{(t)}}\Vert\leq\epsilon$$for a suitable small positive value $\epsilon$.

- **Stopping Criteria & Overfitting:** We stop the algorithm when we reach a compute budget (iteration number) or a satisfactory low loss. Finding the exact global minimum (a loss of exactly zero) is actually undesirable, as it usually means the model has memorized the training noise and will overfit.

## 2. Gradient properties

**Iso-contours (Level Curves):** Instead of a 3D surface, we often view the loss from above using iso-contours—lines representing regions where the loss has a constant value.

### A. Orthogonality

The gradient vector, which points in the direction of steepest ascent, is **strictly orthogonal (perpendicular)** to the iso-contours.
- _Proof:_ The directional derivative along an iso-contour is zero (because the function value doesn't change). Since the directional derivative is the inner product of the gradient and the direction vector, an inner product of zero implies the vectors are orthogonal.
### B. Differentiability

Gradient Descent requires the function to be differentiable (smooth without breaks). Deep learning loss functions often have singularities (like the non-differentiable "kink" in the ReLU activation function). However, because these singular points are sparse, GD continues to work in practice.

### C. Stationary Points

These are points where the gradient vanishes (becomes exactly zero), which causes standard Gradient Descent to get stuck,. They include local minima, local maxima, and saddle points,.

## 3. The Learning Rate (**α**) & Schedules

- **Definition:** The learning rate is a hyperparameter that scales the gradient, dictating the length of the step taken in the parameter space,.
    - **Small** α**:** Safe, but convergence is agonizingly slow,.
    - **Large** α**:** Causes the trajectory to **overshoot** the minimum, violently bouncing back and forth or diverging,.
- **Line Search:** For strictly convex functions, algorithms can dynamically calculate the perfect step length at each iteration,.
- **Learning Rate Schedules:** In deep learning, we manually tune how α changes over time. Common schedules include:
    - **Exponential Decay:** Starts with a large step to make rapid progress, then exponentially decays to settle carefully into the minimum,.
    - **Cosine Annealing:** A highly popular schedule that smoothly varies the learning rate following a cosine curve (famously used to successfully train models like LLaMA).

## 4. Momentum: Escaping Local Minima

- **The Physics Analogy:** Standard GD is "memoryless." Momentum adds a memory mechanism, similar to a physical ball accumulating velocity and mass as it rolls down a hill,.
- **How it Works:** The step update v(t+1) becomes a combination of the current gradient and the previous step, scaled by a friction/decay parameter λ (e.g., 0.9),.
    - **Constructive Accumulation:** If subsequent gradients point in the same direction, momentum builds up, amplifying the step size up to 10× faster,.
    - **Overshooting as a Feature:** This built-up velocity allows the optimizer to intentionally overshoot small, suboptimal local minima (pitfalls) and escape saddle points,.
- **Unrolled Equation:** Expanding the momentum formula reveals that the current step is actually a sum of all past gradients, with older gradients having exponentially less influence,.

## 5. Stochastic Gradient Descent (SGD) & Mini-batches

- **The Computational Bottleneck:** To compute the true, exact gradient, one must compute the derivative for _every single data point_ in the dataset (n),. In deep learning, where n can be in the millions, a single GD step is computationally infeasible,.
- **The Mini-Batch Approximation:** SGD solves this by computing the gradient on a tiny, random subset of data called a **mini-batch** (e.g., 32 or 64 examples),. The data must be shuffled before sampling to ensure the mini-batch is an unbiased representation of the dataset (e.g., to avoid a batch entirely made of "dogs"),.
- **Epochs:** Because data is evaluated in small batches, training time is measured in "epochs." One epoch is completed when the model has processed enough mini-batches to cover the entire dataset one time,.
- **The "Unreasonable Effectiveness" of SGD:** Because SGD uses an approximated gradient, its trajectory oscillates and has no guarantee of going strictly downwards,. However, it is fundamentally successful: it is vastly faster than standard GD, and the noise introduced by the mini-batch sampling acts as an implicit regularizer, preventing the model from overfitting to the training data,,.
- **Crucial Distinction:** Using SGD or choosing different batch sizes **does not change the mathematical landscape of the loss function**. The surface of the loss function is defined entirely by the data and the network architecture. SGD is simply a "broken" or approximated compass used to navigate that exact same landscape.