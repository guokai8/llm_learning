# Chapter 3: Natural Language Processing Fundamentals
*From Words to Vectors: The Journey Begins*

## What We'll Learn Today 🎯
- How computers "chop up" text (tokenization) 
- Turning words into numbers that computers understand (embeddings)
- Teaching computers to predict language (language modeling)
- How to measure if our model is actually good (evaluation)

**Big Idea:** Computers don't understand words - they only understand numbers. So we need to convert language into math!

---

## 3.1 Tokenization: Chopping Up Text 🔪

### The Fundamental Challenge

#### The Problem
```
Humans read: "I love machine learning!"
Computers see: A string of meaningless characters

We need to break text into meaningful pieces (tokens)
```

#### What is a Token?
```
Think of tokens as "LEGO blocks" of language:
- Each block represents a meaningful unit
- We can build sentences by combining blocks
- The key question: How big should each block be?
```

### Method 1: Word-Level Tokenization 📝

#### How It Works
```
Input: "I love cats and dogs"
Output: ["I", "love", "cats", "and", "dogs"]

Simple rule: Split on spaces and punctuation
```

#### Analogy: Cutting a Sentence Like a Pizza 🍕
```
Imagine each word is a pizza slice:
- Easy to understand what each slice represents
- Each slice has clear meaning
- You can eat (process) each slice independently
```

#### Advantages ✅
- **Intuitive:** Each token has clear meaning
- **Interpretable:** Easy for humans to understand  
- **Semantic preservation:** Meaning of words is kept intact

#### Problems ❌

**Problem 1: Vocabulary Explosion**
```
English has ~170,000 words in current use
Add proper nouns, technical terms, slang...
Result: HUGE vocabulary = HUGE memory requirements
```

**Problem 2: Out-of-Vocabulary (OOV) Words**
```
Training data: "I like cats"
New text: "I like GPU" 
Model: "What the heck is a GPU??" 🤔
```

**Problem 3: Morphological Variants**
```
The model treats these as completely different:
- "run", "running", "runs", "ran"
- But they're clearly related!
```

### Method 2: Character-Level Tokenization 🔤

#### How It Works
```
Input: "I love cats"
Output: ["I", " ", "l", "o", "v", "e", " ", "c", "a", "t", "s"]
```

#### Analogy: Reading Letter by Letter 📚
```
Like reading a book one letter at a time:
- You never encounter an "unknown" letter
- But it takes forever to get to the meaning
- Hard to understand what's going on
```

#### Advantages ✅
- **No OOV problem:** Fixed, small vocabulary (26 letters + punctuation)
- **Language agnostic:** Works for any language
- **Handles typos:** Can process any character combination

#### Problems ❌
- **Very long sequences:** "Hello" becomes 5 tokens instead of 1
- **Lost semantics:** Hard to capture word-level meaning
- **Computational cost:** Much longer sequences to process

### Method 3: Subword Tokenization (The Sweet Spot!) 🎯

#### The Big Idea
```
"What if we could have the best of both worlds?"
- Keep common words as single tokens (like word-level)
- Break rare words into smaller pieces (like character-level)
```

#### Analogy: Smart Text Compression 📦
```
Think of a smart compression algorithm:
- Frequent patterns get short codes
- Rare patterns get longer codes
- But everything can still be represented!
```

### Byte Pair Encoding (BPE): The Most Popular Method

#### The Algorithm (Step by Step)

**Step 1: Start with characters**
```
Text: "low lower lowest"
Initial vocab: {"l", "o", "w", "e", "r", "s", "t", "</w>"}
Note: </w> marks word endings
```

**Step 2: Count all adjacent pairs**
```
Text: "l o w </w> l o w e r </w> l o w e s t </w>"

Pair counts:
- "l o": 3 times
- "o w": 3 times  
- "w e": 2 times
- "e r": 1 time
- ... and so on
```

**Step 3: Merge the most frequent pair**
```
Most frequent: "l o" (appears 3 times)
Merge: "l o" → "lo"

New text: "lo w </w> lo w e r </w> lo w e s t </w>"
Add "lo" to vocabulary
```

