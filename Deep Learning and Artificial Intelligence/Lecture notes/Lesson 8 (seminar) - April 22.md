# Quantization

### **1. The Compute vs. Memory Bottleneck**

To understand why we compress neural networks, we must first understand how hardware processes them.

![](Images/Pasted%20image%2020260503181528.png)

- **Matrix Multiplication (Matmul) & MACs:** A neural network's forward pass is overwhelmingly dominated by matrix multiplications. At the hardware level, these are broken down into **Multiply and Accumulate (MAC)** operations (a scalar multiplication followed by a scalar addition).
- **CPUs vs. GPUs:** CPUs can only process these dot products sequentially or with very limited parallelization. GPUs are fundamentally faster because they feature thousands of CUDA cores (and modern Tensor Cores) that compute tens of thousands of MAC operations simultaneously.
- **The Real Cost (Data Movement):** While computational operations (the actual math) are incredibly cheap, **moving the data from the memory (DRAM) to the processor core (SRAM/Registers) is extremely expensive**—often costing roughly 200 times more time and energy than the calculation itself.
- **The Goal of Compression:** Because data movement is the bottleneck, we intentionally spend a little bit of cheap compute power to compress the data, making it much more affordable to move around.

### **2. The Two Levers of Compression**

The "cost" of storing and moving a network is determined by a simple formula: **$\text{Cost} = \text{Number of Values} \times \text{Number of Bits per Value}$**. To compress a model, we can pull two different historical levers:

1. **Reduce the number of values:** The **Pruning** era (dominant up to 2018).
2. **Reduce the bits per value:** The **Quantization** era (dominant from 2018 onward).

### **3. Pruning (Reducing the Number of Values)**

Pruning removes redundant weights entirely from the network.

![](Images/Pasted%20image%2020260503183956.png)

- **The Pipeline (Han et al., 2015):** The standard process is to 
	1) Train the network from scratch
	2) Apply a threshold and zero-out any weights that fall below it
	3) Freeze those zeroed-out connections so gradients don't pass through them
	4) **Retrain (fine-tune) the remaining weights**.
- **Network Co-adaptation:** If you prune 90% of a network's weights, the accuracy drops severely. However, by fine-tuning the remaining 10%, the network's remaining weights **co-adapt** to the damage, remarkably restoring the accuracy back to near-original levels.
- **Denoising Effect:** In some cases, light pruning actually improves the baseline accuracy. By restricting the network's capacity, pruning inadvertently acts as a regularizer, discarding noise.

![](Images/Pasted%20image%2020260503185454.png)

### **4. K-Means Quantization (Storage Compression)**

Quantization fundamentally means mapping a large continuous space (like 32-bit real numbers) down to a discrete, smaller set of values (like integers), which inherently introduces a small **quantization error**. The first major approach was K-Means Quantization (Han et al., 2016).

- **How it Works:** The algorithm groups the network's thousands of unique 32-bit floating-point weights into a small number of clusters ($k$). The heavy float weights are replaced by tiny integer indices (e.g., a 2-bit integer) that point to a "codebook" containing the 32-bit cluster centroids.
- **Training the Centroids:** During backpropagation, the gradients are grouped by their cluster assignments and reduced (e.g., summed) to fine-tune the 32-bit centroids.
- **The Limitation:** While this provides immense **storage compression** (sometimes shrinking models by 40x when combined with pruning), it **does not compress the compute**. The processor must still look up the 32-bit float centroid and perform an expensive floating-point multiplication.

![](Images/Pasted%20image%2020260503185936.png)

### **5. Linear Affine Quantization (Compute Compression)**

To truly speed up a neural network, we must compress the computation itself by executing operations entirely in integer math (Jacob et al., 2018).

- **Why Integer Math?:** Using lower-bit integer types is drastically cheaper in energy. For instance, an 8-bit integer addition costs about **30x less energy** than a 32-bit float addition, and an 8-bit integer multiplication costs **16x less energy**.
- **The Quantization Formula:** To map an original 32-bit float value ($r$) to a low-bit integer ($q$), the following linear transformation is applied: $$q = \text{clip}\left(\text{round}\left(\frac{r}{S} + Z\right), q_{\min}, q_{\max}\right)$$
- **Finding the Scale ($S$):** The scale parameter determines the step size that maps the floating-point range to the integer range. $$S = \frac{r_{\max} - r_{\min}}{q_{\max} - q_{\min}}$$
- **Finding the Zero Point ($Z$):** The zero point acts as a shift to guarantee that a real value of exactly $0.0$ translates perfectly to an integer, which is vital for mathematical stability. $$Z = \text{round}\left(q_{\min} - \frac{r_{\min}}{S}\right)$$
- **Dequantization:** You can reconstruct a rough approximation of the original floating-point weight by applying the inverse: $r \approx S \times (q - Z)$. However, due to the rounding step, the reconstructed value will not match the original exactly, representing the quantization error.