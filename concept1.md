This is one of the most important concepts in modern AI. Once you understand vector databases, you'll understand how ChatGPT can answer questions about documents it has never been trained on.

Let's build it from the ground up.

---

# Imagine a normal database

Suppose you have a MySQL table:

| ID | Name  | Age |
| -- | ----- | --- |
| 1  | Shrey | 22  |
| 2  | Rahul | 25  |

You search like this:

```sql
SELECT * FROM users
WHERE name = 'Shrey';
```

The database looks for an **exact value**.

---

# But AI doesn't search for exact words

Suppose your notes contain:

```
Spring Security uses JWT authentication.
```

Now you ask:

> How does token-based authentication work?

Notice something?

The words **JWT** and **token-based authentication** are related, but they're not identical.

A normal SQL database doesn't understand that relationship.

---

# AI converts text into numbers

This is called an **embedding**.

An embedding model reads a sentence and converts it into a list of numbers.

For example:

```
"I love Java"

↓

[0.21, -0.91, 0.72, 0.11, ... 768 numbers]
```

Another sentence:

```
"I enjoy programming in Java"

↓

[0.19, -0.89, 0.70, 0.13, ...]
```

Although the words are different, the vectors are very similar.

---

# Why numbers?

Imagine every sentence becomes a point in a giant space.

Instead of this:

```
Sentence
```

you have this:

```
Sentence

↓

(0.21, -0.91, 0.72, ...)
```

Now similar meanings end up close together.

Imagine a simplified 2D picture:

```
             Dogs

      ●
            ●

Cats                 Animals
       ●      ●

                     Programming

                              ● Java

                               ● Spring Boot

                                   ● REST API
```

The actual space has hundreds or thousands of dimensions, not just two.

---

# Now imagine thousands of documents

Suppose you upload

```
Resume.pdf

Hospital_Project.pdf

Spring Notes.pdf

Docker Guide.pdf

Internship Report.pdf
```

Each paragraph becomes a vector.

```
Paragraph 1

↓

[0.22, ...]

Paragraph 2

↓

[-0.61, ...]

Paragraph 3

↓

[0.91, ...]
```

Now you have thousands of vectors.

Where do you store them?

In a **vector database**.

---

# What does a vector database actually store?

Instead of

```
ID
Name
Age
```

it stores

```
ID

Vector

Original text

Metadata
```

Example:

```
ID: 145

Vector:
[0.22, -0.51, 0.88, ...]

Text:
Spring Security uses JWT.

File:
security.pdf

Page:
13
```

---

# What happens when you ask a question?

Suppose you ask:

```
How does JWT authentication work?
```

Step 1

Convert your question into a vector.

```
Question

↓

Embedding

↓

[0.24, -0.53, 0.91, ...]
```

Step 2

Search for the **closest vectors**.

The vector database might find:

```
Result 1

Spring Security uses JWT authentication.

Similarity:
98%
```

```
Result 2

JWT contains three parts.

Similarity:
95%
```

```
Result 3

OAuth login flow.

Similarity:
92%
```

---

# Then Ollama receives only those results

Instead of sending your entire PDF collection, you send only the most relevant passages.

```
User

↓

Question

↓

Embedding

↓

Vector DB

↓

Top 5 matching paragraphs

↓

Ollama

↓

Answer
```

This is called **RAG (Retrieval-Augmented Generation)**.

---

# Why not send the whole PDF?

Suppose you have

```
100 PDFs

↓

20,000 pages
```

Llama can't process all of that at once because every model has a **context window** (a limit on how much text it can read in one prompt).

Instead, the vector database quickly finds the few paragraphs that are most relevant.

---

# How does it know what is "close"?

It compares vectors using similarity measures such as:

* **Cosine similarity** (the most common)
* Euclidean distance
* Dot product

Think of it like comparing directions. Two vectors pointing in nearly the same direction represent text with similar meaning, even if they don't use the same words.

---

# Popular vector databases

Some of the most widely used options are:

| Database | Best for                            |
| -------- | ----------------------------------- |
| Chroma   | Learning and small local projects   |
| Qdrant   | Fast, production-ready, open source |
| Milvus   | Large-scale AI systems              |
| Pinecone | Managed cloud service               |
| Weaviate | Rich semantic search features       |
| pgvector | Adds vector search to PostgreSQL    |

For someone learning AI, I'd recommend starting with **Chroma** because it's easy to set up locally. When you build production applications, **Qdrant** is an excellent next step.

---

# Here's a real example for you

Imagine you build an AI assistant for your own resume.

You upload:

```
Resume.pdf

Internship Report.pdf

Hospital Project.pdf

Movie Recommendation Project.pdf
```

A recruiter asks:

> Did you use RabbitMQ anywhere?

The system works like this:

```
Recruiter
     │
     ▼
"Did you use RabbitMQ?"
     │
     ▼
Embedding Model
     │
     ▼
Vector Database
     │
     ▼
Finds:

• Implemented asynchronous API calls using RabbitMQ.
• Reduced response time by 40%.
     │
     ▼
Ollama
     │
     ▼
"Yes. RabbitMQ was used to handle asynchronous API calls in the Movie Recommendation project, reducing response time by approximately 40%."
```

Notice that Ollama didn't memorize your resume. It answered by retrieving the relevant passages first.

---

## Why vector databases are such a big deal

Traditional databases answer:

> **"Where is the exact data?"**

Vector databases answer:

> **"Which pieces of information mean something similar to what I'm asking?"**

That ability to search by **meaning** instead of exact keywords is what powers document chatbots, AI coding assistants, enterprise search, recommendation systems, and many modern AI applications.

Since you're already running Ollama locally and have a Spring Boot background, a great next project would be to build a RAG application where users upload PDFs, the application stores embeddings in a vector database, and Ollama answers questions based on those documents. It's one of the most valuable AI projects you can add to your portfolio because it combines backend engineering with modern LLM techniques.