**Step 4: Repeat until desired vocabulary size**
```
Next iteration: "o w" is most frequent
Merge: "o w" → "ow"
Result: "low </w> low e r </w> low e s t </w>"

Continue until we have enough tokens...
```

**Final Result:**
```
Vocabulary: {"l", "o", "w", "e", "r", "s", "t", "</w>", "lo", "ow", "low", "er", "est"}
Text: ["low</w>", "low", "er</w>", "low", "est</w>"]
```

#### Why BPE is Brilliant 🧠
```
✅ Common words stay together: "the", "and", "because"
✅ Rare words get broken down: "antidisestablishmentarianism" 
✅ No OOV problem: Any new word can be broken into known subwords
✅ Balances vocabulary size vs sequence length
```

### Other Subword Methods

#### WordPiece (Used by BERT)
```
Similar to BPE but:
- Uses likelihood-based scoring instead of frequency
- Adds "##" prefix for continuation subwords
- Example: "playing" → ["play", "##ing"]
```

#### SentencePiece (Used by T5, many modern models)
```
Key innovations:
- Treats spaces as regular characters
- No pre-tokenization required
- Reversible: can perfectly reconstruct original text
- Language-independent
```

### Practical Example: How GPT Tokenizes

```
Input: "Hello, how are you today?"

GPT tokenization:
["Hello", ",", " how", " are", " you", " today", "?"]

Notice:
- Spaces are included with words (" how")  
- Punctuation often separate tokens
- Common words stay together
```

### Choosing Vocabulary Size: The Trade-off

```
Small vocabulary (1K tokens):
✅ Less memory 
❌ Longer sequences
❌ Less semantic preservation

Large vocabulary (100K tokens):  
✅ Shorter sequences
✅ Better semantic preservation
❌ More memory
❌ Sparse training signal

Sweet spot: 30K-50K tokens for most models
```

---

## 3.2 Word Embeddings: From Words to Vectors 🔢

### The Core Problem

#### How Do We Represent Words to Computers?
```
Humans: "cat" and "dog" are similar (both animals)
Computer: "cat" = ? and "dog" = ?

We need a way to represent words as numbers that capture meaning!
```

### Attempt 1: One-Hot Encoding (The Naive Approach)

#### How It Works
```
Vocabulary: ["cat", "dog", "bird", "car"]

"cat"  = [1, 0, 0, 0]
"dog"  = [0, 1, 0, 0]  
"bird" = [0, 0, 1, 0]
"car"  = [0, 0, 0, 1]
```

#### Problems with One-Hot
```
❌ No semantic similarity:
   - Distance between "cat" and "dog" = distance between "cat" and "car"
   - Computer can't tell that cat and dog are both animals!

❌ Huge vectors:
   - 50K vocabulary = 50K dimensional vectors
   - Mostly zeros (sparse and inefficient)
```

### The Distributional Hypothesis 💡

#### The Key Insight
```
"You shall know a word by the company it keeps" - J.R. Firth

Words that appear in similar contexts have similar meanings.
```

#### Examples
```
Context: "The ___ is sleeping on the couch"
Likely words: cat, dog, baby, person
→ These words are similar!

Context: "The ___ flew over the trees"  
Likely words: bird, plane, helicopter
→ These words are similar!
```

### Word2Vec: The Breakthrough

#### Two Flavors: CBOW vs Skip-gram

**CBOW (Continuous Bag of Words):**
```
Given context words → Predict center word

Input: ["the", "cat", "on", "the"]
Target: "sat"

Think: "Given these surrounding words, what's the missing word?"
```

**Skip-gram:**
```
Given center word → Predict context words

Input: "sat"  
Target: ["the", "cat", "on", "the"]

Think: "Given this word, what words should appear around it?"
```

#### Skip-gram in Detail (More Popular)

**Step 1: Set up the problem**
```
Sentence: "The cat sat on the mat"
Window size: 2

For word "sat":
Context: ["The", "cat", "on", "the"]
```

**Step 2: The neural network**
```
Input: One-hot vector for "sat"
↓
Hidden layer: Dense vector (embedding!)
↓
Output: Probability distribution over all words

Goal: High probability for context words, low for others
```

