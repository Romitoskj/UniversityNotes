# Regularization

## 1. Regularization Fundamentals: $L_1$ and $L_2$ Penalties

Regularization encompasses techniques intended to reduce a model's generalization error on unseen data, without necessarily reducing its training error. This prevents **overfitting**, which occurs when a model has excessive representational capacity compared to the training examples.

Therefore, many regularization techniques are aimed at reducing the number of free parameters of a model, to limit its representational power. This is done by constraining the parameters of the model to behave in a specified way, limiting their freedom.

One explicit way to regularize is by adding a penalty term to the loss function:$$\underbrace{\ell(\boldsymbol{\Theta})}_{\text{loss}} + \lambda \underbrace{\rho(\boldsymbol{\Theta})}_{\text{regularizer}}$$Many regularizers enforce some form of norm penalty, trying to penalize the weights for growing too much. The optimizer will understand (from the gradient) that decreasing the very large parameters will lead to a large decrease in the loss function, even if at the expense of some fitting error, and will change the parameters accordingly.
### $L_2$ Penalty (Tikhonov / Shrinkage):

Adds the squared magnitude of the parameters to the loss.
- _Mechanism:_ It penalizes values greater than 1 quadratically while applying very little penalty to small values near zero.
- _Effect:_ It produces a **dense solution** where weights are squashed into a bounded range (e.g., between -1 and 1) but rarely exactly zero. This increases optimization stability by preventing the wild parameter growth often seen in overfitted polynomial models.

![](Images/Pasted%20image%2020260503213226.png)


### $L_1$ Penalty (Lasso / Sparsity):

Adds the absolute value of the parameters to the loss.
- _Mechanism:_ It applies a constant proportional penalty to all weights regardless of their size.
- _Effect:_ It encourages a highly **sparse solution**, driving a large percentage of parameter values exactly to zero.
- _Benefits:_ Sparsity is excellent for compressing the model, improving interpretability, and performing **automatic feature selection** (an idea leveraged in modern "sparse encoders").

![](Images/Pasted%20image%2020260503213253.png)

## **2. Early Stopping and Double Descent**

- **Early Stopping:** Typically, as training progresses, training error continually decreases, but validation error eventually starts to rise (forming a U-shape) as the model begins memorizing noise. Early stopping halts training as soon as this validation error increases to prevent overfitting. 
	
	![516](Images/Pasted%20image%2020260503231322.png)
	
	- It is important to say that it is not always true that more parameters lead to overfitting (like for polynomial regression).
	 ![](Images/Pasted%20image%2020260503232032.png)
	 
	- *Patience:* The number of steps (or epoch) we allow the validation to increase, since it can grow because of noise. Every time the validation loss is smaller, we save the weights. Instead, if the validation loss has not decreased by some some amount $\Delta\ell$ after as many epoch as defined by patience, we stop training and we will used the last saved weights.
	 
	- _Early stopping as regularizer:_ Training starts with small weights that grow larger, by strictly cutting off the training time, early stopping is cutting off the optimizer's ability to inflate the weights. It forces the network to settle for the smaller, simpler weights it had halfway through the process and act similar to $L_2$ regularization.

- **Double Descent:** A highly surprising phenomenon where increasing the capacity (number of parameters $\rightarrow$ the set of function the network can represent well) of the network (or training time) eventually causes the validation error to drop again _after_ the initial overfitting peak.
	
    - As you add parameters, you increase the size of the function class the network can represent.
	- At some point, the capacity is so large that there exists a function that perfectly describes the data, so the training error goes to zero, leading to overfitting. 
	- The *sweet spot* is were the capacity of the model is optimal, meaning it contains functions that describes both the training and validation data well enough.
	- If we keep increasing the capacity the validation error will start decreasing again (**double descent**).
	 ![](Images/Pasted%20image%2020260504163330.png)
	
	- It starts to happen where training loss reaches $0$ and eventually the capacity will become so large that will also fit the validation data perfectly.
	
    - _Epoch-wise Double Descent:_ Research has shown this double U-shape occurs not just when scaling the number of parameters (capacity), but also when extending the training time (epochs):
 ![](Images/Pasted%20image%2020260504164919.png)

## **3. Batch Normalization**

Deep networks suffer from **internal covariate shift**: as parameters update during training, the statistical distribution of the inputs to each subsequent layer continuously changes, making training noisy and difficult.

- **The Operation:** Batch Normalization fixes this by standardizing the inputs *at each layer*, in order to obtain a mean of $0$ and variance of $1$. For every mini-batch, it computes the mean and standard deviation of the features across the batch, subtracts the mean, and divides by the standard deviation:$$\mathbf{x} \mapsto \frac{\mathbf{x} - \mathrm{E}[\mathcal{X}]}{\sigma(\mathcal{X})}$$A small constant ($10^{-5}$) is added to the variance to prevent division by zero.

- **Learnable Parameters ($\gamma$ and $\beta$):** Normalization might accidentally limit the network by removing its ability to represent the identity function. To fix this, two trainable weights (scale $\gamma$ and shift $\beta$) are introduced to restore the network's representational power, by cancel out normalization: $$\mathbf{x} = \gamma \left( \frac{\mathbf{x} - \mathrm{E}[\mathcal{X}]}{\sigma(\mathcal{X})} \right) + \beta$$Since $\beta$ acts as a shift, standard bias terms in the preceding linear layer become redundant and are usually removed.

- **Regularization Effect:** Because a sample might end up in a different mini-batch in every epoch, its normalization shifts slightly based on the other images in the batch. This stochastic uncertainty acts as a built-in regularizer.

- **Inference Time:** During inference, you typically process one image, not a batch. To handle this, the network records an **exponential moving average** of the mini-batch means and variances during training and uses these fixed statistics at test time.

- _Note:_ Other variants exist, such as Layer Normalization (which computes statistics across the channels of a single sample instead of across a batch), which is famously used in Transformers where the concept of batches doesn't apply well.

## **4. Dropout**

Dropout brings the power of **ensemble learning** (like Random Forests) to deep learning in a computationally feasible way.

- **Mechanism:** During training, random portions of the network are temporarily removed ("dropped out") for each mini-batch. This means the model visits a vast amount of different subnetworks, approximating a massive ensemble. Because all these subnetworks are part of the same base architecture, it effectively combines "bagging" with **weight sharing**.
- **Reducing Co-adaptation:** By making neurons randomly disappear, the network cannot rely too heavily on any single neuron to fix the mistakes of previous ones. This breaks co-adaptation and forces the network to learn robust, generalized features.
- **Inference Time:** At test time, no neurons are dropped. Instead, the output of a neuron is scaled down by the probability $p$ that it was dropped during training, compensating for the fact that all neurons are suddenly active.
- **Theory vs. Practice (PyTorch Implementation):** The audio lecture points out a crucial conflict between the original research paper and actual software implementation. While the theory dictates zeroing out entire _rows_ of the linear layer's weight matrix (effectively removing a node), PyTorch explicitly performs the full matrix multiplication and instead zeroes out random dimensions in the **output representation** (the resulting feature map) because it is vastly more computationally efficient.