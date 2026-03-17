# Stochastic Grandient Descent

## 1. The Bridge: From Logistic Regression to Gradient Descent

- **The Need for Optimization:** Chapter 3 establishes that while linear and polynomial regression can be solved analytically with a closed-form formula, **classification problems like logistic regression cannot** due to the nonlinear sigmoid function. To train these models, we must abandon exact formulas and rely on iterative **nonlinear optimization techniques** like Gradient Descent.
- **Redefining the Goal (Overfitting):** Chapter 3 warns that a model with too much capacity can achieve zero error by perfectly memorizing the noise in the training data, leading to severe overfitting. The audio expands on this by emphasizing that **finding the true global minimum of the training loss is actually undesirable in deep learning**. Reaching a training loss of exactly zero usually equals memorization, meaning the model will generalize poorly to new data.

## 2. Gradient Descent Mechanics & Visualization

- **The Intuition:** Optimization is like a ball rolling down a mountain, continuously taking steps in the direction of the steepest descent. Mathematically, this direction is the **negative gradient**.
- **Iso-contours (Level Curves):** Instead of visualizing the loss function as a 3D surface, we can view it from above using iso-contours—lines where the loss function has a constant value.
- **Orthogonality:** A crucial geometric property is that **the gradient vector is always strictly orthogonal (perpendicular) to the iso-contours**. This is proven by taking the directional derivative along the curve: because the function value is constant along the curve, its derivative in that tangent direction is exactly zero, meaning the gradient's inner product with the curve is zero.
- **Stationary Points:** The algorithm stops when it reaches a stationary point where the gradient vanishes (becomes zero). However, this is not guaranteed to be a minimum; it could also be a maximum or a saddle point.

## 3. The Learning Rate (**α**) and Schedules

- **Definition:** The learning rate is a scalar hyperparameter that modulates the length of the step taken in the direction of the negative gradient.
- **The Goldilocks Problem:**
    - If α is too small, convergence is agonizingly slow, wasting compute budget.
    - If α is too large, the algorithm suffers from **overshooting**, violently bouncing back and forth across the valley without ever settling into the minimum.
- **Learning Rate Schedules:** Because a single static learning rate is rarely optimal, practitioners use schedules to dynamically change the learning rate during training. Common strategies include:
    - **Exponential Decay:** Starting with a large step to make rapid initial progress, then exponentially decaying the learning rate to safely settle into the minimum.
    - **Cosine Annealing:** A highly popular schedule that smoothly varies the learning rate following a cosine curve.

## 4. Momentum: Escaping Local Minima

- **The Concept:** Standard gradient descent is "memoryless," completely discarding the previous direction at each new step. **Momentum** adds a memory mechanism, similar to a physical ball accumulating velocity as it rolls down a hill.
- **How it Works:** At each step, the algorithm looks at the previous gradient updates, scales them by a decay factor (e.g., λ=0.9), and adds them to the current direction.
    - If subsequent gradients point in the **same direction**, momentum constructively builds up, amplifying the step size (e.g., up to 10x faster) and accelerating convergence down long, shallow valleys.
    - If the gradients **alternate directions** (like bouncing off the walls of a narrow valley), momentum dampens the oscillations by canceling out the opposing forces.
- **Overshooting as a Feature:** Momentum intentionally causes the optimization trajectory to overshoot small local minima. If the ball rolls into a shallow basin, its accumulated velocity carries it up and out the other side, hopefully allowing it to fall into a deeper, better minimum.

## 5. Stochastic Gradient Descent (SGD) & Mini-batches

- **The Computational Bottleneck:** To compute the exact, true gradient, you must sum the gradients of every single data point in your dataset. In deep learning, where datasets have millions of examples, this single step would take far too long.
- **The Stochastic Approximation:** SGD solves this by computing the gradient on a tiny, randomly sampled subset of the data called a **mini-batch** (e.g., 32 or 64 images). The randomness (stochasticity) requires the data to be shuffled so the mini-batch is a fair, unbiased representation of the whole dataset.
- **Epochs:** Because we are evaluating data in small chunks, we measure training time in "epochs." One epoch is completed when the model has looked at enough mini-batches to cover the entire dataset one time.
- **The Paradox of SGD:** By approximating the gradient, SGD's trajectory is noisy and unpredictable; there is no guarantee that every single step will actually decrease the loss. However, it is **unreasonably effective**: it executes vastly faster than standard gradient descent and the added noise actually acts as an implicit regularizer, preventing the model from overfitting to the training data.
- **A Crucial Distinction:** Using SGD does **not** change the mathematical landscape of the loss function itself. The loss function is entirely defined by the network architecture and the dataset. SGD is merely a "broken" or approximated compass used to navigate that exact same loss landscape.