**Step 3: Training**
```
For each (center, context) pair:
1. Forward pass: Get predicted probabilities
2. Calculate loss: How wrong were we?
3. Backpropagation: Update embeddings
4. Repeat millions of times!
```

#### The Magic Result 🪄
```
After training, similar words have similar embeddings:

"king" ≈ [0.2, 0.8, -0.1, 0.5, ...]
"queen" ≈ [0.3, 0.7, -0.2, 0.4, ...]
"car" ≈ [-0.1, 0.1, 0.9, -0.3, ...]

Distance between king and queen < Distance between king and car!
```

#### Famous Word2Vec Results
```
Vector arithmetic that actually works:
"king" - "man" + "woman" ≈ "queen"
"Paris" - "France" + "Italy" ≈ "Rome"
"walking" - "walk" + "swim" ≈ "swimming"

This blew everyone's minds! 🤯
```

### GloVe: Global Vectors

#### The Motivation
```
Word2Vec problem: Only uses local context windows
GloVe idea: Use global corpus statistics!
```

#### How GloVe Works
```
1. Build global co-occurrence matrix
   - Count how often word i appears with word j
   
2. Optimize objective function:
   - Want: dot product of embeddings ≈ log(co-occurrence count)
   - With weights to handle rare/frequent words
   
3. Result: Embeddings that capture global statistics
```

#### GloVe vs Word2Vec
```
Word2Vec: Local context, prediction-based
GloVe: Global statistics, count-based
Performance: Similar, but GloVe often faster to train
```

### FastText: Handling Rare Words

#### The Innovation
```
Problem: Word2Vec can't handle words not seen in training
Solution: Represent words as sum of character n-grams!
```

#### Example
```
Word: "where"
Character 3-grams: ["<wh", "whe", "her", "ere", "re>"]
Plus the full word: "where"

Final embedding = sum of all n-gram embeddings
```

#### Benefits
```
✅ Can handle OOV words (break into n-grams)
✅ Captures morphological relationships
✅ Works great for morphologically rich languages
✅ Same speed as Word2Vec
```

### Evaluating Word Embeddings

#### Intrinsic Evaluation

**Word Similarity Tasks:**
```
Human judgment: "cat" and "dog" similarity = 7/10
Model similarity: cosine(cat_vector, dog_vector) = 0.73
Correlation: How well do model scores match human scores?
```

**Analogy Tasks:**
```
Question: "man" : "king" :: "woman" : ?
Method: king - man + woman = ?
Correct if closest word is "queen"
```

#### Extrinsic Evaluation
```
Use embeddings in downstream tasks:
- Sentiment analysis
- Named entity recognition  
- Text classification

Better embeddings → Better downstream performance
```

---

## 3.3 Language Modeling: Predicting What Comes Next 🔮

### What is Language Modeling?

#### The Core Task
```
Given: "The cat sat on the"
Predict: "mat" (most likely), "floor", "chair", etc.

Language model assigns probabilities to sequences of words
```

#### Why This Matters
```
If you can predict the next word well:
✅ You understand grammar
✅ You understand semantics  
✅ You understand context
✅ You can generate coherent text!

This is the foundation of GPT, ChatGPT, and all modern LLMs!
```

### N-gram Language Models (The Classical Approach)

#### The Markov Assumption
```
Assumption: Next word depends only on previous N words
P(word | entire history) ≈ P(word | previous N words)
```

#### Bigram Model (N=2)
```
P(word_i | word_1, ..., word_{i-1}) ≈ P(word_i | word_{i-1})

Example:
P("cat" | "The") = Count("The cat") / Count("The")

If we saw "The cat" 100 times and "The" 1000 times:
P("cat" | "The") = 100/1000 = 0.1
```

#### Training N-gram Models
```
1. Count all N-gram occurrences in training data
2. Estimate probabilities using counts
3. Apply smoothing for unseen N-grams
```

#### Problems with N-gram Models
```
❌ Can't capture long-range dependencies
❌ Sparse data problem (many N-grams never seen)
❌ No semantic understanding
❌ Exponential parameter growth with N
```

