# Self-attention and transformers

### **1. Sequential Data and Task Paradigms**

While text is the most common example of sequential data, sequences appear across many domains, such as 1D time series (e.g., financial market trends), 3D spatial trajectories (e.g., Brownian motion of a molecule), or complex objects like 3D shape motions.

Crucially, working with sequences does **not** exclusively mean generating or predicting the next token. While autoregressive generation is popular, sequential tasks also include **classification** (e.g., categorizing a 3D motion as "running") and **interpolation** (e.g., filling in missing poses in computer graphics). For text, sequences are defined by establishing a dictionary of symbols (tokens) such as letters, words, or syllables.

### **2. Defining Sequence-to-Sequence (Seq2Seq) Models**

A true seq2seq model possesses one fundamental property: **the same weights must apply to sequences of any arbitrary length**.

- **Why MLPs fail:** A standard Multi-Layer Perceptron (MLP) cannot be a seq2seq model because its initial linear layer rigidly expects a fixed input dimension. If trained on sequences of length 20, it cannot process a sequence of length 19 or 21 at inference time.
- **CNNs as Seq2Seq:** A purely convolutional neural network (without a flattening operation and dense layers at the end) naturally acts as a seq2seq model because the sliding filter weights are independent of the input sequence's length.

### **3. Causality, Autoregressive Modeling, and Teacher Forcing**

When designing a seq2seq architecture, one must decide if the layers should be causal or non-causal:

- **Non-Causal Layers:** The prediction for a token at position $i$ can look at the entire sequence (past and future). This is ideal for classification or translation.
- **Causal Layers:** The prediction for a token at position $i$ is strictly restricted to looking only backward at previous tokens. This is mandatory for generation to prevent the model from "cheating" by looking ahead.

**Autoregressive modeling** uses causal architectures to predict the next token in a sequence.

- **Training (Teacher Forcing):** During training, the model does not generate the sequence iteratively. Instead, the entire sequence is processed simultaneously in a single shot. The model attempts to predict the shifted sequence, and its predictions are compared to the actual ground truth (Teacher Forcing) using cross-entropy loss.
- **Inference (Sequential Sampling):** At inference, the process is iterative. Given a seed sequence, the model outputs a **probability distribution** over the entire dictionary. Instead of just picking the most likely word, the algorithm samples from this distribution, appends the chosen token to the seed, and repeats. This sampling introduces diversity, allowing the model to generate novel sequences (generative modeling) rather than perfectly memorizing training data.

### **4. The Core Self-Attention Mechanism**

Self-attention computes an output token $y_i$ as a **linear combination of all input tokens** $x_j$.

- **The Unnormalized Scores ($W'$):** The relevance of token $j$ to token $i$ is determined by their inner product: $w'_{ij} = x_i^\top x_j$. Conceptually, this measures how much the two tokens have in common or how much influence one should have over the other. In matrix notation, this is $W' = X^\top X$.
- **The Softmax Transformation ($W$):** To turn these raw inner products into valid combination weights, a row-wise **Softmax** is applied: $w_{ij} = \frac{e^{w'_{ij}}}{\sum_j e^{w'_{ij}}}$. This differentiable operation turns the weights into positive numbers that sum to 1 (a convex combination), effectively amplifying the largest values and suppressing the others. W becomes a stochastic matrix where each row is a probability distribution.
- **The Memory Bottleneck:** Because the attention matrix calculates the interaction between every single pair of tokens, its size scales quadratically with the sequence length ($N \times N$). This massive memory overhead is the primary reason why large language models cannot process infinitely long context windows on standard hardware.
- **Causal Masking:** To make self-attention causal, the model simply "masks" out future tokens. Before the Softmax is applied, the inner products for any token $j > i$ are set to a massive negative number (effectively zeroing them out in the exponential), ensuring the output only sums over previous tokens.

### **5. Trainable Attention: Queries, Keys, and Values**

Raw self-attention simply multiplies the input embeddings together, which lacks learnable parameters. To make the mechanism trainable, the model assigns three different functional roles to the tokens:

1. **Query ($q_i$):** What the current token is "looking for."
2. **Key ($k_j$):** What the other tokens "contain."
3. **Value ($v_j$):** The actual content that will be combined to form the output.

Instead of directly computing $x_i^\top x_j$, the model applies three small trainable linear transformations to $X$ to compute $Q$, $K$, and $V$. The attention scores are computed using $Q$ and $K$, and the final output is a weighted sum of $V$.

### **6. Permutation Equivariance and Positional Encoding**

A critical mathematical quirk of standard self-attention is that it is strictly **permutation-equivariant**. If you shuffle the input sequence, the output tokens are identical, just shuffled in the exact same order. The mechanism treats the sequence as a **"set"**, completely blind to the original temporal ordering.

To fix this and inject sequential awareness, models add **Positional Information** to the input vectors before they enter the attention layer.

- **Position Embedding:** Learning a unique vector for position 1, position 2, etc., and summing it with the token's embedding.
- **Position Encoding:** Defining the positions _a priori_ using mathematical rules (like alternating sine and cosine functions) so the model can inherently understand absolute and relative distances.

### **7. Transformer Architecture Design**

A Transformer is loosely defined as any model that primarily relies on self-attention across tokens. The standard Transformer block wraps the self-attention layer with residual connections, feed-forward networks, and normalization.

- **Layer Normalization vs. Batch Normalization:** Transformers use Layer Normalization rather than Batch Normalization. A mini-batch of text contains unrelated, fragmented sentences. Normalizing across the batch axis would mathematically entangle an input sequence with completely unrelated sequences, breaking the logic of the model.
- **Encoder-Only Models:** Use non-causal self-attention and typically culminate in a pooling layer for classification (e.g., sentiment analysis on a full sentence).
- **Decoder-Only Models:** Use causal self-attention to iteratively generate outputs (e.g., standard autoregressive models like GPT).
- **Encoder-Decoder Models:** The original Transformer architecture. An encoder processes a context sequence (like a prompt or an image) and creates a compact representation. A decoder then generates the output, using "cross-attention" to constantly condition its generation on the encoder's representation.