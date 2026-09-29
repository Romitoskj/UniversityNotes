# Advanced Machine Learning & Computer Vision: Neural Networks and Backpropagation

### 1. Foundations of Machine Learning and Computer Vision

The strategic evolution of Artificial Intelligence represents a transition from explicit programming—where developers manually hard-coded every rule—to experiential learning. In a professional architecture context, we define this transition using the formalization provided by Tom Mitchell (1998): A computer program is said to learn from **Experience (E)** with respect to a **Task (T)** and a **Performance measure (P)**, if its performance on T, as measured by P, improves with experience E. This contrasts with Arthur Samuel’s (1959) more intuitive definition of "learning without being explicitly programmed."

#### The Role of Experience

In the current state-of-the-art, "Experience" has evolved beyond simple supervised datasets. While supervised learning remains the standard for deployment, **self-supervised pre-training** is the strategic "key to actually make it work." By learning robust representations from unlabeled data before fine-tuning, models achieve the generalization required for complex perception.

#### The Dual Nature of Computer Vision

Computer Vision operates at the intersection of theory and industrial utility:

- **Science:** Exploring the foundations of perception by building computational models of human vision to understand how biological agents process their environment.
- **Engineering:** Designing robust systems for real-world autonomy, such as training vehicles to detect pedestrians or navigate complex urban environments.
- **Applications:** A broad suite ranging from medical imaging (diagnostic support) to surveillance (tracking in airports) and entertainment. Notably, some sectors are considered mature; for instance, the automatic processing of addresses on envelopes (OCR) has been 99% solved for over 15 years.

#### Biological vs. Digital Vision Systems

Biological hardware operates on a hierarchy of processing speeds: the retina processes signals in 20–40ms, the V1 cortex in 40–60ms, and categorical judgments/decision-making occur within 100–190ms. This serial-like biological hierarchy contrasts with the massive parallelism of digital systems. Human eyes utilize **Rods** (sensitive to light/dark, dense in the periphery for evolutionary predator avoidance/motion detection) and **Cones** (dense in the fovea for high-resolution color).

As architects, we recognize that RGB sensors are a "human-friendly" approximation because our neurons are specifically tuned to those frequencies. However, modern AI has moved toward a "super-engineered" paradigm. To borrow an industry analogy: if you want to fly, you go to the airport; you do not "find a mosquito" to study how to flap wings. We seek the underlying principles—like lift (portance)—rather than biological mimicry. This shift necessitates a move toward the mathematical representation of visual data.

### 2. Linear Image Classifiers and Probabilistic Frameworks

To move from raw pixels to semantic understanding, we must map high-dimensional visual data into discrete categories. The linear classifier serves as the foundational unit for this mapping and is the essential building block of deep architectures.

#### From Pixels to Scores

An image (e.g., 32x32x3) is vectorized into N = 3072 features. We apply the linear function f(x,W) = Wx + b. For a 10-class problem, the dimensions are:

- **Weight Matrix (****W****):** [10 \times 3072], representing [Classes \times Input\_Features].
- **Bias (****b****):** [10 \times 1]. The result is a 10 \times 1 vector of **class scores**, or what the literature calls **unnormalized logical probabilities** (commonly referred to in industry as **logits**).

#### The Softmax Classifier

In Multinomial Logistic Regression, we interpret these logical probabilities as actual probabilities. We exponentiate the scores to ensure they are positive and normalize them so they sum to 1. This transformation allows us to treat the output as a probability distribution.

#### Loss Functions and Information Theory

To train the model, we use **Cross-Entropy Loss**: L_i = -\log P(Y = y_i | X = x_i). From an information theory perspective, we are minimizing the KL divergence between the predicted distribution and the "one-hot" ground truth distribution. We effectively penalize the model until the predicted probability for the correct class approaches 1.00.

#### Regularization: Bending the Function

Training purely on data loss leads to overfitting. We calculate **Total Loss** as \text{Data Loss} + \lambda R(W), where \lambda is the regularization strength. Regularization prevents the model from "doing too well" on training data by penalizing large weights. Large weights allow the model to "bend" the classification function too sharply to fit noise. Modern architectural preference leans toward large-capacity models that are heavily regularized (using L2, L1, Dropout, or Batch Normalization) to ensure generalization. Once the error is defined, we must minimize it through optimization.

### 3. Optimization and Stochastic Gradient Descent (SGD)

Optimization in high-dimensional parameter spaces requires navigating a complex "loss surface" via iterative refinement.

