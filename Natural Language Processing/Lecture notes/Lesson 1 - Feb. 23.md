
## **Lecture 1: Introduction to Natural Language Processing (NLP)**

### **1. Definition & Scope of NLP**

**What is NLP?**

- It is an interdisciplinary field concerned with the interactions between computers and natural human languages (speech or text).
- It sits at the intersection of **Linguistics** (understanding language structure) and **Computer Science** (translating knowledge into programs via AI).

**Common Applications:**

- Personal voice assistants (Siri, Alexa), Autocomplete, Spell checking, Machine Translation (Google Translate), Chatbots, and Sentiment Analysis.
- While generative AI (like GPT) is the current trend, it relies heavily on the foundational NLP tasks taught in the first part of this course.

---

### **2. The Linguistics Perspective (The 6 Levels)**

To process language, NLP analyzes it through a taxonomy of six linguistic levels:

1. **Phonetics:** Study of individual sounds of human language.
2. **Phonology:** Study of sound systems and how sounds aggregate in languages.
3. **Morphology:** Study of the formation and internal structure of **words**.
4. **Syntax:** Study of the formation and structure of **sentences** (grammar).
5. **Semantics:** Study of **meaning** (of words and sentences).
6. **Pragmatics:** Study of how sentences are used for specific communicative goals (contextual usage).

---

### **3. The AI Perspective (Taxonomies)**

**The AI Watch Taxonomy (2020):** The lecture references the European Union's "AI Watch" taxonomy to categorize AI subfields:

- **Communication:** This is the branch where NLP resides.
- **Related Fields:** Reasoning, Planning, Learning, Perception.
- _Note:_ Modern Deep Learning has blurred these lines; for example, an NLP model might now also perform reasoning or interact with Computer Vision (multimodal).

**Interactions:** NLP interacts with other domains, such as **AI Ethics** (handling fake news, deepfakes, bias) and **AI Services** (APIs from Google/AWS).

---

### **4. Historical Milestones in NLP**

The course views NLP history through three main paradigms:

**A. Rule-Based Systems (Symbolic)**

- **Definition:** Systems relying on hand-crafted, domain-specific rules (e.g., Regular Expressions).
- **Pros:** Interpretable.
- **Cons:** High complexity to maintain, they do not **generalize** well to new domains or languages, and are not robust against variations (e.g., a spam filter that fails when spammers change one word).

**B. Statistical & Classical Machine Learning**

- **Definition:** Algorithms that learn parameters by observing examples (input/output pairs).
- **Key Concepts:**
    - **Probabilistic Models:** The foundation of modern AI.
    - **Feature Engineering:** Techniques like **Bag of Words** and **Part-of-Speech tagging** used to represent text.
    - **One-Hot Encoding:** An older method of representing text where words are vectors of 0s and 1s. This had huge memory issues and lacked semantic understanding.

**C. Deep Learning (The Current Era)**

- **Definition:** Uses neural networks to learn both the task and the best representation of the data.
- **Key Innovation:** **Word Embeddings** (e.g., Word2Vec, 2013). This allowed words to be represented as dense vectors in a latent space, capturing semantic meaning and solving the memory problems of One-Hot encoding.
- **Advantage:** Superior generalization capabilities compared to previous methods.