### Neural Language Models: The Revolution

#### The Big Idea
```
Instead of counting N-grams:
Use neural networks to predict next word!

Benefits:
✅ Distributed representations
✅ Automatic feature learning
✅ Better generalization
✅ Can handle longer contexts
```

#### Feed-forward Neural Language Model

**Architecture:**
```
Input: Previous N words
↓
Embedding layer: Convert words to vectors
↓  
Concatenate: Combine all word vectors
↓
Hidden layers: Learn complex patterns
↓
Output layer: Probability distribution over vocabulary
```

**Example:**
```
Context: "The cat sat"
Word embeddings: [e_the, e_cat, e_sat]
Concatenated: [e_the || e_cat || e_sat]  
Hidden layer: Learn patterns like "animals sit on things"
Output: P(next_word = "on") = 0.8
```

#### Recurrent Neural Language Models

**The Innovation: Memory!**
```
Problem with feed-forward: Fixed context window
Solution: RNN can theoretically handle unlimited context!
```

**How RNNs Work:**
```
hidden_0 = initial_state
for each word in sequence:
    hidden_i = RNN(word_i, hidden_{i-1})
    predict_next = output_layer(hidden_i)
```

**Benefits:**
```
✅ Variable-length sequences
✅ Shared parameters across positions  
✅ Theoretical unlimited memory
✅ Sequential processing matches language nature
```

**Problems:**
```
❌ Vanishing gradients (forgets long-term info)
❌ Sequential processing (can't parallelize)
❌ Still struggles with very long dependencies
```

### Modern Language Models: Transformers

#### The Transformer Revolution
```
Key insight: We don't need recurrence!
Self-attention can capture all relationships directly!

Every word can directly "talk" to every other word
No more forgetting long-term dependencies!
```

#### Autoregressive Generation
```
How GPT generates text:

1. Start with prompt: "The cat"
2. Predict next word: P(sat | The cat) → "sat"  
3. Add to sequence: "The cat sat"
4. Predict next: P(on | The cat sat) → "on"
5. Continue: "The cat sat on the mat"
```

---

## 3.4 Evaluation Metrics: How Good is Our Model? 📊

### Intrinsic Evaluation: Perplexity

#### What is Perplexity?
```
Perplexity measures: "How surprised is the model by the actual text?"

Low perplexity = Model predicts text well
High perplexity = Model is confused by the text
```

#### Mathematical Definition
```
Perplexity = 2^(-average log probability)

Example:
If model assigns probability 0.25 to each word:
Perplexity = 1/0.25 = 4

Interpretation: "Model is as confused as random choice among 4 options"
```

#### Intuitive Example
```
Text: "The cat sat on the mat"
Good model: P = [0.9, 0.8, 0.7, 0.9, 0.8, 0.9]
Bad model: P = [0.1, 0.2, 0.3, 0.1, 0.2, 0.1]

Good model → Low perplexity
Bad model → High perplexity
```

### Extrinsic Evaluation: Downstream Tasks

#### Text Generation Quality

**BLEU Score (for Translation):**
```
Measures N-gram overlap between generated and reference text

Example:
Reference: "The cat is sleeping"
Generated: "The cat sleeps"  
BLEU considers: How many 1-grams, 2-grams, etc. match?
```

**Problems with BLEU:**
```
❌ Only surface-level matching
❌ Doesn't understand semantic equivalence
❌ "The cat sleeps" vs "The feline rests" = low BLEU but same meaning!
```

**ROUGE (for Summarization):**
```
Similar to BLEU but focuses on recall
"Did the summary include the important content?"
```

**BERTScore (Modern Approach):**
```
Uses embeddings to measure semantic similarity
Can recognize that "cat" and "feline" are similar!
Much better correlation with human judgment
```

#### Human Evaluation

**What Humans Judge:**
```
1. Fluency: Does the text sound natural?
2. Coherence: Does it make logical sense?
3. Factual Accuracy: Are the facts correct?
4. Relevance: Does it answer the question?
```

**Challenges:**
```
❌ Expensive and time-consuming
❌ Subjective (humans disagree!)
❌ Hard to scale
❌ But most reliable for quality assessment
```