#### Gradient Descent and the Learning Rate

We minimize loss by moving in the negative gradient direction. The most critical hyperparameter in this process is the **step size** (or **learning rate**): W = W - \text{step\_size} \times \text{grad} A step size that is too large will cause the model to diverge, while one that is too small will stall training.

#### Efficiency via Stochasticity

Calculating the gradient for an entire dataset is computationally prohibitive. **Stochastic Gradient Descent (SGD)** utilizes minibatches to estimate the gradient. While "noisy," this stochasticity is a strategic advantage: it acts as a form of implicit regularization, helping the optimization process escape sharp local minima that might otherwise trap a full-batch approach. This strategy is implemented through computational graphs.

### 4. Computational Graphs and Backpropagation Mechanics

Backpropagation decomposes complex functions into a chain of local, modular operations, allowing us to compute gradients efficiently.

#### The Chain Rule and Gradient Decomposition

The fundamental algorithm for downstream gradient calculation is: \frac{\partial L}{\partial x} = \frac{\partial L}{\partial y} \cdot \frac{\partial y}{\partial x} _(Downstream Gradient = Upstream Gradient_ _\times_ _Local Gradient)_

#### Standard Gate Behaviors

- **Add Gate:** A "gradient distributor" that passes the upstream gradient equally to all inputs.
- **Mul Gate:** A "swapper." For f = xy, the local gradient \partial f/\partial x is y. It scales the upstream gradient by the value of the _other_ input.
- **Max Gate:** A "gradient router." It sends the entire upstream gradient to the input that was highest during the forward pass (gradient is 0 for others).
- **Copy Gate:** A "gradient adder" that sums gradients from multiple branches.

#### Vector and Matrix Backpropagation

When x is a vector, the local gradient is a **Jacobian matrix**. For elementwise operations like **ReLU**, the Jacobian for N elements is an N \times N **diagonal matrix**. In production environments, we never explicitly form these massive matrices (avoiding O(N^2) memory costs). Instead, we use **implicit multiplication**: applying a bitmask (1 where x > 0, 0 otherwise) to the upstream gradient.

#### Implementation Logic

1. **Forward Pass:** Process nodes in a **topological sort** to compute loss and **cache** intermediate activations.
2. **Backward Pass:** Reverse the topological sort, applying the chain rule at each gate to update weights.

This modularity allows us to transition from single operations to multi-layer deep architectures.

### 5. Deep Neural Networks and Activation Functions

Deep architectures are required to learn "bent," non-linear classification functions for data that is not linearly separable.

#### The Necessity of Non-Linearity

Stacking linear layers (W_2 W_1 x) is mathematically equivalent to a single linear layer. Non-linear **activation functions** are required to break this linear collapse and allow the network to capture complex features.

#### Comparative Analysis of Activation Functions

|                |                              |                        |                                                        |
| -------------- | ---------------------------- | ---------------------- | ------------------------------------------------------ |
| Function       | Formula                      | Gradient Behavior      | Characteristics                                        |
| **ReLU**       | \max(0, x)                   | 1 for x > 0, else 0    | **Modern Default.** Efficient, but can "die" if x < 0. |
| **Sigmoid**    | 1 / (1 + e^{-x})             | Vanishing at extremes  | Historically popular; squashes to [0, 1].              |
| **Tanh**       | \tanh(x)                     | Vanishing at extremes  | Zero-centered; squashes to [-1, 1].                    |
| **Leaky ReLU** | \max(0.01x, x)               | 1 for x > 0, else 0.01 | Prevents "dying" neurons by allowing a small gradient. |
| **ELU**        | x if x>0 else \alpha(e^x -1) | Smooth gradient        | Robust against noise; computationally heavier.         |

#### Implementation Walkthrough (2-Layer Neural Network)

A 2-layer network f = W_2 \max(0, W_1 x) is implemented in NumPy as follows:

- **Forward:**
    1. Compute `h1 = W1.dot(x)`.
    2. **Store** `**h1**` **in cache** (required for the backward pass).
    3. Compute `a1 = np.maximum(0, h1)` (ReLU).
    4. Compute `scores = W2.dot(a1)`.
- **Backward:**
    1. Compute gradient of loss with respect to `scores`.
    2. Backprop through W_2.
    3. Backprop through the ReLU "router" using the cached `h1`.
    4. Backprop through W_1.

The modern success of computer vision stems from this synergy between architectural depth, non-linear activation, probabilistic loss formulation, and the efficient gradient flow of backpropagation.