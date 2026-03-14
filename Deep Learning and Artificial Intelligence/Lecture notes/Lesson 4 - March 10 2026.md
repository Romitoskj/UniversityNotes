# Overfitting and going nonlinear

## 1. Moving to Nonlinear Models: Polynomial Regression

While simple linear regression fits a straight line, we often encounter data that requires a non-linear hypothesis. Polynomial regression introduces non-linearity by using polynomial features of the data (e.g., $x^2$,$x^3$ ):$$y_i=b+\sum_{j=1}^ka_jx_i^j\text{ for all data points }i=1,\dots,n$$
### Linearity in Parameters vs. Data:
However, it is crucial to understand that this model is still **linear in its parameters** (the weights are just multiplied by these new features). Because of this polynomial regression can be expressed in matrix notation too:$$
\underbrace{
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
x_1^k & x_1^{k-1} & \cdots & x_1 & 1 \\
x_2^k & x_2^{k-1} & \cdots & x_2 & 1 \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
x_n^k & x_n^{k-1} & \cdots & x_n & 1
\end{pmatrix}
}_{\mathbf{X}}
\underbrace{
\begin{pmatrix}
a_k \\
a_{k-1} \\
\vdots \\
a_1 \\
b
\end{pmatrix}
}_{\boldsymbol{\theta}}$$So we can apply the same least-square approach as for linear regression, with the only requirement that $k<n$.

#### Features
The concept of evaluating data through mathematical transformations before applying weights introduces the term **"features"**—a foundational concept in deep learning where data points are stacked into a matrix of transformed coordinates. One feature is simply one column of $\mathbf{X}$.

### The Stone-Weierstrass Theorem
This theorem states that any continuous function on a compact (closed) interval can be approximated arbitrarily well by a polynomial of some order. This might suggest polynomial regression is all we ever need, but in practice, it suffers from severe limitations regarding generalization, regularization, and categorical predictions.

## 2. Underfitting, Overfitting, and Generalization

Finding the right model complexity (e.g., the degree of a polynomial regression) is an ongoing challenge in deep learning, as models must generalize to unseen data rather than just memorizing the training set.

>[!example] Fit data with different polynomial degrees
>![|650](../../Pasted%20image%2020260314140250.png)

- **Underfitting:** Occurs when a model is too simple (e.g., fitting a line to a curved distribution). It results in a large Mean Squared Error (MSE), which represents the vertical distance from the data points to the prediction. Underfitting yields high errors on both training and validation sets.

- **Overfitting:** Occurs when a model is too complex (e.g., a degree-15 polynomial for simple data). The model perfectly memorizes the training data—including the observation noise—driving the training error to near zero, but fails completely on new data. Adding complexity to a model is not necessarily a good thing.

- **Data Splits:** To detect these phenomena, data is strictly divided into three sets:
    1. **Training set:** Used to fit the model parameters.
    2. **Validation set:** Used to tune choices (like polynomial degree) and check for overfitting.
    3. **Test set:** Unseen data used only at the very end to evaluate true generalization. You must assume you have no access to it during training.

>[!Important] **K-Fold Cross-Validation:**
>A practical defense against overfitting the validation set itself. The training data is split into k subsets; the model is trained on k−1 subsets and validated on the remaining one, rotating through all combinations and averaging the errors.
>
>![|650](../../Pasted%20image%2020260314143339.png)
>
>It can be used to tune *hyperparameters*, e.g., k-fold cross-validation in polynomial regression with different degree and choose the run with the smallest MSE.


## 3. Regularization (Preview)

Regularization encompasses techniques intended to reduce the generalization error without explicitly reducing the training error, often by controlling model complexity.

- Instead of manually lowering a polynomial's degree, we can add a penalty to the loss function to constrain the parameters' growth.
- **L2 Regularization (Weight Decay/Ridge):** Adds the squared Frobenius norm of the parameters (λ∣∣Θ∣∣F2​) to the loss. It shrinks large weights proportionally, distributing the penalty smoothly.
- **L1 Regularization (Lasso):** Adds an absolute value penalty (λ∣Θ∣1​). This applies a constant push toward zero, forcing many parameters exactly to zero, which induces **sparsity** (effectively selecting only the most important features).
- Both L1​ and L2​ norms are convex, meaning adding them to a convex loss like MSE preserves the mathematical guarantees of finding a global optimum.

## 4. Classification and Logistic Regression

Standard polynomial or linear regression fails for classification tasks (e.g., identifying a tumor as malignant or benign ). Regression models output continuous numbers (like 42.5), whereas classification requires discrete categorical outputs or bounded probabilities.

- **The Logistic Sigmoid (**σ**):** To constrain predictions, we apply a non-linear activation function called the logistic sigmoid: σ(x)=1+e−x1​. This "squashes" any real-numbered output of the linear model (ax+b) into a range between 0 and 1, making it interpretable as a probability.
- **The Problem with MSE:** If we use the Mean Squared Error loss with the non-linear sigmoid function, the resulting loss landscape becomes **non-convex**. This destroys our ability to easily find a global minimum.

## 5. Cross-Entropy Loss (Log-Loss)

To restore convexity and heavily penalize wrong, confident predictions, we replace MSE with the Binary Cross-Entropy loss.

- **Piecewise Definition:** For a single data point, the loss is defined as −ln(σ(axi​+b)) if the true label is 1, and −ln(1−σ(axi​+b)) if the true label is 0.
    - If the true class is 1 and the model predicts 1, −log(1)=0 (no penalty).
    - If the true class is 1 but the model predicts 0, −log(0)→∞ (massive penalty).
- **Unified Expression:** Since the ground truth y is strictly 0 or 1, we can combine the pieces mathematically using a convex combination: −yln(p)−(1−y)ln(1−p), where p is the model's predicted probability.
- **Multi-Class Generalization (Log-Probabilities):** The loss can be interpreted as the inner product between the ground truth distribution (a "one-hot" vector where the correct class is 1 and all others are 0) and the predicted log-probabilities of the model. Because the one-hot vector zeros out everything except the correct class, the model only minimizes the loss for the correct answer, largely ignoring the distribution of uncertainty among the incorrect classes.

## 6. Optimization Challenges

Because logistic regression utilizes the sigmoid and logarithmic functions, the model parameters enter the gradient equations in a highly nonlinear way.

- **The Chain Rule:** Computing the gradient of the cross-entropy loss requires rigorously applying the chain rule (differentiating the log, then the sigmoid, then the linear component).
- **No Closed-Form Solution:** The resulting derivatives form a nonlinear system of **transcendental equations**. Unlike linear regression, which can be solved analytically with the pseudo-inverse, logistic regression requires iterative **nonlinear optimization** techniques, setting the stage for algorithms like Stochastic Gradient Descent.