*March 03 2026*

### 1. Parametric Models & Deep Learning

- **Definition:** Neural networks are essentially highly parameterized models, denoted as $$f_\Theta(x)=y$$where $\Theta$ represents a massive set of parameters (weights and biases).
- **Input/Output Spaces:** The domain and codomain of the network function f depend entirely on the task. The input $x$ and output $y$ can be high-dimensional vectors (e.g., an image x mapping to a probability distribution y).
- **Training vs. Testing:**
    - **Training Time:** The data points $X$ and $Y$ are given and fixed. The goal is to solve for the unknown parameters $\Theta$ that best map $X$ to $Y$.
    - **Testing Time:** The parameters $\Theta$ are fixed, and the model is used to predict a new output $y$ from a new, unseen input $x$.

### 2. Linear Regression & The Loss Function

Linear regression is the simplest non-trivial learning model, where the function $f$ is assumed to be linear in its parameters ($f=ax+b$). So we can write the model as:$$f_\Theta(x_i)=y_i$$
Having $a$ and $b$ makes possible to construct a mapping such that given some new input we can apply the function learned and get some new output. ![](Images/Pasted%20image%2020260308224432.png)

- **The Loss Function ($\ell$):** To find the parameters $a$ and $b$, we must quantify the error between the model's predictions $f_\Theta(x)$ and the true labels $y$ to perform an optimization process of that function. 
- **Mean Squared Error (MSE):** The most common choice for linear regression is the MSE, which computes the sum of the squared differences: 
	$$\ell_\Theta(\{x_i,y_i\})=\sum_{i=1}^N\frac{1}{N}(y_i-f_\Theta(x_i))^2$$
	- _Note on Constants:_ The 1/n averaging factor is often dropped because dividing by a constant does not change the location of the minimum parameters $\Theta$. This results in a similar function called **Sum of Squared Error (SSE)**.
- **Domain and Codomain of the Loss:** It is crucial to distinguish the loss function from the network function. The loss function $\ell:\mathbb{R}^n\rightarrow\mathbb{R}$ always takes the parameters as input (Domain: $\mathbb{R}^n$, where $n$ is the number of parameters) and outputs a single scalar score (Codomain: $\mathbb{R}$).
- **Least-square approximation:** To find the model we need to minimize the loss function: $$\epsilon=\min_\Theta\ell_\Theta(\{x_i,y_i\})$$In particular we are interested in finding the parameters $\Theta$ (minimizers) that makes the error reach the minimum value:$$\Theta^*=\arg\min_{\Theta \in \mathbb{R}}\ell(\Theta)$$When $f$ is linear the problem of choosing the parameters $\Theta$ that minimize the MSE is called *least-squares approximation* problem.

### 3. Convexity

To find the parameters that minimize the loss, we rely on optimization. The easiest functions to minimize are convex functions, because there always exists a unique minimum in them.

- **Jensen's Inequality:** A function is convex if $f(\alpha x+(1-\alpha)y) \leq \alpha f(x) + (1-\alpha)f(y), \forall x,y$ and $\alpha \in [0,1]$.
- **Visual Intuition:** By using a parameter α between 0 and 1, we trace a straight line (a convex combination) between two points on the function. If the function is convex, this straight line will always lie strictly above the curve of the function.![](Images/Pasted%20image%2020260308224154.png)
- **Global Minimum:** For a differentiable convex function, finding the minimum is straightforward: *compute the derivative, set it to zero, and solve for the parameters* (the global minimizer $x$ is where $\frac{df(x)}{dx}=0$).

### 4. Gradients and Steepest Ascent

Because the loss function has a high-dimensional domain (millions of parameters), we replace the standard 1D derivative with the **gradient**, denoted as $\nabla f$.

- **Definition:** The gradient is a column vector containing the partial derivatives of the function with respect to each parameter:
$$\nabla_\mathbf{x} f(\mathbf{x}) =
\begin{pmatrix}
\frac{\partial f}{\partial x_1} \\
\vdots \\
\frac{\partial f}{\partial x_n}
\end{pmatrix}
$$
- **Geometric Intuition:** While a 1D derivative tells you whether to move left or right on the x-axis, the gradient operates in a multi-dimensional domain. It acts as an arrow pointing in the **direction of the steepest ascent** (where the function increases the most). The length (norm) of the gradient vector indicates how steep that increase is.![](Images/Pasted%20image%2020260308231712.png)
- **Stationary Points:** If the gradient is a vector of zeros, the function is not growing in any direction. For convex functions like the MSE, a zero-gradient guarantees that we have found the global minimum: $\nabla_X f(X)=0$.

### 5. Distances and $L_p$​ Norms

To measure the magnitude of errors or parameters, we use distance metrics. The standard Euclidean distance is generalized as the $L_p$​ distance.

