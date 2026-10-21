1. Foundations of Sequence-to-Sequence Architecture and the Bottleneck Dilemma

Historical Evolution of Sequence Processing

The paradigm of sequence processing in deep learning experienced a major evolutionary shift when moving from standard recurrent architectures to Encoder-Decoder frameworks. Standard Recurrent Neural Networks (RNNs) initially established the capability to process sequential data, yet they lacked a flexible structural mechanism to map input sequences to output sequences of differing lengths.

The introduction of the Sequence-to-Sequence (Seq2Seq) Encoder-Decoder architecture (Sutskever et al., 2014) solved this structural limitation, enabling breakthroughs in complex sequence conversion tasks such as machine translation (e.g., mapping an English sequence to a Spanish target). However, these early Seq2Seq frameworks introduced a severe structural bottleneck: they compressed variable-length source sequences into a single fixed-dimensional context vector, constraining model capacity when handling long temporal dependencies.

```
                  ENCODER                                      DECODER
x₁ ──> [RNN] ──> h₁                                       s₁ ──> [RNN] ──> y₁ ("estamos")
        │                                                  ▲
x₂ ──> [RNN] ──> h₂ ──> Static Context Vector (c = h₄) ───┤
        │                                                  ▼
x₃ ──> [RNN] ──> h₃                                       s₂ ──> [RNN] ──> y₂ ("comiendo")
        │                                                  │
x₄ ──> [RNN] ──> h₄ ───────────────────────────────────────┘
```

Recurrent Encoder-Decoder Mechanics

Mathematical Formulation of the Encoder

The encoder sequentially ingests an input token sequence $\mathbf{X} = (x_1, x_2, \dots, x_T)$. At each discrete time-step $t$, the recurrent unit updates its internal representation according to the parametric transition rule:

$$h_t = f_W(x_t, h_{t-1})$$

where:

- $x_t \in \mathbb{R}^{d_{model}}$ represents the vector embedding of the input token at time-step $t$.
- $h_t \in \mathbb{R}^{d_{model}}$ denotes the hidden state at time-step $t$, capturing temporal information up to $t$.
- $f_W$ is a non-linear mapping parameterized by learnable recurrent weight matrix $W$.

Mathematical Formulation of the Decoder

