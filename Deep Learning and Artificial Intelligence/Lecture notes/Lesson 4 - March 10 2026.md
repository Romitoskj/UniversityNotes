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
>![|650](Images/Pasted%20image%2020260314140250.png)

- **Underfitting:** Occurs when a model is too simple (e.g., fitting a line to a curved distribution). It results in a large Mean Squared Error (MSE), which represents the vertical distance from the data points to the prediction. Underfitting yields high errors on both training and validation sets.

- **Overfitting:** Occurs when a model is too complex (e.g., a degree-15 polynomial for simple data). The model perfectly memorizes the training data—including the observation noise—driving the training error to near zero, but fails completely on new data. Adding complexity to a model is not necessarily a good thing.

- **Data Splits:** To detect these phenomena, data is strictly divided into three sets:
    1. **Training set:** Used to fit the model parameters.
    2. **Validation set:** Used to tune choices (like polynomial degree) and check for overfitting.
    3. **Test set:** Unseen data used only at the very end to evaluate true generalization. You must assume you have no access to it during training.

>[!Important] **K-Fold Cross-Validation:**
>A practical defense against overfitting the validation set itself. The training data is split into k subsets; the model is trained on k−1 subsets and validated on the remaining one, rotating through all combinations and averaging the errors.
>
>![|650](Images/Pasted%20image%2020260314143339.png)
>
>It can be used to tune *hyperparameters*, e.g., k-fold cross-validation in polynomial regression with different degree and choose the run with the smallest MSE.

## 3. Classification and Logistic Regression

Standard polynomial or linear regression fails for classification tasks (e.g., identifying a tumor as malignant or benign ). Regression models output continuous numbers, whereas classification requires discrete categorical outputs or bounded probabilities.

- **The Logistic Sigmoid ($\sigma$):** To constrain predictions, we apply a non-linear activation function called the logistic sigmoid: $$\sigma(x)=\frac{1}{1+e^{-x}}$$This "squashes" any real-numbered output of the linear model ($ax+b$) into a range between $0$ and $1$, making it interpretable as a probability.

  ![](../../Pasted%20image%2020260314235926.png)

- **The Classification Model and Prediction Formula:** For a binary classification task where the true labels $y_i\in\{0,1\}$, the base linear model is defined as $\hat{f}​(x_i​)=ax_i​+b$. To actually make a categorical prediction $\hat{y_i}$​, the model applies a threshold to the sigmoid output:
    - It predicts $1$ **(Positive class) if** $\sigma(ax_i​+b)>0.5$.
    - It predicts $0$ **(Negative class) if** $\sigma(ax_i​+b)≤0.5$.

- **The New MSE with Sigmoid:** To train this classification model, we initially attempt to adapt the standard Mean Squared Error (MSE) loss function by placing the sigmoid transformation directly inside it. This yields the new loss formula: $$\ell_\Theta(\{x_i,y_i\})=\sum_{i=1}^n(y_i-\sigma(ax_i+b))^2$$
- **The Problem with MSE:** If we use the Mean Squared Error loss with the non-linear sigmoid function, the resulting loss landscape becomes **non-convex**. This destroys our ability to easily find a global minimum.

## 4. Cross-Entropy Loss (Negative Log Likelihood)

To restore convexity and heavily penalize wrong, confident predictions, we replace MSE with the Binary Cross-Entropy loss.

- **Piecewise Definition:** For a single data point, the loss is defined as$$c(x_i, y_i) = 
\begin{cases} 
-\ln(\sigma(ax_i + b)), & y_i = 1 \\ 
-\ln(1 - \sigma(ax_i + b)), & y_i = 0 
\end{cases}$$
    - If the true class is $1$ and the model predicts $1$, $−\ln(1)=0$ (no penalty).
    - If the true class is $1$ but the model predicts $0$, $−\ln(0)\rightarrow\infty$ (massive penalty).
    - This remains the same for true class equal to $0$

- **Unified Expression:** Since the ground truth y is strictly 0 or 1, we can combine the pieces mathematically using a convex combination: −yln(p)−(1−y)ln(1−p), where p is the model's predicted probability.

- **Multi-Class Generalization (Log-Probabilities):** The loss can be interpreted as the inner product between the ground truth distribution (a "one-hot" vector where the correct class is 1 and all others are 0) and the predicted log-probabilities of the model. Because the one-hot vector zeros out everything except the correct class, the model only minimizes the loss for the correct answer, largely ignoring the distribution of uncertainty among the incorrect classes.

## 5. Optimization Challenges

Because logistic regression utilizes the sigmoid and logarithmic functions, the model parameters enter the gradient equations in a highly nonlinear way.

- **The Chain Rule:** Computing the gradient of the cross-entropy loss requires rigorously applying the chain rule (differentiating the log, then the sigmoid, then the linear component).
- **No Closed-Form Solution:** The resulting derivatives form a nonlinear system of **transcendental equations**. Unlike linear regression, which can be solved analytically with the pseudo-inverse, logistic regression requires iterative **nonlinear optimization** techniques, setting the stage for algorithms like Stochastic Gradient Descent.