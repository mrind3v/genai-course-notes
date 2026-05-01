Models do not understand text, they understand vectors of numbers. So we need a way to convert text into vectors.

---

## 1. Tokenization: The "Vocabulary" Phase
Tokenization is the process of breaking raw text into smaller chunks called **tokens**. 
*   **Why not words?** Words like "unhappy" and "happily" share a root. Sub-word tokenization allows the model to learn the meaning of "happy" once, rather than treating every variation as a brand-new concept.
*   **Why not characters?** Character-level processing is too granular and loses semantic meaning.

### Using `tiktoken` in JavaScript
OpenAI uses a specific sub-word tokenization method called **Byte Pair Encoding (BPE)**. In JS, you can use the `tiktoken` library (or its WASM wrapper) to see this in action.

**Setup:**
```bash
npm install tiktoken
```

**Implementation:**
```javascript
import { encoding_for_model } from "tiktoken";

const modelName = "gpt-4o"; // Or "gpt-3.5-turbo"
const enc = encoding_for_model(modelName);

const text = "Tokenization is awesome!";
const tokens = enc.encode(text);

console.log("Token IDs:", tokens); 
// Output looks like: [40, 1045, 2341, ...] (Numerical IDs)

// To see the actual strings each ID represents:
tokens.forEach(id => {
  console.log(`ID: ${id} -> Text: "${new TextDecoder().decode(enc.decode(new Uint32Array([id])))}"`);
});

enc.free(); // Free memory (important for WASM)
```

---

## 2. Input Embeddings: The "Meaning" Phase
Once you have Token IDs (integers), the model looks them up in a massive **Embedding Matrix**. 

*   **The Lookup:** Imagine a giant table where every Token ID has a corresponding row. That row is a dense vector of floating-point numbers (e.g., a vector of size 768 or 1536).
*   **The Goal:** These numbers represent "semantic space." In a well-trained embedding space, the vector for "King" and "Queen" will be mathematically closer to each other than the vector for "King" and "Apple."

---

## 3. Positional Encoding: The "Order" Phase
Transformers process all tokens in a sentence **simultaneously** (parallelization). Because they don't read from left to right like humans (or older RNNs), they "forget" the order of words. Without positional encoding, "The dog bit the man" and "The man bit the dog" look identical to the model.

**Positional Encoding** is a unique "signature" added to the embedding vector to tell the model *where* in the sentence a word is.

### How it works (The Sinusoidal Method)
Instead of just adding 1, 2, 3 (which can get too large for long sequences), the original Transformer paper uses **Sine and Cosine functions** of different frequencies.


For a position $pos$ and dimension $i$:
$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$
$$PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

*   **Result:** Every position gets a unique vector. Because these are wave functions, the model can easily learn **relative positions** (e.g., it can "feel" that one word is always three spots away from another).

---

## Summary of the Flow
1.  **Text:** "I love Arch"
2.  **Tokenization:** `[73, 3021, 15432]` (Integers)
3.  **Embedding:** `[[0.12, -0.5], [0.9, 0.1], ...]` (Semantic vectors)
4.  **Positional Encoding:** Add a "position" vector to the semantic vector.
