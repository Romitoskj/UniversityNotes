# PCA and VAEs

## **1. Dimensionality Reduction and PCA**

*   **The Goal:** Dimensionality reduction transforms data from a high-dimensional space ($d$) into a low-dimensional representation ($k$) while retaining the most meaningful properties of the original data. It is crucial for visualization, denoising, and outlier detection.
*   **Principal Component Analysis (PCA):** A linear dimensionality reduction technique that identifies the orthogonal directions (principal components) where the data varies the most. 
*   **Mathematical Mechanism:** PCA projects data points onto these principal components. The optimal components are found by minimizing the **reconstruction error** (the distance between the original data and the data reconstructed from the lower-dimensional code), which is mathematically equivalent to maximizing the variance of the projected data. The solution is given globally by the first $k$ eigenvectors of the data's covariance matrix.
*   **PCA as a Generative Model:** PCA can act as the simplest parametric generative model. You can sample a new low-dimensional code $z_{new}$ and multiply it by the principal component matrix $W$ to generate a new, high-dimensional data point $x_{new}$. 

## **2. Autoencoders & The Manifold Hypothesis**

While PCA is limited by its linear nature, **Autoencoders** generalize this concept by replacing linear encoding and decoding steps with Deep Neural Networks, which can approximate any non-linear function.
*   **Architecture:** An autoencoder consists of an **Encoder** network that compresses the input into a lower-dimensional latent code $z$, and a **Decoder** network that attempts to reconstruct the original input from $z$. 
*   **The Bottleneck:** The lower-dimensional latent space acts as a bottleneck. Because the model cannot simply copy the input perfectly, it is forced to prioritize and learn the most important, structural properties of the data to minimize the reconstruction loss.
*   **The Manifold Hypothesis:** High-dimensional data (like images) does not fill the ambient space randomly but is constrained to a lower-dimensional, curved topological structure called a **manifold**. 
*   **Decoders as Charts:** In differential geometry, a manifold is mapped using "coordinate charts". The decoder of an autoencoder acts as a parametric chart, learning a differentiable mapping from the flat, low-dimensional Euclidean latent space to the curved manifold of the data space.

## **3. Audio Integration: Sparsity and "Sparse Encoders"**

The lecture audio connects the concept of **regularization** directly to autoencoder architectures. 
*   While autoencoders learn dense compressed representations, the audio notes that applying an **L1 penalty** (which pushes parameters to exactly zero) promotes **sparsity**. 
*   **Sparse Encoders:** The audio highlights that modern models (specifically mentioning 2025 advancements) utilize "sparse encoders". By enforcing L1 regularization, these encoders perform automatic **feature selection**, capturing complex data phenomena with only a few active features. This sparsity makes the resulting models highly interpretable and vastly easier to compress.

## **4. Variational Autoencoders (VAEs)**

A fundamental limitation of standard autoencoders is that they do not guarantee a smooth or well-organized latent space, making them poor generators for *new* data. **Variational Autoencoders (VAEs)** solve this by using probability distributions.
*   **Probabilistic Encoders:** Instead of outputting a single deterministic vector $z$, the VAE encoder outputs the parameters (mean $\mu$ and variance $\sigma$) of a multivariate Gaussian distribution $q_\phi(z|x)$ for a given input. The latent code $z$ is then sampled from this distribution.
*   **The Regularizer (KL Divergence):** To ensure the latent space is completely smooth and compact, a prior is enforced: the latent distributions must generally resemble a standard normal Gaussian distribution $N(0, I)$. This is achieved by adding the **Kullback-Leibler (KL) divergence** to the loss function, which penalizes the encoded distributions if they deviate too far from $N(0, I)$.
*   **The ELBO:** Training a VAE involves maximizing the Evidence Lower Bound (ELBO), which mathematically translates to the combined VAE loss function: **Loss = Reconstruction Error + KL Divergence Regularizer**.

## **5. The Reparametrization Trick**

To train a VAE via Stochastic Gradient Descent, gradients must backpropagate through the entire network. 
*   **The Problem:** The act of randomly sampling $z$ from the encoder's distribution creates a random node in the computational graph, which is non-differentiable and blocks backpropagation.
*   **The Solution:** The **reparametrization trick** removes the randomness from the main network path. The code $z$ is computed deterministically using the encoder's outputs ($\mu$ and $\sigma$) combined with an external, independent source of random noise $\epsilon \sim N(0, I)$ via the formula **$z = \mu + \sigma \odot \epsilon$**. This allows gradients to safely bypass the random noise and update the network parameters.

## **6. Generation, Interpolation, and Disentanglement**

*   **Generation:** At test time, the encoder is completely discarded. To generate new data, you simply sample a random vector $z$ directly from the prior $N(0, I)$ distribution and feed it into the decoder. 
*   **Interpolation:** Because VAEs force the latent space to be continuous and smooth, you can pick two real images, find their latent codes $z_1$ and $z_2$, and mathematically draw a line between them. Decoding the points along this line will yield smooth, realistic transitional images.
*   **Disentanglement:** A well-trained VAE can sometimes map specific, human-understandable traits to specific dimensions in the latent space (e.g., one dimension controls "pose", another controls "identity"), allowing for highly controlled generation.