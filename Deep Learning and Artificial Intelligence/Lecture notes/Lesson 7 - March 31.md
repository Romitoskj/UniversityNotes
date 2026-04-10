# Convolutional Neural Networks

## 1. The Motivation: Why CNNs?

While standard Multi-Layer Perceptrons (MLPs) are theoretically universal approximators, applying them directly to highly complex, high-dimensional data like images leads to millions of parameters, severe optimization difficulties, and massive overfitting. To solve this, we must inject **priors** into our models—meaning we build networks that inherently understand the structure of the data they are processing.

Despite feeling like '90s technology, CNNs remain a foundational building block in state-of-the-art models today, including diffusion models for image generation and modern audio/music generation systems,.

## 2. Structural Priors in Natural Data

Real-world data, unlike pure random "salt and pepper" noise, contains highly predictable underlying structures that CNNs exploit:

- **Smooth Gradients:** Natural images tend to have smooth transitions between adjacent pixels (e.g., the sky), with sharp edges being relatively rare,.
- **Self-Similarity and Repetition:** Specific patterns (like striped textures, corners, or human faces) repeat across different locations within the same image and across a dataset,,,. In music, this self-similarity appears as oscillating waves and repeating refrains.
- **Translation Invariance:** The identity of an object does not change if it is shifted in space. A cat in the bottom-right corner is the same as a cat in the top-left,. While you can teach an MLP this via data augmentation, it is vastly more efficient to design an operator that is invariant to translation by construction.
- **Hierarchy and Compositionality:** Local, low-level features (like edges and curves) compose together to form mid-level features (wheels, eyes), which compose into high-level semantic features (cars, faces),,.

## 3. The Convolution Operator

To leverage these priors, CNNs replace standard fully connected matrix multiplications with the **convolution** operator.

- **Definition:** In a continuous 1D space, the convolution of an input function f and another function g involves three steps: **flipping** g backward, **shifting** it along the axis, and computing the inner product (the integral of their element-wise multiplication) at each shifted position,,. _Note: If you perform this operation without the flip, it is formally called cross-correlation__._
- **Terminology:** In deep learning, f is the input data, g is the learnable **filter** (or kernel), and the resulting output is called a **feature map**,.
- **Key Mathematical Properties:**
    - **Commutativity:** Convolution is commutative (f∗g=g∗f). Shifting and flipping f yields the exact same result as shifting and flipping g,,.
    - **Linearity:** Convolution is a linear operator with respect to the input function. Convolving a scaled or summed input is equal to scaling or summing the convolved outputs,.
    - **Shift-Equivariance:** If you translate the input and then apply convolution, it is exactly the same as applying convolution first and then translating the output,. This property is called **equivariance** (the operators commute) and must not be confused with strict **invariance** (where the translation would disappear from the output entirely),.

## 4. Discrete Convolution in Deep Learning

When working with digital images, convolution becomes a discrete mathematical operation:

- **The Moving Window:** For 2D data, the filter g acts as a small sliding window (e.g., 3x3 pixels) that moves across the input grid, computing inner products at each step,.
- **Boundaries and Padding:** As the filter slides to the edge of the input, the operation may become undefined. To fix this, we use **padding**, which involves injecting artificial pixels around the boundary. The most common choice is **zero-padding**, though cyclic or mirrored padding can be used if appropriate,,.
- **Output Dimensions:** A common misconception is that convolution always shrinks the data. Depending on the padding size, the filter size, and the **stride** (the jump size between sliding steps), the resulting feature map can be smaller, exactly the same size, or even larger than the input,,,.
- **Multi-Channel Filters:** A single convolutional layer rarely uses just one filter. Instead, it stacks multiple different filters (e.g., one for detecting horizontal edges, one for colors) to produce a multi-channel, high-dimensional feature map,,.

## 5. Why CNNs beat MLPs (Weight Sharing & Sparse Interactions)

CNNs are vastly more efficient than MLPs due to two architectural features that emerge directly from the convolution operator:

- **Sparse Interactions:** In an MLP, every output node connects to every single input pixel. In a CNN, an output pixel is calculated using only a tiny local patch of the input (e.g., 9 pixels for a 3x3 filter). This captures local semantic features effectively,,.
- **Weight Sharing:** The weights of the small filter are fixed and reused across the entire image. Therefore, the parameter count of a convolutional layer depends _only_ on the size and number of the filters, completely independent of the input image size,,,.

## 6. Pooling Layers

To incrementally reduce the spatial size of the feature maps, and to reduce the computational burden on deeper layers, CNNs regularly interleave convolutional layers with **pooling layers**,.

- **The Operation:** Pooling divides the feature map into small patches and replaces each patch with a summary statistic, most commonly using **Max Pooling** (keeping the highest value) or Average Pooling,.
- **Properties:** The mathematical operations used inside a pooling patch are deliberately chosen to be **permutation invariant** (e.g., the maximum value is the same regardless of the pixels' specific layout inside the 2x2 patch).
- **Purpose:** Beyond downsampling, pooling acts as an infinitely strong prior that forces the network to become strictly invariant to small, local translations, discarding exact positional data in favor of detecting whether a feature is simply "present".