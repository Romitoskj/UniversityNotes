# 
### **1. Transformers Recap: Encoders vs. Decoders**

A fundamental distinction in Transformer architectures lies in causality. **Encoders** allow information to flow bidirectionally (non-causal), making them ideal for tasks where the full context is needed. **Decoders**, such as GPT models, are strictly causal; future tokens cannot transmit information back to previous ones. A decoder generates text by processing past information to predict the next token.

### **2. The Residual Stream**

Inside a Transformer, information does not simply flow sequentially through one block and into the next. Instead, there is a central pathway called the **Residual Stream**.

- **Skip Connections:** The input embeddings are copied. One path goes into the Attention or MLP blocks, and the output is then added back into the main stream via a skip (or residual) connection.
- **Reading and Writing:** Every Attention and MLP layer can be thought of as an operation that either _reads_ from the residual stream or _writes_ (adds) new information back into it.
- **Information Bottleneck:** The residual stream contains absolutely everything the model has computed up to that point: the full prompt, logical reasoning steps, intermediate answers, and abstract thoughts. Because Transformers are injective, the information in the stream is so rich that it is theoretically possible to fully reconstruct the original prompt strictly from the intermediate embeddings.

### **3. The Logit Lens and Mechanistic Interpretability**

To reverse-engineer the "black box" of the residual stream, researchers use a technique called the **Logit Lens**.

- **The Technique:** Normally, the model's final embedding is passed through an unembedding matrix to predict the final word token. The Logit Lens applies this exact same unembedding matrix to the _intermediate_ layers of the residual stream to see what the model is "thinking" at any given stage.
- **Tracing Thoughts:** For a prompt like _"Kimchi is eaten in the country known for popstar"_, applying the Logit Lens reveals the model's internal thought process. In early layers, it processes "Kimchi"; in middle layers, it starts surfacing related concepts like "Korea", "rice", or "pizza"; and finally, in the later layers, the model's confidence grows as it locks onto the correct answer ("Psy").
- **The Multilingual Problem:** Applying the Logit Lens to a multilingual model (like Qwen) often outputs complete garbage in the middle layers. This happens because a multilingual model cannot afford to align abstract semantic concepts with specific words for every language. Instead, it behaves like an autoencoder: it heavily compresses the input tokens into a deep, language-agnostic latent space for reasoning, and only decodes this logic back into specific text tokens at the very last layers.

### **4. The Linear Representation Hypothesis & Superposition**

A core theory in interpretability is that models encode key concepts (features) simply as **linear directions (vectors)** within the embedding space.

- **The Dimensionality Problem:** A model like GPT2-XL has a residual stream of 1600 dimensions, yet the data it learns contains vastly more than 1600 features. If features always occurred simultaneously, the model could only map them perfectly orthogonally (one feature per dimension) and would have to discard the rest.
- **Sparsity and Superposition:** In reality, features are **sparse** (e.g., a text rarely discusses "cars" and "long rabbit ears" at the exact same time). Because of this sparsity, the model can pack features "quasi-orthogonally." By accepting a tiny amount of interference ($\epsilon$) between features, the number of features a model can successfully pack grows _exponentially_ as the dimensionality of the space increases.
- This phenomenon—where a single neuron or dimension handles multiple different features depending on the context—is called **Superposition**.

### **5. Neurons, Key-Value Memory, and Activation Oracles**

- **MLP as Key-Value Memory:** The MLP layers can be viewed as key-value memories. The first layer reads the residual stream, using its neurons to detect specific features (the Key). The second layer then decides how to linearly combine and write that new information back into the stream for the next layer to use (the Value).
- **Monosemantic Neurons:** While many neurons are in superposition, some become highly specialized **monosemantic neurons** that only care about one specific concept. Famous examples include a neuron dedicated exclusively to detecting mentions of the "Golden Gate Bridge," or neurons that strictly fire when processing palindromes.
- **Interpreting the Black Box:** To understand these neurons, researchers are currently building **Activation Oracles**—using one LLM to inspect the raw weights of another model and output a human-readable explanation of what that specific neuron is designed to detect. Additionally, **Sparse Autoencoders** are heavily utilized to mathematically break superposition and untangle the features the model has learned.