Upon consuming the final source token $x_T$, the encoder synthesizes the full sequence history into a static context vector $c$ (typically assigned directly as the final encoder state, $c = h_T$). The decoder network generates the target sequence $\mathbf{Y} = (y_1, y_2, \dots, y_{T'})$ autoregressively. The dynamic update of the decoder state $s_t$ and output token generation proceed as:

$$s_t = g_U(y_{t-1}, s_{t-1}, c)$$

where:

- $s_t \in \mathbb{R}^{d_{model}}$ is the internal hidden state of the decoder at generation time-step $t$.
- $y_{t-1}$ is the previously generated output token (with $y_0$ initialized to a special `[START]` token).
- $c$ is the fixed static context vector provided across all decoding steps.
- $g_U$ is the decoder transition function parameterized by learnable weights $U$.

Mapping Paradigms and Sequence Topologies

Sequence processing architectures accommodate distinct structural configurations based on input-to-output cardinality, as illustrated below:

```
1. One-to-One          2. One-to-Many         3. Many-to-One
   [Image Class]          [Image Captioning]     [Sentiment Analysis]
     [Output]               [y₁]  [y₂]  [y₃]           [Output]
        ▲                    ▲     ▲     ▲                ▲
     [Hidden]             [h₁]──>[h₂]──>[h₃]             [h₃]
        ▲                    ▲                            ▲
     [Input]              [Input]                     [x₁][x₂][x₃]

2. Many-to-Many (Asynchronous / Delayed)      5. Many-to-Many (Synchronous)
   [Machine Translation]                        [Video Frame Tagging]
        [y₁]  [y₂]  [y₃]                             [y₁]  [y₂]  [y₃]
         ▲     ▲     ▲                                ▲     ▲     ▲
  [h₁]──>[h₂]──>[h₃]──>[s₁]──>[s₂]──>[s₃]           [h₁]──>[h₂]──>[h₃]
   ▲      ▲      ▲                                   ▲     ▲     ▲
  [x₁]   [x₂]   [x₃]                                [x₁]  [x₂]  [x₃]
```

- **One-to-One:** Standard feed-forward architecture without temporal recurrence (e.g., fixed-size image classification).
- **One-to-Many:** Single static input mapped to an unrolling variable-length output sequence (e.g., standard image captioning).
- **Many-to-One:** Variable-length sequential input collapsed into a single output decision (e.g., sentiment analysis or sentence classification).
- **Many-to-Many (Asynchronous / Delayed):** Variable-length input encoded completely before unrolling an independent variable-length output (e.g., sequence-to-sequence translation).
- **Many-to-Many (Synchronous):** Input and output sequences aligned step-by-step at identical temporal intervals (e.g., frame-level video tagging).

Critical Analysis of the Context Bottleneck

The structural failure of early Seq2Seq networks stems from forcing an arbitrary input sequence $x_1, \dots, x_T$—whether $T=10$ or $T=1000$ tokens—into a single fixed-dimensional vector space $c \in \mathbb{R}^{d_{model}}$ (e.g., $d_{model} = 4000$).

As the temporal sequence length $T$ grows, the fixed vector capacity causes severe information loss. The continuous updating of $h_t$ via non-linear functions leads to catastrophic forgetting, where early input tokens ($x_1, x_2$) are obscured by subsequent hidden state updates ($h_{T-1}, h_T$). Consequently, when decoding target tokens later in sequence generation, the decoder loses critical fine-grained source context, impairing translation accuracy on long sequences.

Architectural Transition

The context bottleneck demonstrated that forcing an entire sequence into a single static representation is fundamentally unscalable, creating the need for a dynamic lookup mechanism that exposes all intermediate encoder hidden states to the decoder on demand.

---

2. The Dynamic Attention Mechanism — Bridging Long-Range Dependencies

Dynamic Memory Access in Sequence Models

To eliminate the context bottleneck, Bahdanau et al. (2015) introduced the dynamic attention mechanism. Instead of constraining the decoder to a single fixed vector $c = h_T$, attention treats the complete sequence of intermediate encoder hidden states $(h_1, h_2, \dots, h_T)$ as a dynamic, content-addressable memory archive. At each decoding step $t$, the network computes a customized context vector $c_t$ based on the relevance of every source token to the current target state.

```
ENCODER HIDDEN STATES      ALIGNMENT SCORES           ATTENTION WEIGHTS         DYNAMIC CONTEXT
  h₁ ("we") ───────────┐
  h₂ ("are") ──────────┼──> e₁,ᵢ = f_att(s₀, hᵢ) ──> Softmax ──> a₁,ᵢ ──> c₁ = ∑ a₁,ᵢ hᵢ ──> Decoder State s₁
  h₃ ("eating") ───────┤
  h₄ ("bread") ────────┘
```

Mathematical Formulation of Bahdanau Attention

1. Alignment Scores

At decoder time-step $t$, the architecture evaluates the similarity between the prior decoder hidden state $s_{t-1}$ and each encoder hidden state $h_i$ via an alignment function $f_{att}$:

$$e_{t,i} = f_{att}(s_{t-1}, h_i)$$

where $f_{att}$ is typically implemented as a learnable Multi-Layer Perceptron (MLP) or inner product, producing a scalar score $e_{t,i}$ indicating the relevance of source position $i$ to target generation step $t$.

2. Softmax Normalization

To transform scalar alignment scores into a normalized spatial probability distribution across all source indices $k \in \{1, \dots, T\}$, the network applies the Softmax function:

$$a_{t,i} = \frac{\exp(e_{t,i})}{\sum_{k=1}^T \exp(e_{t,k})}$$

yielding attention weights $a_{t,i}$ that satisfy $0 \le a_{t,i} \le 1$ and $\sum_{i=1}^T a_{t,i} = 1$.

3. Dynamic Context Vector Calculation

The step-specific dynamic context vector $c_t$ is constructed as the weighted linear combination of all intermediate encoder states:

$$c_t = \sum_{i=1}^T a_{t,i} h_i$$

4. Decoder Hidden State Update

The decoder state $s_t$ is updated autoregressively using the dynamically calculated context vector $c_t$:

$$s_t = g_U(y_{t-1}, s_{t-1}, c_t)$$

Interpretability and Alignment Analysis

Visualizing the attention weight matrix $A \in \mathbb{R}^{T' \times T}$ (where entry $(t, i) = a_{t,i}$) yields explicit interpretability regarding source-target token alignments across language pairs.

```
       English Source Tokens ──>
       "agreement"  "on"  "the"  "European"  "economic"  "area"
     ┌─────────────────────────────────────────────────────────┐
 "L'"│   0.05       0.05   0.80     0.05        0.02     0.03  │
"accord"│0.90       0.02   0.03     0.02        0.01     0.02  │
"sur"│   0.02       0.92   0.02     0.01        0.01     0.02  │
"l'" │   0.01       0.02   0.85     0.05        0.02     0.05  │
"espace"│0.02       0.01   0.02     0.05        0.05     0.85  │ <── Non-diagonal
"économique"│0.01   0.01   0.01     0.02        0.90     0.05  │ <── Syntactic
"européen"│0.01     0.01   0.01     0.92        0.03     0.02  │ <── Reordering
     └─────────────────────────────────────────────────────────┘
```

When translating the English clause _"agreement on the European economic area"_ to the French _"l'accord sur l'espace économique européen"_, the attention matrix heatmaps display non-diagonal activations:

- Direct monotonic correspondences generate high attention along diagonal paths (e.g., _"agreement"_ $\rightarrow$ _"accord"_).
- Syntactic variations across language structures (such as English adjective-noun placement vs. French noun-adjective placement) naturally produce non-diagonal activations. The target token _"espace"_ focuses directly on the distant source token _"area"_, while _"européen"_ attends back to _"European"_.
- Crucially, these alignments emerge entirely unsupervised via end-to-end backpropagation, driven solely by the global translation loss without direct token-level alignment supervision.

Cross-Modal Application — Visual Attention in Image Captioning

Xu et al. (2015) extended dynamic attention beyond 1D text sequences to 2D spatial image grids in the _Show, Attend and Tell_ architecture.

```
INPUT IMAGE                CNN FEATURE MAP                  SPATIAL ATTENTION               CAPTION GENERATION
┌───────────┐            ┌──────────────────┐             ┌───────────────────┐
│           │            │ Z ∈ R^(H×W×d_mod)│             │ a_{t,i,j} Over    │           "A dog [a_{t,i,j}]"
│  [Image]  │ ──CNN───>  │ Grid Patches     │ ──Softmax──>│ Grid Regions      │ ──Decoder──>  "standing on"
│           │            │ z_{i,j}          │             │                   │            "hardwood floor"
└───────────┘            └──────────────────┘             └───────────────────┘
```

1. **Spatial Feature Extraction:** A Convolutional Neural Network (CNN) processes an input image, extracting a 3D feature tensor $Z \in \mathbb{R}^{H \times W \times d_{model}}$. This tensor corresponds to a spatial grid of $H \times W$ visual feature vectors $z_{i,j} \in \mathbb{R}^{d_{model}}$, where each vector represents a localized sub-region of the image.
2. **2D Alignment Scoring:** At each text decoding step $t$, alignment scores $e_{t,i,j}$ are computed between the prior decoder state $s_{t-1}$ and each spatial grid cell $z_{i,j}$:

$$e_{t,i,j} = f_{att}(s_{t-1}, z_{i,j})$$

1. **Spatial Softmax Normalization:** Scores are normalized across the entire $H \times W$ spatial domain:

$$a_{t,i,j} = \frac{\exp(e_{t,i,j})}{\sum_{i'=1}^H \sum_{j'=1}^W \exp(e_{t,i',j'})}$$

1. **Visual Context Vector Aggregation:** The dynamic visual context vector $c_t$ is computed as a spatially weighted sum:

$$c_t = \sum_{i=1}^H \sum_{j=1}^W a_{t,i,j} z_{i,j}$$

This mechanism allows the dynamic visual context vector $c_t$ to shift its spatial focus across distinct image regions at each generation step (e.g., placing high spatial weight over canine features when predicting the word _"dog"_, or over background regions when predicting _"hardwood floor"_).

Trade-Off Analysis: Fixed-Context RNN vs. Attention-Augmented RNN

```
+----------------------------------------------------------------------------------------------------+
|                                      COMPUTATIONAL OVERHEAD                                        |
|  Fixed-Context RNN: O(T + T') Sequential Steps                                                     |
|  Attention RNN:     O(T × T') Structural Interactions                                              |
+----------------------------------------------------------------------------------------------------+
|                                         MEMORY FOOTPRINT                                           |
|  Fixed-Context RNN: Keeps Vector 'c' Only ──> O(d_model) Memory                                    |
|  Attention RNN:     Retains All Encoder Hidden States {h₁...h_T} ──> O(T × d_model) Memory         |
+----------------------------------------------------------------------------------------------------+
|                                     BACKPROPAGATION PATHWAY                                        |
|  Fixed-Context RNN: Path Length O(T + T') through Sequential Bottleneck                            |
|  Attention RNN:     Direct O(1) Gradient Paths to Any Historical Encoder State                     |
+----------------------------------------------------------------------------------------------------+
```

- **Computational Complexity:** Fixed-context Seq2Seq models operate with linear dynamic complexity $O(T + T')$. Attention-augmented models scale to $O(T \cdot T')$ operations because every individual output generation step $t \in \{1, \dots, T'\}$ must evaluate inner products or MLP functions against all $T$ source encoder vectors.
- **Memory Overhead:** Fixed-context models require storing only the current state vector $h_t$, maintaining an $O(d_{model})$ operational memory footprint. Attention models must retain the entire sequence of intermediate encoder states $(h_1, \dots, h_T)$ in GPU memory throughout the decoding process, yielding an $O(T \cdot d_{model})$ footprint.
- **Backpropagation Dynamics:** Fixed-context models propagate error gradients through a long sequence of recurrent time-steps, exposing backpropagation to vanishing or exploding gradients. Attention introduces direct, differentiable shortcuts from decoder step $s_t$ to any historical encoder hidden state $h_i$, providing stable gradient flows across temporal distances without requiring explicit positional supervision.

Architectural Transition

While dynamic attention solves the context bottleneck over recurrent states, the underlying architecture remains constrained by sequential recurrence ($h_t = f_W(x_t, h_{t-1})$). Eliminating sequential dependence entirely requires rethinking dynamic attention as an autonomous transformation operating directly on sequence representations: Self-Attention.

---

3. Core Mechanics of Self-Attention (Query, Key, Value Formulation)

Intra-Sequence Transformation Blocks

Self-Attention (Intra-Attention) acts as an autonomous sequence transformation layer that computes interactions directly between all positions in a single sequence, operating independently of temporal recurrences or spatial convolutions. It replaces sequential state updates with direct, pairwise interactions across all constituent tokens.

The Tripartite Projection (Query, Key, Value)

Given an input sequence of embedding vectors $\mathbf{X} \in \mathbb{R}^{N \times d_{model}}$ (where $N$ is sequence length and $d_{model}$ is the core hidden dimension), each input vector $x_i$ is linearly projected into three distinct functional spaces using learnable weight matrices $W_Q, W_K \in \mathbb{R}^{d_{model} \times d_k}$ and $W_V \in \mathbb{R}^{d_{model} \times d_v}$:

```
                    ┌──> Query Vector (Qᵢ = xᵢ W_Q) ───> Current Token Seeking Context
Input Token (xᵢ) ───┼──> Key Vector   (Kᵢ = xᵢ W_K) ───> Structural Identity/Match Target
                    └──> Value Vector (Vᵢ = xᵢ W_V) ───> Content Aggregated into Output
```

1. **Query (**$Q_i = x_i W_Q \in \mathbb{R}^{d_k}$**):** Represents the current token $i$ actively probing the sequence for relevant context.
2. **Key (**$K_i = x_i W_K \in \mathbb{R}^{d_k}$**):** Represents the indexed feature identity of token $i$, serving as the target matched against incoming queries.
3. **Value (**$V_i = x_i W_V \in \mathbb{R}^{d_v}$**):** Represents the actual semantic content aggregated to compute the final output representation.

Mathematical Operations & Matrix Computation

Step-by-Step Scalar Formulation

1. **Alignment Score Computation:** Evaluates the pairwise dot-product similarity between Query vector $Q_i$ and Key vector $K_j$:

$$e_{i,j} = Q_i K_j^T$$

1. **Softmax Normalization:** Normalizes pairwise alignment scores across all sequence positions $j \in \{1, \dots, N\}$ to obtain attention probabilities $\alpha_{i,j}$:

$$\alpha_{i,j} = \text{Softmax}_j(e_{i,j}) = \frac{\exp(e_{i,j})}{\sum_{k=1}^N \exp(e_{i,k})}$$

1. **Value Aggregation:** Computes the transformed output representation $Y_i$ as a weighted sum over all projected Value vectors:

$$Y_i = \sum_{j=1}^N \alpha_{i,j} V_j$$

Parallel Matrix Formulation & Scaling Justification

By stacking input embeddings into matrix $\mathbf{X} \in \mathbb{R}^{N \times d_{model}}$, the projections are processed in parallel across all tokens: $Q = \mathbf{X} W_Q \in \mathbb{R}^{N \times d_k}$, $K = \mathbf{X} W_K \in \mathbb{R}^{N \times d_k}$, and $V = \mathbf{X} W_V \in \mathbb{R}^{N \times d_v}$. The standard matrix equation is expressed as:

$$\text{Attention}(Q, K, V) = \text{Softmax}\left(\frac{Q K^T}{\sqrt{d_k}}\right) V$$

```
   [ Q ]           [ K ]ᵀ                    Softmax                    [ V ]            OUTPUT [ Y ]
┌─────────┐     ┌─────────┐             ┌───────────────┐            ┌─────────┐        ┌─────────┐
│  Q₁...  │  ×  │ K₁...   │  ──Scale─>  │  α₁,₁   α₁,₂  │     ×      │ V₁...   │   =    │ Y₁...   │
│  Q_N... │     │ K_N...  │  (/√d_k)    │  α_N,₁  α_N,N │            │ V_N...  │        │ Y_N...  │
└─────────┘     └─────────┘             └───────────────┘            └─────────┘        └─────────┘
 (N × d_k)       (d_k × N)                   (N × N)                  (N × d_v)          (N × d_v)
```

**Why Scale by** $\frac{1}{\sqrt{d_k}}$**?** Assume $Q_i$ and $K_j$ components are independent random variables with zero mean and unit variance. Their dot product $Q_i K_j^T = \sum_{m=1}^{d_k} Q_{i,m} K_{j,m}$ has mean $0$ and variance $d_k$. For large key dimensions $d_k$ (e.g., $d_k = 64$ or $128$), dot products grow large in magnitude, pushing the Softmax function into regions with extremely small gradients ($\text{Softmax}'(z) \to 0$). Dividing by $\sqrt{d_k}$ normalizes variance back to $1$, preventing the Softmax gradient from vanishing during backpropagation.

Multi-Head Attention (MHA) Mechanism

Single-head attention averages interactions across the sequence, which can obscure multiple distinct relationship types between tokens. Multi-Head Attention (MHA) addresses this by running $h$ parallel attention "heads" (typically $h=8$).

```
                                      ┌──> Head 1 (QW_Q¹, KW_K¹, VW_V¹) ──> Head₁ ┐
                                      │                                           │
Input Tokens (X) ── Projection Matrices ┼──> Head 2 (QW_Q², KW_K², VW_V²) ──> Head₂ ┼──> Concat ──> Wᵒ ──> Multi-Head Output
                                      │                                           │
                                      └──> Head h (QW_Qʰ, KW_Kʰ, VW_Vʰ) ──> Head_h┘
```

Each head $k \in \{1, \dots, h\}$ uses independent projection parameter sets $(W_Q^{(k)}, W_K^{(k)}, W_V^{(k)})$, projecting input embeddings into distinct subspace dimensions $d_k = d_v = d_{model}/h$. This separation allows different heads to simultaneously track varied syntactic and semantic relationships (e.g., tracking grammatical subject-verb dependencies in Head 1 while capturing co-reference resolution in Head 2):

$$\text{Head}_k = \text{Attention}\left(Q W_Q^{(k)}, K W_K^{(k)}, V W_V^{(k)}\right)$$

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{Head}_1, \text{Head}_2, \dots, \text{Head}_h) W^O$$

where $W^O \in \mathbb{R}^{h d_v \times d_{model}}$ aggregates concatenated projections back to the core model dimensional space.

**Parameter Efficiency:** Because each head projects down to $d_k = d_{model}/h$, the total computational complexity across all $h$ heads is mathematically identical to single-head attention operating at full $d_{model}$ dimension ($h \cdot (N^2 \cdot d_k) = N^2 \cdot d_{model}$), granting rich multi-subspace representation capacity at zero extra computational cost.

Architectural Significance

Self-Attention removes the sequential step dependencies of recurrent architectures ($O(N)$ operations). Because every matrix row interacts directly with every matrix column in parallel matrix multiplication, the maximum structural path length between any two sequence positions drops to constant time $O(1)$. This architectural change enables full GPU training parallelization across large datasets.

```
RECURRENT ARCHITECTURE (RNN)              SELF-ATTENTION LAYER
Path Length: O(N) Sequential Steps        Path Length: O(1) Constant Parallel Operations

x₁ ──> [RNN] ──> h₁                       x₁ ────────┐
        │                                 x₂ ────────┼┐
x₂ ──> [RNN] ──> h₂                       x₃ ────────┼┼─> [All-to-All Self-Attention Matrix] ──> [Y₁, Y₂, Y₃, Y₄]
        │                                 x₄ ────────┼┘
x₃ ──> [RNN] ──> h₃                                  │
        │                                            │
x₄ ──> [RNN] ──> h₄ ─────────────────────────────────┘
```

Architectural Transition

Although Self-Attention provides constant path length and parallel execution, the matrix multiplication $\text{Softmax}(Q K^T)$ is permutation-equivariant. Reordering the input token sequence yields an identically reordered output representation sequence. Consequently, explicit positional information must be added to preserve sequence order.

---

4. Positional Encoding — Injecting Sequence Order into Permutation-Invariant Networks

Permutation Equivariance in Self-Attention

Because Self-Attention operates via unordered set operations over Query-Key projections, it lacks an inherent awareness of sequence order. Swapping two input tokens $x_i$ and $x_j$ alters the row order of output matrix $Y$ by the same permutation, leaving the structural token representations identical regardless of position.

To process ordered structures such as text or spatial image grids, explicit positional representations must be added directly to token input embeddings.

Mathematical Criteria for Effective Positional Representations

An effective positional encoding function $PE: \mathbb{N} \to \mathbb{R}^{d_{model}}$ must satisfy four operational criteria:

1. **Unique Output:** Generates a distinct coordinate signature for each discrete time-step or position index $pos$.
2. **Consistent Relative Distance:** Maintains uniform metric distances between relative offset positions ($pos + \Delta$) across sequences of varying length.
3. **Bounded and Generalizable:** Produces deterministic, bounded values that generalize reliably to sequence lengths beyond those encountered during training.
4. **Integration Compatibility:** Integrates efficiently with input embeddings without increasing feature vector dimensionality.

Sinusoidal Positional Encoding Formulation

Vaswani et al. (2017) introduced continuous sinusoidal positional encodings. For token position $pos$ and feature dimension index $2k, 2k+1 \in \{0, \dots, d_{model}-1\}$, encodings are computed as:

$$PE_{(pos, 2k)} = \sin\left(\frac{pos}{10000^{2k/d_{model}}}\right) = \sin(\omega_k \cdot pos)$$

$$PE_{(pos, 2k+1)} = \cos\left(\frac{pos}{10000^{2k/d_{model}}}\right) = \cos(\omega_k \cdot pos)$$

where $\omega_k = \frac{1}{10000^{2k/d_{model}}}$ defines the angular frequency for dimension $k$.

```
DIMENSION INDEX (k) ──>
Low Dimensions (k → 0):    Rapid Oscillation (Short Wavelengths λ ≈ 2π)   ──> [High-Frequency Spatial Counter]
High Dimensions (k → d/2): Slow Oscillation  (Long Wavelengths λ ≈ 10000·2π) ──> [Low-Frequency Global Counter]
```

This geometric design functions similarly to a continuous binary counter:

- Low-index dimensions ($k \to 0$) oscillate at high frequencies, tracking short-range positional shifts between adjacent tokens.
- High-index dimensions ($k \to d_{model}/2$) oscillate at lower frequencies, establishing global coordinate frameworks capable of spanning thousands of positions (wavelengths up to $10,000 \cdot 2\pi$).

Integration Method

Positional signatures are integrated via element-wise addition directly at the model input layer, combining token embeddings $\mathbf{E} \in \mathbb{R}^{N \times d_{model}}$ with positional matrix $\mathbf{PE} \in \mathbb{R}^{N \times d_{model}}$:

$$\mathbf{X}_{embedded} = \mathbf{E} + \mathbf{PE}$$

```
Token Embedding (E_pos) : [ e₁,₁ ,  e₁,₂ ,  e₁,₃ , ... ,  e₁,d_model ]
                            +       +       +              +
Positional Vector (PE_pos): [ sin   , cos   , sin   , ... , cos       ]
                            =       =       =              =
Input Feature Vector    : [ x₁,₁ ,  x₁,₂ ,  x₁,₃ , ... ,  x₁,d_model ]
```

Element-wise addition preserves model dimension $d_{model}$ (unlike vector concatenation, which inflates model size). The lower self-attention layers learn to project and separate semantic content from positional signatures across distinct linear channels.

Relative Positional Encoding Offsets

In applications sensitive to relative distances—such as speech processing, audio modeling, or localized visual processing—relative self-attention formulations incorporate distance offsets ($i - j$) directly into attention logits:

$$e_{i,j}^{\text{relative}} = Q_i K_j^T + Q_i R_{i-j}^T$$

where $R_{i-j}$ is a learnable or deterministic embedding representing the relative displacement index $i - j$.

Architectural Transition

Injecting positional encodings restores sequence order awareness to permutation-invariant self-attention layers. Combined with causal masking, these elements form the fundamental building blocks of the Transformer architecture.

---

5. The Complete Transformer Architecture (Vaswani et al., 2017)

Structural Overview

The Transformer architecture (Vaswani et al., 2017) establishes an end-to-end Sequence-to-Sequence framework constructed entirely from Self-Attention, Positional Encodings, Feed-Forward Networks, and Residual Normalization blocks, eliminating recurrent and convolutional operations.

```
                  TRANSFORMER ENCODER ($N=6$)                             TRANSFORMER DECODER ($N=6$)
            ┌────────────────────────────────────┐                   ┌────────────────────────────────────┐
            │   Position-wise Feed-Forward FFN   │                   │   Position-wise Feed-Forward FFN   │
            │     [Add & Layer Normalization]    │                   │     [Add & Layer Normalization]    │
            ├────────────────────────────────────┤                   ├────────────────────────────────────┤
 Context ───┼────────────────────────────────────┼─ Keys/Values ────>│    Multi-Head Cross-Attention      │
 Output     │     Multi-Head Self-Attention      │                   │     [Add & Layer Normalization]    │
            │     [Add & Layer Normalization]    │                   ├────────────────────────────────────┤
            └────────────────────────────────────┘                   │  Masked Multi-Head Self-Attention  │
                              ▲                                      │     [Add & Layer Normalization]    │
                              │                                      └────────────────────────────────────┘
                     Input Embeddings +                                                ▲
                    Positional Encodings                                               │
                                                                             Target Embeddings +
                                                                            Positional Encodings
```

Transformer Encoder Block Structure ($N=6$ Layers)

The encoder consists of a stack of $N=6$ identical processing layers. Each layer contains two primary sub-layers:

1. **Unmasked Multi-Head Self-Attention:** Computes bidirectional contextual representations across all input sequence tokens simultaneously.
2. **Residual Connection & Layer Normalization:** Applies a residual skip connection followed by Layer Normalization:

$$\text{SubLayer}_{out} = \text{LayerNorm}(x + \text{SubLayer}(x))$$

_Layer Normalization vs. Batch Normalization:_ Layer Normalization computes feature mean and variance across channels for each token position independently of the batch size or dynamic sequence length. Unlike Batch Normalization, which relies on mini-batch statistics that fluctuate on variable-length sequences, LayerNorm operates reliably across individual sequence instances.

1. **Position-wise Feed-Forward Network (FFN):** A two-layer fully connected network applied identically to each sequence token position:

$$\text{FFN}(x) = \max(0, x W_1 + b_1) W_2 + b_2$$

where $W_1 \in \mathbb{R}^{d_{model} \times d_{ff}}$ (typically $d_{ff} = 4 d_{model}$) and $W_2 \in \mathbb{R}^{d_{ff} \times d_{model}}$.

1. **Secondary Residual Connection & Layer Normalization:** Wraps the Feed-Forward Network with an additional skip connection and Layer Normalization step.

Transformer Decoder Block Structure ($N=6$ Layers)

The decoder consists of $N=6$ stacked layers, each containing three functional sub-layers:

1. **Masked Multi-Head Self-Attention (Causal Masking):** Restricts target generation step $t$ from attending to future target tokens $t+1, \dots, T'$.

```
CAUSAL MASK MATRIX (M)                        UNMASKED ATTENTION LOGITS                    MASKED SOFTMAX PROBABILITIES
┌──────────────────────┐                     ┌───────────────────────────┐                 ┌───────────────────────────┐
│  0   -∞   -∞   -∞    │                     │  e₁,₁   e₁,₂  e₁,₃  e₁,₄  │                 │  α₁,₁    0     0     0    │
│  0    0   -∞   -∞    │   +   Q Kᵀ / √d_k ──>│  e₂,₁   e₂,₂  e₂,₃  e₂,₄  │ ──Softmax──>    │  α₂,₁   α₂,₂   0     0    │
│  0    0    0   -∞    │                     │  e₃,₁   e₃,₂  e₃,₃  e₃,₄  │                 │  α₃,₁   α₃,₂  α₃,₃   0    │
│  0    0    0    0    │                     │  e₄,₁   e₄,₂  e₄,₃  e₄,₄  │                 │  α₄,₁   α₄,₂  α₄,₃  α₄,₄  │
└──────────────────────┘                     └───────────────────────────┘                 └───────────────────────────┘
```

Setting ungenerated future alignments to $-\infty$ forces $\text{Softmax}(-\infty) = 0$, enforcing causal autoregressive sequence generation during training and inference.

1. **Multi-Head Cross-Attention:** Connects the encoder and decoder stacks. Queries ($Q$) are projected from the decoder's prior sub-layer, while Keys ($K$) and Values ($V$) are derived directly from the output of the Encoder stack. This allows every position in the decoder to attend over all source tokens in the input context.
2. **Position-wise Feed-Forward Network:** Applies the position-wise linear transformations wrapped in residual skip connections and layer normalizations.

Comparative Complexity Analysis

|Layer Paradigm|Computational Complexity per Layer|Sequential Operations (Parallelization Limit)|Maximum Structural Path Length|
|---|---|---|---|
|**Self-Attention**|$\mathcal{O}(N^2 \cdot d_{model})$|$\mathcal{O}(1)$|$\mathcal{O}(1)$|
|**Recurrent Layer (RNN)**|$\mathcal{O}(N \cdot d_{model}^2)$|$\mathcal{O}(N)$|$\mathcal{O}(N)$|
|**Convolutional Layer (1D CNN)**|$\mathcal{O}(k \cdot N \cdot d_{model}^2)$|$\mathcal{O}(1)$|$\mathcal{O}(\log_k(N))$|

_where_ $N$ _represents sequence length,_ $d_{model}$ _is feature embedding dimensionality, and_ $k$ _is kernel spatial width._

Architectural Transition

By eliminating sequential recurrences and structural inductive biases, the Transformer achieves high GPU hardware parallelization. This capability enabled scaling models to massive datasets and extended their application beyond NLP to visual domains.

---

6. Extending Transformers to Computer Vision — Vision Transformers (ViT) and Swin Transformers

The Shift Away from Convolutional Inductive Biases

Convolutional Neural Networks rely on two explicit structural inductive biases:

- **Locality:** Assuming nearest-neighbor pixels are semantically correlated.
- **Translation Invariance:** Applying identical convolutional filters uniformly across all spatial coordinates.

While effective for localized image visual signals, these assumptions constrain a network's ability to model non-local, dynamic spatial interactions (such as multi-object occlusions or long-range visual context). Vision Transformers eliminate these fixed convolutional biases, learning dynamic spatial relationships directly from raw image patch sequences.

```
CONVOLUTIONAL LAYER: Fixed Local Receptive Fields      VISION TRANSFORMER: Global All-to-All Patch Relationships
┌───┬───┬───┐                                         ┌──────┐ ───┐
│   │ ★ │   │ Local 3×3 Kernel Computation            │ Patch│    │
├───┼───┼───┤ (Restricted Receptive Field)            ├──────┤    ├──> Full-Image Dynamic Interactions
│ ★ │ X │ ★ │                                         │ Patch│    │    (Constant O(1) Receptive Field)
├───┼───┼───┤                                         ├──────┤    │
│   │ ★ │   │                                         │ Patch│ ───┘
└───┴───┴───┘                                         └──────┘
```

The Vision Transformer (ViT) Pipeline (Dosovitskiy et al., 2020)

```
Input Image (H×W×C) ──> 2D Patch Extraction ──> Flattened Sequence (N) ──> Linear Projection + [CLS] Token ──> Add 2D Positional Enc. ──> Transformer Encoder Stack
```

1. Image Patchification

A 2D image $X \in \mathbb{R}^{H \times W \times C}$ is partitioned into a sequence of non-overlapping $P \times P$ spatial patches. Each patch is flattened into a 1D vector $x_p \in \mathbb{R}^{P^2 \cdot C}$. The total sequence length $N$ is defined as:

$$N = \frac{H \cdot W}{P^2}$$

2. Linear Projection and Token Prep

The flattened patch vectors are mapped to the core model embedding dimension $d_{model}$ using a learnable projection matrix $E \in \mathbb{R}^{(P^2 \cdot C) \times d_{model}}$. A learnable classification token `[CLS]` ($z_0^0 = x_{class}$) is prepended to the patch sequence, serving as an aggregated global representation vector for classification tasks:

$$\mathbf{Z}_0 = [x_{class}; x_p^1 E; x_p^2 E; \dots; x_p^N E] + \mathbf{E}_{pos}$$

where $\mathbf{E}_{pos} \in \mathbb{R}^{(N+1) \times d_{model}}$ provides 2D spatial positional embeddings.

3. Encoder Processing

The resulting sequence of visual tokens $\mathbf{Z}_0$ is processed through standard Transformer Encoder blocks, modeling global visual interactions across the entire image grid.

The Swin Transformer — Hierarchical Windowed Attention (Liu et al., 2021)

```
VISION TRANSFORMER (ViT)                        SWIN TRANSFORMER (Hierarchical Windowed Attention)
Global Quadratic Complexity: O(N²)              Linear Local Window Attention: O(N) + Shifted Window Cross-Communication

┌───┬───┬───┬───┐ All Patches                   ┌───┬───┐ ┌───┬───┐ Local Windows
│   │   │   │   │ Attend to All                 │   │   │ │   │   │ (Computes Attention Within
├───┼───┼───┼───┤ Other Patches                 ├───┼───┤ ├───┼───┤ Local P×P Boundaries)
│   │   │   │   │ Globally                      │   │   │ │   │   │
└───┴───┴───┴───┘                               └───┴───┘ └───┴───┘ Layer Shift Interconnects Windows
```

1. Quadratic Bottleneck of Standard ViT

Standard ViT models evaluate global self-attention across all $N$ image patches, resulting in quadratic computational complexity $\mathcal{O}(N^2) = \mathcal{O}\left(\left(\frac{HW}{P^2}\right)^2\right)$. While manageable for low-resolution classification images (e.g., $224 \times 224$), this quadratic cost becomes computationally unfeasible for dense, high-resolution visual tasks like object detection or semantic segmentation on multi-megapixel images.

2. Shifted Windowing (Swin)

The Swin Transformer restricts self-attention computation to local $M \times M$ patch windows (typically $M=7$), yielding linear computational complexity relative to image size:

$$\mathcal{O}(\text{Swin}) = 4 N M^2 d_{model} + 2 N d_{model}^2 = \mathcal{O}(N)$$

To establish spatial communication between adjacent local windows, consecutive layers alternate between standard local windowing (W-MSA) and shifted windowing configurations (SW-MSA), shifting window boundaries by $\left(\left\lfloor \frac{M}{2} \right\rfloor, \left\lfloor \frac{M}{2} \right\rfloor\right)$ patches.

3. Hierarchical Patch Merging

As visual representations propagate through deeper layers, Swin progressively concatenates adjacent $2 \times 2$ patch groups and applies linear projections, reducing spatial sequence resolution while doubling feature dimension depth ($C \to 2C \to 4C$). This constructs a multi-scale hierarchical feature pyramid essential for dense visual prediction tasks.

Architectural Transition

By adapting self-attention to high-resolution visual inputs, Vision Transformers establish a unified processing paradigm across text, images, and multimodal domains.

---

7. Core Takeaways and Architectural Summary

Synthesized Takeaway Checklist

```
                      EVOLUTION OF SEQUENCE AND VISION ARCHITECTURES
                      
    RECURRENT BOTTLENECK           DYNAMIC ATTENTION               SELF-ATTENTION (TRANSFORMER)             VISION TRANSFORMERS (ViT/SWIN)
   ┌────────────────────┐       ┌────────────────────┐            ┌────────────────────────────┐           ┌──────────────────────────────┐
   │ Fixed Context 'c'  │ ───>  │ Dynamic Alignment  │ ────────>  │ Parallel Matrix Operations │ ────────> │ Patchified Image Tokens      │
   │ O(N) Path Length   │       │ Variable Memory    │            │ O(1) Constant Path Length  │           │ Linear Local/Shifted Windows │
   └────────────────────┘       └────────────────────┘            └────────────────────────────┘           └──────────────────────────────┘
```

1. **Recurrent Context Bottleneck:** Early Seq2Seq networks compressed variable-length source sequences into a single fixed vector $c = h_T$, leading to catastrophic forgetting over long temporal horizons ($T \ge 1000$).
2. **Dynamic Memory Lookup:** Bahdanau Attention introduced a dynamic lookup mechanism, constructing a custom context vector $c_t = \sum a_{t,i} h_i$ at each decoding step by evaluating alignment scores over intermediate encoder states.
3. **Self-Attention Abstraction:** Self-attention replaces temporal updates with dynamic matrix projections ($Q = \mathbf{X} W_Q$, $K = \mathbf{X} W_K$, $V = \mathbf{X} W_V$), computing direct token interactions in parallel via $\text{Softmax}\left(\frac{Q K^T}{\sqrt{d_k}}\right) V$.
4. **Positional Encoding:** Because self-attention is permutation-equivariant, sinusoidal functions ($PE_{(pos, 2k)} = \sin(pos / 10000^{2k/d_{model}})$) are added directly to input token embeddings to explicitly encode sequence order.
5. **Causal Auto-Regressive Decoders:** Decoder self-attention uses causal masking, setting future alignment logits to $-\infty$ to ensure autoregressive prediction without look-ahead leakage.
6. **Unified Multimodal Paradigm:** Vision Transformers (ViT) flatten 2D spatial image patches into sequence tokens, removing convolutional inductive biases to enable scalable end-to-end learning across visual and text domains.

By replacing fixed inductive biases with unconstrained dynamic attention layers, modern Transformer architectures leverage parallel GPU computation to learn complex context patterns, establishing the architectural foundation for modern foundation models across language and computer vision.