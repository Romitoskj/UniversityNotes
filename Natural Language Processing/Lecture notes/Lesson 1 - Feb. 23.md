
### **Lecture 1: Introduction to Natural Language Processing (NLP)**

**Instructor:** Prof. Stefano Faralli (Part 1) & Prof. Roberto Navigli (Part 2)

---

### **1. Course Logistics & Organization**

**Structure:**

- **Part 1 (Prof. Faralli):** Focuses on "Foundations of NLP." This covers historical but essential methods needed to understand modern pipelines, including text preprocessing, regex, automata, and probabilistic models.
- **Part 2 (Prof. Navigli):** Focuses on "Deep Learning and Multimodal NLP." This covers neural networks, transformers, sequence modeling, and generative AI.

**Grading System:** The final grade is a combination of a written test and optional project work.

- **Written Test:** Traditional open-ended questions (no coding required in the test).
    - _Maximum Grade:_ **28/30** if you only do the written test.
- **Optional Homework/Project:** Required to achieve a grade higher than 28 (up to 30L).
    - **Homework:** Individual work. Worth max **+2 points**.
    - **Homework Project:** Can be done in teams of two. Worth max **+4 points**.
    - _Project Format:_ Must follow a scientific paper structure (Introduction, Related Work, Methodology, Experiments). Length: ~4 pages (excluding references).
    - _Topic:_ You must submit a proposal via a form. Topics often involve reproducing a paper, comparing methodologies, or testing on new datasets.

**Resources:**

- **Classroom Code:** Provided in the lecture (video recordings will be shared here).
- **Textbook:** "Jurafsky & Martin" (Stanford) is the reference for the foundational algorithms.
- **Google Form:** Available for project proposals and seeing exam examples.

---

### **2. Definition & Scope of NLP**

**What is NLP?**

- It is an interdisciplinary field concerned with the interactions between computers and natural human languages (speech or text).
- It sits at the intersection of **Linguistics** (understanding language structure) and **Computer Science** (translating knowledge into programs via AI).

**Common Applications:**

- Personal voice assistants (Siri, Alexa), Autocomplete, Spell checking, Machine Translation (Google Translate), Chatbots, and Sentiment Analysis.
- While generative AI (like GPT) is the current trend, it relies heavily on the foundational NLP tasks taught in this course.

---

### **3. The Linguistics Perspective (The 6 Levels)**

To process language, NLP analyzes it through a taxonomy of six linguistic levels:

1. **Phonetics:** Study of individual sounds of human language.
2. **Phonology:** Study of sound systems and how sounds aggregate in languages.
3. **Morphology:** Study of the formation and internal structure of **words**.
4. **Syntax:** Study of the formation and structure of **sentences** (grammar).
5. **Semantics:** Study of **meaning** (of words and sentences).
6. **Pragmatics:** Study of how sentences are used for specific communicative goals (contextual usage).

---

### **4. The AI Perspective (Taxonomies)**

**The AI Watch Taxonomy (2020):** The lecture references the European Union's "AI Watch" taxonomy to categorize AI subfields:

- **Communication:** This is the branch where NLP resides.
- **Related Fields:** Reasoning, Planning, Learning, Perception.
- _Note:_ Modern Deep Learning has blurred these lines; for example, an NLP model might now also perform reasoning or interact with Computer Vision (multimodal).

**Interactions:** NLP interacts with other domains, such as **AI Ethics** (handling fake news, deepfakes, bias) and **AI Services** (APIs from Google/AWS).

---

### **5. Historical Milestones in NLP**

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