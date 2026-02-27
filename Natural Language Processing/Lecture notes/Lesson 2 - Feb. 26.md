**Part 1: NLP Research Progress & The #BenderRule**

**The #BenderRule (Ethics and Inclusion in NLP)**

- Proposed by Emily M. Bender in 2019, this rule emphasizes that **"English is neither synonymous with nor representative of natural language"**.
- It highlights a historical bias in the NLP community, which has traditionally focused heavily on "high-resource languages" (like English) while neglecting "low-resource languages" (like many African languages).
- The rule pushes researchers to explicitly name the languages they are studying rather than assuming English as the default.

**Categorization of NLP Tasks (Based on English)**

- **1. Mostly Solved:**
    - **Text Classification:** e.g., spam detection in emails.
    - **Part of Speech (POS) Tagging:** Determining if a word is a noun, verb, adjective, etc..
    - **Named Entity Recognition (NER):** Identifying entities like people, locations, and organizations in a text.
- **2. Making Solid Progress:**
    - **Sentiment Analysis:** Determining the polarity (positive/negative) or specific emotions of a sentence.
    - **Co-reference Resolution:** Figuring out which words refer to the same entity (e.g., knowing that "he" refers to "Manning" in a specific sentence).
    - **Word Sense Disambiguation (WSD):** Determining the correct meaning of a polysemous word based on context (e.g., a river "bank" vs. a financial "bank").
    - **Machine Translation:** Automated translation between languages, which has seen massive improvements recently.
- **3. Still Challenging:**
    - **Open-domain Dialogue Agents/Chatbots:** Building systems that can converse coherently about _any_ topic.
    - **Abstractive Summarization:** Generating a concise summary using new, abstracted concepts rather than simply extracting existing sentences from the source.
    - **NLP for Low-Resource Languages:** Applying NLP techniques to languages with limited training data.
- **4. New Frontiers:**
    - Multimodal generation, such as generating images, audio, video, or interactive 3D environments strictly from text prompts.
    - Software requirements and code generation from text.

--------------------------------------------------------------------------------

**Part 2: Regular Expressions & Pattern Matching**

**What are Regular Expressions (Regex)?**

- A regular expression is a **compact textual representation of a set of strings** that represent a language.
- They are heavily used in **pattern matching** (e.g., using `grep`, finding files in a directory, or searching through large text corpora).
- Regex is a powerful tool for **"shallow" processing**. Before applying complex deep learning models, regex can be used to quickly find word frequencies, locate co-occurrences, or clean up text.

**Basic Syntax & Operators**

- **Brackets** **[ ]**: Used to match a single character from a specified set.
    - `[wW]oodchuck` matches "Woodchuck" or "woodchuck".
    - Ranges can be specified: `[A-Z]` for uppercase letters, `[a-z]` for lowercase, for digits.
- **Caret** **^**: When placed _inside_ brackets `[^...]`, it acts as a negation (e.g., `[^A-Z]` matches anything that is _not_ an uppercase letter). When outside, it signifies the start of a string.
- **Period** **.**: Matches any single character (e.g., `beg.n` matches "begin", "begun", etc.).
- **Quantifiers**: Specify how many times the previous character/pattern should occur:
    - `*` : Zero or more occurrences.
    - `+` : One or more occurrences.
    - `?` : Exactly zero or one occurrence (makes the preceding character optional, e.g., `colou?r` matches "color" and "colour").
    - `{n}` or `{n,m}` : Exactly _n_ occurrences, or between _n_ and _m_ occurrences.
- **Aliases / Shortcuts**:
    - `\d` : Any digit (equivalent to ).
    - `\w` : Any alphanumeric character or underscore.
    - `\s` : Any whitespace (space, tab, newline).
    - _(Note: Capitalized versions like_ _\D__,_ _\W__,_ _\S_ _represent the exact opposites/negations)_.

**Advanced Uses of Regex**

- **Substitutions:** Regex can be used to find and replace text. For example, the Perl substitution operator `s/regexp/pattern/` replaces a matched regex string with a new pattern.
- **Registers (Capture Groups):** Parentheses `( )` can be used to group parts of a pattern. You can then use a number operator (like `\1`) to refer back to whatever was matched in that specific group.
    - _Example:_ `/the (.*)er they were, the \1er they will be/` will match sentences where the same word fills both blanks.

**Historical Application: ELIZA (1966)**

- Created by Joseph Weizenbaum, **ELIZA was the very first chatbot**.
- It simulated a Rogerian psychologist communicating with users.
- Despite appearing to understand human input, **ELIZA relied entirely on simple pattern matching (regular expressions)** to rephrase user statements into questions, possessing zero actual knowledge of the world or the conversation.