- **Formula:** $$d(x,y)=||x-y||_p\triangleq (\sum_{i=1}^{k}|x_i - y_i|^p)^\frac{1}{p}$$The absolute value is crucial to prevent negative distances when using odd powers for $p$.
- **Length or Norm:** the length or norm of a vector is simply its distance from the origin: $$||\mathbf{x} - O||_2=||\mathbf{x}||_2=\sqrt{\sum_{i=1}^k|x_i|^2}=\sqrt{\mathbf{x}^\intercal \mathbf{x}}$$
- **Geometric Shapes of** $L_p$​ **"Circles":** The shape of points at a distance of $1$ from the origin changes based on $p$:
    - $L_2$​ **(Euclidean):** Forms a standard round circle.
    - $L_1$​ **(Manhattan):** Forms a diamond or rhombus shape. For example, $(0.5,0.5)$ has an $L_1$​ distance of $1$ from the origin ($∣0.5∣+∣0.5∣=1$).
    - $L_\infty$​ **(Max norm):** As $p\rightarrow\infty$, the shape approaches a perfect square, mathematically equivalent to taking the maximum of the coordinates.

![](../../Pasted%20image%2020260309120748.png)

### 6. Matrix Notation & The Closed-Form Solution

In Deep Learning, tracking individual equations is inefficient. We can compactly represent all the equation ($y_i=ax_i+b$) at once using matrix notation: $\mathbf{y}=\mathbf{X}\Theta$.
$$\underbrace{
\begin{pmatrix}
y_1 \\
y_2 \\
\vdots \\
y_n
\end{pmatrix}
}_{\mathbf{y}}
=
\underbrace{
\begin{pmatrix}
x_1 & 1 \\
x_2 & 1 \\
\vdots & \vdots \\
x_n & 1
\end{pmatrix}
}_{\mathbf{X}}
\underbrace{
\begin{pmatrix}
a \\
b
\end{pmatrix}
}_{\boldsymbol{\theta}}$$
- **MSE in Matrix Form:** Doing so the MSE can be expressed as the squared $L_2$​ distance  using matrix transposes: $$\ell(\theta)=||\mathbf{y}-\mathbf{X}\theta||_2^2=(\mathbf{y}-\mathbf{X}\theta)^\intercal(\mathbf{y}-\mathbf{X}\theta)$$
- **Expanding the Quadratic:** Expanding this product yields $$\ell(\theta)=\mathbf{y}^\intercal\mathbf{y}-2\mathbf{y}^\intercal\mathbf{X}\theta+\theta^\intercal\mathbf{X}^\intercal\mathbf{X}\theta$$ which breaks down into a constant term, a linear term in $\theta$, and a quadratic term in $\theta$.
- **Solving via Gradients:** Taking the gradient of this matrix expression with respect to $\theta$ (dropping the constant, dropping $\theta$ from the linear term, and reducing the quadratic term) and setting it to zero yields $$-2\mathbf{X}^\intercal\mathbf{y}+2\mathbf{X}^\intercal\mathbf{X}\theta$$
- **The Closed-Form Solution:** Setting the gradient to the zero vector gives $$\mathbf{X}^\intercal\mathbf{X}\theta=\mathbf{X}^\intercal\mathbf{y}$$Solving for $\theta$ yields:$$\theta=(\mathbf{X}^\intercal\mathbf{X})^{-1}\mathbf{X}^\intercal\mathbf{y}$$ The term $(\mathbf{X}^\intercal\mathbf{X})^{-1}\mathbf{X}^\intercal$ is known as the **pseudo-inverse**.
- **Why the Pseudo-Inverse?** We cannot simply invert $\mathbf{X}$ because our dataset matrix $\mathbf{X}$ is usually rectangular (more data points than features/parameters). The matrix $\mathbf{X}^\intercal\mathbf{X}$ is always square, allowing us to invert it to find the parameters that perfectly minimize the least-squares error.
- **General Case with Higher Dimensional Datapoints:** In reality, training data inputs and outputs are rarely single numbers; they are high-dimensional vectors (e.g., xi​ might be a grid of pixels, and yi​ might be a one-hot vector for classification probabilities). In this general case, the linear model becomes yi​=Axi​+b. We can stack these multi-dimensional inputs and outputs into large matrices. For X⊺, we append a column of 1s to account for the bias vector b, and the parameter matrix Θ groups together both the weight matrix A and the biases. The MSE loss function is then calculated using the matrix trace operator: ℓ(Θ)=∣∣Y⊺−X⊺Θ∣∣22​=tr(Y⊺Y)−2tr(YX⊺Θ)+tr(Θ⊺XX⊺Θ). Despite this increase in dimensionality and complexity, the mathematical structure remains consistent, and the closed-form pseudo-inverse solution retains exactly the same formula as the one-dimensional case.