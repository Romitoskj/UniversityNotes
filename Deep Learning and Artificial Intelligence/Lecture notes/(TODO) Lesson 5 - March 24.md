# Multi-layer Perceptron and Back-propagation

**1. Multi-Layer Perceptrons (MLPs) & Deep Networks**

- **Deep Composition:** A deep neural network, specifically a Multi-Layer Perceptron (or feed-forward neural network), is built by stacking simple blocks one after another. Each block consists of a linear transformation (a fine map) followed by a non-linear activation function (σ).
- **Why Non-Linearity?** Concatenating purely linear models results in another linear model, which does not increase the computational capacity or complexity of the model. Introducing non-linearities like the Logistic Sigmoid or Rectified Linear Unit (ReLU) allows the network to represent highly non-linear functions.
- **Network Architecture Terminology:**
    - **Layers:** The blocks are called layers. The one closest to the input is the input layer, the last one is the output layer, and the intermediate ones produce **hidden representations**. Note: Some practitioners confusingly refer to hidden representations as "activations".
    - **Neurons:** A neuron is mathematically defined as a single row of the linear map matrix within a block. Thus, a linear layer can be viewed as a collection of neurons operating in parallel, mapping inputs to a scalar via an inner product.
    - **Graph Illustrations:** In network diagrams, **nodes represent dimensions** of the input or output, and **edges represent the weights** of the linear map. If a layer is "fully connected," every input dimension connects to every output dimension. Missing edges imply weights that are exactly zero (sparsity).
- **The Output Layer & Co-domain:** The overall co-domain (output space) of the network is entirely determined by its very last transformation. For example, ending a network with a sigmoid function forces the output into a (0,1) range, which is ideal for classification. _Practical Tip:_ You can often repurpose a pre-trained generative model for classification simply by replacing and re-training (fine-tuning) its final layer.

**2. Deep ReLU Networks & Universality**

- **Deep ReLU Networks:** If you build a network using entirely ReLU activations (σ(x)=max{0,x}), the resulting mathematical function is **piecewise-linear**. The network creates many ReLU-shaped hyperplanes and stitches them together. The resulting function is continuous but has non-smooth discontinuities (kinks) everywhere the ReLU truncates to zero.
- **Universal Approximation Theorem (UAT):** This theorem guarantees that a network with just **one hidden layer** and a logistic sigmoid activation function can approximate _any continuous function_ over a compact domain to any desired accuracy.
- **The Catch:** The theorem is an existence proof; it is **not constructive**. It guarantees that a wide enough layer exists to represent the function, but it does not tell you how many neurons (q) are needed, nor how to find the optimal weights. Therefore, network design remains a process of trial and error.

**3. Training: Gradients and the Loss Function**

- **The Goal:** Training involves finding the optimal parameters (weights and biases) spread across all the linear blocks. To do this via gradient descent, we must compute the gradient of the **loss function** with respect to the **parameters**.
- **A Crucial Distinction:** We do _not_ compute the "gradient of the network." The gradient is computed over the scalar loss function, whose inputs are the millions of parameters, not the data inputs.
- **Numerical Approximation:** Approximating gradients using finite differences (wiggling the input by a small δ and checking the output) is computationally infeasible and numerically unstable for models with millions of parameters.

**4. Computational Graphs**

- To compute exact gradients efficiently, software like PyTorch dynamically constructs a **Computational Graph**, which is a Directed Acyclic Graph (DAG).
- **Nodes store Values, not Formulas:** As data passes from input to output (a **forward pass** or forward traversal), intermediate variables are created. These nodes store the actual _numerical values_ resulting from the atomic operations, which is fundamentally different from symbolic differentiation software (like Mathematica) which manipulates algebraic symbols.

**5. Automatic Differentiation: Forward Mode vs. Reverse Mode** Automatic differentiation uses the chain rule to compute gradients, but the _order_ of evaluation completely changes its efficiency.

- **Forward-Mode Autodiff:**
    - Traverses the graph from left to right (input to output), computing partial derivatives of the inner nodes with respect to the input.
    - It computes both the function value and its derivative in a single pass.
    - **The Bottleneck:** It must be repeated for _every single input parameter_. If a network has 7 billion parameters (p), forward mode requires 7 billion forward passes, making its computational complexity O(p). It is efficient only for functions with few inputs and many outputs.
- **Reverse-Mode Autodiff (Backpropagation):**
    - Traverses the graph from right to left (output to input). It computes the derivatives of the final output with respect to every intermediate node.
    - **The Algorithm:** It requires **one initial forward pass** to evaluate and store all the numerical values of the intermediate nodes, followed by **one backward pass** to accumulate the gradients via the chain rule.
    - **Why it is superior for Deep Learning:** Deep learning loss functions map millions of inputs (parameters) to exactly one output (a scalar loss). Reverse mode scales linearly with the number of outputs. Thus, it can compute the gradients for all 7 billion parameters simultaneously in just one backward pass, sharing the computation across the graph.
    - **The Matrix Associativity Analogy:** Because the chain rule relies on multiplying Jacobians, the difference between forward and reverse mode is mathematically equivalent to choosing where to put the parentheses in a chain of matrix multiplications (e.g., (XYT)Z vs X(YTZ)). Placing the parentheses to favor vector-matrix products avoids computing massive, intractable matrices.
    - **Clarification:** Backpropagation is _not_ just "the chain rule." Both forward and reverse modes use the chain rule. Backpropagation is the specific **computational technique** (reverse-mode autodiff) applied to the computational graph of the loss function.