### The Evaluation Challenge

#### Why Evaluation is Hard
```
Language is:
- Subjective (multiple good answers)
- Contextual (meaning depends on situation)  
- Creative (novel combinations are good!)
- Nuanced (subtle differences matter)

How do you measure creativity and understanding? 🤔
```

#### Current Best Practices
```
1. Use multiple metrics (no single metric is perfect)
2. Include human evaluation for final assessment
3. Task-specific metrics when possible
4. Consider both automatic and human metrics
5. Be aware of metric limitations
```

---

## Putting It All Together: The NLP Pipeline 🔄

### From Raw Text to Model Predictions

#### Step 1: Text Preprocessing
```
Raw text: "Hello world! How are you today???"
Cleaned: "Hello world! How are you today?"
Normalized: Remove excessive punctuation, fix encoding
```

#### Step 2: Tokenization  
```
Text: "Hello world! How are you today?"
Tokens: ["Hello", " world", "!", " How", " are", " you", " today", "?"]
```

#### Step 3: Vocabulary Creation
```
Collect all unique tokens from training data
Add special tokens: [PAD], [UNK], [BOS], [EOS]
Final vocabulary size: ~30K-50K tokens
```

#### Step 4: Embedding
```
Each token gets mapped to dense vector:
"Hello" → [0.2, -0.1, 0.8, 0.3, ...]
" world" → [0.1, 0.5, -0.2, 0.7, ...]
```

#### Step 5: Model Processing
```
Embeddings → Transformer layers → Output probabilities
```

#### Step 6: Decoding
```
Probabilities → Sample next token → Convert back to text
```

---

## Common Student Questions 🙋‍♀️

### Q: "Why do we need so many different tokenization methods?"
**A:** Different methods work better for different languages and tasks. It's like having different tools for different jobs!

### Q: "How do embeddings actually capture meaning?"
**A:** Through training! Words that appear in similar contexts get similar embeddings. The model learns that "king" and "queen" are similar because they appear in similar situations.

### Q: "Why is language modeling so important?"
**A:** If you can predict what comes next in language really well, you understand grammar, semantics, context, and world knowledge. It's a powerful general task!

### Q: "Which evaluation metric should I use?"
**A:** Depends on your task! For generation: BLEU/ROUGE + human eval. For understanding: task-specific accuracy. Always use multiple metrics!

---

## Key Takeaways 🎯

1. **Tokenization** is crucial - it determines how your model sees language. Subword methods like BPE are usually best.

2. **Word embeddings** capture semantic similarity by mapping words to dense vectors based on distributional similarity.

3. **Language modeling** (predicting next words) is a powerful task that teaches models about language structure and meaning.

4. **Evaluation** is challenging but essential - use multiple metrics and include human judgment when possible.

5. **The NLP pipeline** connects all these pieces to transform raw text into model understanding.

---

## Fun Exercises 🎮

### Exercise 1: Tokenization Practice
```
Try tokenizing this sentence with different methods:
"The AI model's performance was extraordinary!"

Word-level: ?
Character-level: ?  
Subword (guess): ?
```

### Exercise 2: Embedding Intuition
```
Which pairs should have similar embeddings?
a) "king" and "queen"
b) "king" and "car"  
c) "happy" and "joyful"
d) "run" and "running"
```

### Exercise 3: Language Modeling
```
What should come next?
"The weather today is very ___"

What factors affect this prediction?
```

---

## What's Next? 📚

In Chapter 4, we'll dive deep into the Transformer architecture - the breakthrough that made modern LLMs possible!

**Preview:** We'll learn about:
- The attention mechanism (how models "focus")
- Self-attention (how words relate to each other)
- The complete transformer architecture
- Why this was such a revolutionary breakthrough

Get ready to understand the engine that powers ChatGPT! 🚀

---

## Final Thought 💭

```
"The best way to understand language models is to remember: 
they're pattern matching machines that learned patterns from massive amounts of text.
The patterns they learned happen to correspond to grammar, meaning, and knowledge!"

That's both amazing and important to remember for their limitations! 😊
```
