This is exactly the right question. You're asking about the engineering decisions behind RAG, not just the buzzword. These decisions have a huge impact on how well your system performs.

Let's walk through the entire pipeline.

---

# Step 1: You have a document

Suppose you have this PDF:

```text
Spring Boot Guide

Chapter 1 - Introduction
...

Chapter 2 - Dependency Injection
...

Chapter 3 - Spring Security
...

Chapter 4 - JWT Authentication
...
```

Your goal is to let users ask:

> "How does JWT authentication work?"

The first challenge is:

> **How do I divide this document?**

---

# Option 1: One giant chunk ❌

```
Entire PDF
↓
1 chunk
```

Problems:

* Too large
* Expensive
* Retrieval is poor
* Doesn't fit into the model's context window

---

# Option 2: One word at a time ❌

```
Spring
Boot
uses
Java
...
```

This loses all context.

If you search for:

> Explain JWT

You'll retrieve

```
JWT
```

which is meaningless by itself.

---

# Option 3: One sentence at a time ⚠️

```
Spring Boot is a framework.

Dependency Injection is...

JWT is...
```

Better, but often still too small.

Suppose the explanation spans five sentences.

The retrieval might only return one of them.

---

# Option 4: Paragraphs ✅

Most systems start here.

```
Paragraph 1

↓

Chunk 1

Paragraph 2

↓

Chunk 2

Paragraph 3

↓

Chunk 3
```

Each chunk contains a complete idea.

---

# Option 5: Fixed-size chunks (most common)

Instead of paragraphs, most production systems split by **token count**.

Example:

```
500 tokens

↓

Chunk 1

500 tokens

↓

Chunk 2

500 tokens

↓

Chunk 3
```

Why?

Paragraphs vary a lot.

Some are:

```
20 words
```

Some are:

```
700 words
```

Token-based chunks provide more consistent sizes.

Typical values are:

* 200 tokens
* 300 tokens
* 500 tokens
* 800 tokens

depending on the application.

---

# Why overlap chunks?

Suppose your document says:

```
Chunk 1

JWT consists of three parts...

--------------------------------

Chunk 2

...signature verifies authenticity...
```

The explanation gets cut in half.

Instead, use overlap.

```
Chunk 1

1–500 tokens

Chunk 2

450–950 tokens

Chunk 3

900–1400 tokens
```

Notice:

```
450–500
```

appears in both chunks.

This preserves continuity.

Most systems use about **10–20% overlap**.

---

# Now comes the embedding

Suppose your chunk is

```
JWT consists of three parts:
Header
Payload
Signature
```

You **do not** write an algorithm to convert this into numbers.

Instead, you use an **embedding model**.

Example:

```
Text

↓

Embedding Model

↓

Vector
```

---

# What is an embedding model?

Think of it as another neural network.

LLMs generate text.

Embedding models generate vectors.

Example:

```
Text

↓

Embedding Model

↓

[0.27,
-0.91,
0.55,
...
768 numbers]
```

That's all it does.

---

# Who trains the embedding model?

Companies train them on **millions or billions** of text examples.

During training, the model learns:

```
Dog

Cat

↓

Similar vectors
```

```
Java

Spring Boot

↓

Similar vectors
```

```
Pizza

Football

↓

Very different vectors
```

The exact training process is much more involved, but the outcome is that semantically similar text ends up with nearby vectors.

---

# Are there rules?

Not rules you implement yourself.

Instead, you choose an embedding model.

Examples include:

* `all-MiniLM-L6-v2`
* `bge-small-en`
* `bge-base`
* `e5-large`
* `nomic-embed-text`
* `mxbai-embed-large`

Each has:

* a fixed vector size (384, 768, 1024, etc.)
* different accuracy
* different speed

---

# Example

Suppose you use

```
nomic-embed-text
```

Input:

```
Spring Security uses JWT authentication.
```

Output:

```
[
0.124,
-0.834,
0.277,
...
768 values
]
```

Another sentence:

```
JWT authentication in Spring Boot
```

might become

```
[
0.119,
-0.831,
0.274,
...
768 values
]
```

Notice how the vectors are very similar.

That's exactly what we want.

---

# Why don't we embed every word?

Because meaning comes from context.

Consider:

```
Apple
```

Is it:

* 🍎 the fruit?
* 💻 the company?

The surrounding text disambiguates it:

```
Apple released a new MacBook.
```

vs.

```
Apple is rich in fiber.
```

Embedding the whole chunk captures that context.

---

# A complete RAG pipeline

```
PDF

↓

Extract text

↓

Split into chunks
(500 tokens + 50 overlap)

↓

Embedding model

↓

Vectors

↓

Vector Database
```

When the user asks a question:

```
Question

↓

Embedding model

↓

Question vector

↓

Vector DB

↓

Top 5 similar chunks

↓

LLM (Ollama)

↓

Final answer
```

Notice something important:

**The same embedding model is typically used for both the stored document chunks and the user's query.** That way, both are represented in the same vector space, making similarity search meaningful.

---

# A question you might be thinking

> If the embedding model outputs 768 numbers, what do those numbers actually mean?

Individually, **nothing interpretable**.

Unlike RGB values in an image, you can't say:

* Dimension 1 = "Java"
* Dimension 27 = "Security"

Each dimension is a learned feature. The **pattern across all 768 values** represents the meaning of the text. During training, the model adjusts these values so that text with similar meanings ends up close together in the vector space.

Think of it like GPS coordinates:

```
Delhi
→ (28.61, 77.20)

Mumbai
→ (19.07, 72.88)
```

The individual latitude and longitude values aren't meaningful by themselves, but together they pinpoint a location. Embeddings work similarly: the entire vector represents semantic meaning, and the distance between vectors tells you how related two pieces of text are.

---

Since you're already experimenting with Ollama, the next thing I'd suggest is building a tiny RAG system from scratch **without using LangChain**. Just use:

* Spring Boot (or Python if you want a quick prototype)
* a PDF parser
* a chunking function
* an embedding model
* Chroma or Qdrant
* Ollama

Building it yourself first will make frameworks like LangChain much easier to understand later, because you'll know exactly what they're doing under the hood.

