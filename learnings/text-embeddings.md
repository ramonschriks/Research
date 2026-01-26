# Text Embeddings — How They Work
**Source:** Personal Learning  
**Category:** AI / Natural Language Processing

---

## Context
Understanding *what* text embeddings are and *why* they work is essential when building modern search, recommendation, or AI-assisted systems.

This learning focuses on intuition over theory.

---

## Key Learning: What Text Embeddings Are
Text embeddings are **numerical representations of text** (vectors) that capture *meaning* rather than exact wording.

Similar meanings → similar numbers  
Different meanings → very different numbers

---

## Intuition: The Meaning Map

Think of embeddings as placing text on a map:

```
                Finance
                   |
     "stock market" •
                   |
                   |
                   |
"cat on sofa" •     |
                   |
                   |
                   • "dog sleeping"
                   |
                Animals
```

Texts with similar meanings end up **close together**.  
Distance = semantic difference.

---

## How Text Embeddings Work (Step-by-Step)

### 1. Tokenization
Text is split into tokens (words or word-parts):

```
"The cat sits on the sofa"
→ ["the", "cat", "sits", "on", "the", "sofa"]
```

---

### 2. Neural Encoding
A trained neural network converts tokens into numbers:

```
"The cat sits on the sofa"
→ [0.12, -0.87, 0.44, 0.01, ...]
```

This vector may contain hundreds or thousands of numbers.

---

### 3. Vector Space Placement
Each text becomes a **point in high-dimensional space**.

- "cat" ≈ "dog" → close
- "cat" ≠ "bank" → far

---

### 4. Similarity Comparison
Applications compare embeddings using math (e.g. cosine similarity):

```
similarity("cat on couch", "dog on sofa") → HIGH
similarity("cat on couch", "tax regulations") → LOW
```

---

## What Embeddings Capture
- Semantic meaning
- Context
- Relationships between concepts

They **do not** store:
- Exact wording
- Grammar rules
- Recoverable original text

---

## Why This Is Powerful
Embeddings enable:

- Semantic search (meaning-based)
- Document clustering
- Recommendations
- Question answering over documents (RAG)

All using simple vector math.

---

## Tiny Code Example (Conceptual)

```ts
// Pseudo-code
const embedding1 = embed("I love pizza");
const embedding2 = embed("Pizza is my favorite food");

const similarity = cosineSimilarity(embedding1, embedding2);
// similarity → very high
```

No keyword matching — just meaning.

---

## Core Insight
> **Embeddings turn language into math while preserving meaning.**

This allows computers to *reason about text* instead of just matching words.

---

## TL;DR
- Text → numbers
- Meaning → distance
- Similar meaning → close vectors
- Computers compare meaning using math
