# RAG (Retrieval-Augmented Generation)

## What is RAG?

**RAG stands for Retrieval-Augmented Generation**

It's a technique that enhances AI language models by giving them access to external information sources. Instead of relying solely on their training data, RAG systems can look up relevant information before generating responses.

### The Core Idea

Think of RAG like the difference between:
- **Closed-book exam**: You only know what you memorized
- **Open-book exam**: You can look things up in textbooks before answering

Traditional AI models work like closed-book exams. RAG makes them work like open-book exams.

## Why Do We Need RAG?

### Problems with Traditional LLMs (Large Language Models)

1. **Knowledge Cutoff**: Models only know information from their training data
   - GPT-4 trained on data up to April 2023 doesn't know what happened yesterday
   
2. **Hallucinations**: Models sometimes make up plausible-sounding but incorrect information
   - "The Eiffel Tower is 500 meters tall" (it's actually 330m)
   
3. **No Domain-Specific Knowledge**: Can't access your company's private documents
   - Your internal policies, procedures, customer data
   
4. **Can't Update Easily**: Retraining models is expensive and time-consuming
   - Would need billions of dollars and months to update knowledge

### How RAG Solves These Problems

✅ **Up-to-date information**: Retrieve from current sources  
✅ **Grounded answers**: Responses based on actual documents, not made-up facts  
✅ **Domain expertise**: Access your private knowledge base  
✅ **Easy updates**: Just update your document database, no model retraining needed  
✅ **Source attribution**: Can cite where information came from  

## How RAG Works: The Complete Flow

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────┐
│                     1. INDEXING PHASE                    │
│                   (Done once, offline)                   │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   Collect Documents               │
        │   (PDFs, websites, databases)     │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Split into Chunks               │
        │   (Break into smaller pieces)     │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Create Embeddings               │
        │   (Convert text to numbers)       │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Store in Vector Database        │
        │   (Save for fast searching)       │
        └───────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   2. QUERY PHASE                         │
│                  (Done every query)                      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   User asks a question            │
        │   "What is our refund policy?"    │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Convert question to embedding   │
        │   (Same process as documents)     │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Search Vector Database          │
        │   (Find most similar chunks)      │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Retrieve Top K Documents        │
        │   (e.g., top 5 most relevant)     │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Combine Question + Documents    │
        │   (Create augmented prompt)       │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Send to LLM                     │
        │   (GPT, Claude, etc.)             │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │   Generate Answer                 │
        │   (Based on retrieved context)    │
        └───────────────────────────────────┘
```

## Deep Dive: Each Component

### 1. Document Preparation

**Document Sources:**
- PDF files
- Word documents
- Websites/web pages
- Databases
- APIs
- Email archives
- Chat logs

**Chunking Strategy:**

Why chunk? Because:
- LLMs have token limits (can't process entire books)
- Smaller chunks = more precise retrieval
- Better matching between questions and relevant text

**Common chunking methods:**

```python
# Fixed-size chunking (simple)
chunk_size = 500 words
overlap = 50 words  # Prevent cutting sentences

# Semantic chunking (smarter)
# Split by paragraphs, sections, or topics
# Keeps related information together
```

**Example:**
```
Original Document (2000 words)
        ↓
Chunk 1 (words 1-500)
Chunk 2 (words 450-950)    ← 50 word overlap
Chunk 3 (words 900-1400)   ← 50 word overlap
Chunk 4 (words 1350-1850)
...
```

### 2. Embeddings: The Magic Behind RAG

**What are embeddings?**

Embeddings convert text into numbers (vectors) that capture meaning.

```
"dog" → [0.2, 0.8, 0.1, 0.4, ...]  (768 numbers)
"puppy" → [0.25, 0.75, 0.15, 0.42, ...]  (similar numbers!)
"car" → [0.9, 0.1, 0.7, 0.2, ...]  (different numbers)
```

**Key property**: Similar meanings = similar numbers

This means we can find similar text by comparing numbers!

**Popular embedding models:**
- OpenAI's text-embedding-ada-002
- Sentence-BERT
- Google's Universal Sentence Encoder
- Cohere embeddings

**Dimensions matter:**
- Small (384): Fast, less accurate
- Medium (768): Good balance
- Large (1536): More accurate, slower

### 3. Vector Database

**What is it?**

A specialized database optimized for storing and searching vectors (embeddings).

**Why not use regular databases?**

Finding similar vectors requires comparing millions of numbers. Vector databases use clever algorithms (like HNSW, IVF) to search quickly.

**Popular vector databases:**
- **Pinecone**: Managed, cloud-based
- **Weaviate**: Open-source, feature-rich
- **Chroma**: Simple, lightweight
- **Qdrant**: Fast, written in Rust
- **FAISS**: Facebook's library, very fast
- **Milvus**: Scalable, open-source

**What they store:**
```json
{
  "id": "doc_123_chunk_5",
  "embedding": [0.2, 0.8, ...],
  "text": "Our refund policy allows...",
  "metadata": {
    "source": "policy_doc.pdf",
    "page": 5,
    "date": "2024-01-15"
  }
}
```

### 4. Retrieval Process

**Similarity Search:**

When user asks: "What is your refund policy?"

1. Convert question to embedding: [0.21, 0.79, ...]
2. Find nearest neighbor vectors in database
3. Return top K most similar chunks

**Distance metrics:**
- **Cosine similarity**: Measures angle between vectors (most common)
- **Euclidean distance**: Straight-line distance
- **Dot product**: Fast, works well for normalized vectors

**Example retrieval:**
```
Query: "refund policy"

Top 3 Results:
1. Score: 0.92 → "Our refund policy allows returns within 30 days..."
2. Score: 0.87 → "To request a refund, please email support@..."
3. Score: 0.81 → "Refunds are processed within 5-7 business days..."
```

### 5. Augmented Generation

**Building the Prompt:**

The retrieved documents are added to the LLM prompt:

```
System: You are a helpful assistant. Answer based on the provided context.

Context:
---
Document 1: Our refund policy allows returns within 30 days of purchase...
Document 2: To request a refund, please email support@company.com...
Document 3: Refunds are processed within 5-7 business days...
---

User Question: What is your refund policy?

Instructions: Answer the question using ONLY the information in the context above.
If the context doesn't contain the answer, say "I don't have that information."
```

**The LLM then generates:**
> "Our refund policy allows returns within 30 days of purchase. To request a refund, 
> email support@company.com. Refunds are typically processed within 5-7 business days."

## RAG Architecture Patterns

### Pattern 1: Naive RAG (Basic)

```
Query → Embed → Search → Retrieve → Augment Prompt → Generate
```

**Pros**: Simple, fast  
**Cons**: May retrieve irrelevant docs, no refinement

### Pattern 2: Advanced RAG

Adds pre-retrieval and post-retrieval processing:

```
Query 
  → Query Rewriting (improve search)
  → Embed
  → Search
  → Retrieve
  → Re-rank Results (reorder by relevance)
  → Filter & Deduplicate
  → Augment Prompt
  → Generate
  → Cite Sources
```

**Improvements:**
- **Query rewriting**: "refund policy" → "return and refund procedures"
- **Re-ranking**: Use a more powerful model to re-score results
- **Filtering**: Remove low-quality or redundant chunks
- **Citation**: Link answer to specific sources

### Pattern 3: Modular RAG

Mix and match components for specific needs:

```
┌─────────────────────┐
│   Query Module      │ ← Query expansion, decomposition
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  Retrieval Module   │ ← Multiple retrievers (semantic, keyword, SQL)
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Ranking Module    │ ← Re-rank, filter, diversify
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│ Generation Module   │ ← Generate with citations
└─────────────────────┘
```

## Implementation Example (Python)

### Simple RAG with LangChain

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# 1. Load documents
loader = PyPDFLoader("company_policy.pdf")
documents = loader.load()

# 2. Split into chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
chunks = text_splitter.split_documents(documents)

# 3. Create embeddings and store in vector DB
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings
)

# 4. Create retrieval chain
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(temperature=0),
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)

# 5. Ask questions
question = "What is the refund policy?"
answer = qa_chain.run(question)
print(answer)
```

### Simple RAG from Scratch

```python
import numpy as np
from sentence_transformers import SentenceTransformer
import openai

# 1. Initialize embedding model
embedder = SentenceTransformer('all-MiniLM-L6-v2')

# 2. Prepare documents
documents = [
    "Our refund policy allows returns within 30 days.",
    "Shipping takes 3-5 business days.",
    "Customer service is available 24/7."
]

# 3. Create embeddings
doc_embeddings = embedder.encode(documents)

# 4. User query
query = "What is the return policy?"
query_embedding = embedder.encode([query])[0]

# 5. Find most similar documents (cosine similarity)
similarities = np.dot(doc_embeddings, query_embedding)
top_k = 2
top_indices = np.argsort(similarities)[-top_k:][::-1]

# 6. Retrieve relevant docs
retrieved_docs = [documents[i] for i in top_indices]

# 7. Create augmented prompt
context = "\n".join(retrieved_docs)
prompt = f"""Answer the question based on the context below.

Context:
{context}

Question: {query}

Answer:"""

# 8. Generate answer
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)

print(response.choices[0].message.content)
```

## Advanced RAG Techniques

### 1. Hybrid Search

Combine semantic search with keyword search:

```python
# Semantic search (embeddings)
semantic_results = vectorstore.similarity_search(query, k=5)

# Keyword search (BM25)
keyword_results = bm25.search(query, k=5)

# Combine and re-rank
final_results = rerank(semantic_results + keyword_results)
```

**Why?** Semantic search understands meaning, keyword search finds exact matches.

### 2. Query Transformation

Improve retrieval by rewriting queries:

**Original**: "How do I return stuff?"  
**Rewritten**: "What is the product return and refund process?"

**Techniques:**
- Query expansion (add synonyms)
- Query decomposition (break complex queries into sub-queries)
- Step-back prompting (ask more general question first)

### 3. Re-ranking

Use a more sophisticated model to re-score retrieved documents:

```python
# Initial retrieval (fast, lower quality)
initial_results = vectorstore.similarity_search(query, k=20)

# Re-rank with better model (slower, higher quality)
reranked = cross_encoder.rank(query, initial_results)
final_results = reranked[:5]  # Take top 5
```

### 4. Multi-Query RAG

Generate multiple variations of the user's question:

```
Original: "What's the refund policy?"

Generated queries:
- "How do I get a refund?"
- "What is the return procedure?"
- "Can I get my money back?"

→ Retrieve docs for all queries
→ Combine and deduplicate
→ Generate answer from broader context
```

### 5. Contextual Compression

Remove irrelevant parts of retrieved documents:

```python
# Retrieved chunk (500 words)
chunk = "Our company was founded in 1990... [lots of text] ...Our refund 
         policy allows 30-day returns..."

# Compress to relevant parts only
compressed = compressor.compress(chunk, query)
# Result: "Our refund policy allows 30-day returns..."
```

### 6. Metadata Filtering

Add filters to retrieval:

```python
# Filter by date
results = vectorstore.similarity_search(
    query,
    filter={"date": {"$gte": "2024-01-01"}}
)

# Filter by source
results = vectorstore.similarity_search(
    query,
    filter={"source": "official_policy.pdf"}
)
```

## Evaluation Metrics for RAG

### Retrieval Quality

**1. Recall@K**
- What % of relevant documents are in the top K results?
- Recall@5 = 80% means 4 out of 5 relevant docs were retrieved

**2. Precision@K**
- What % of retrieved documents are actually relevant?
- Precision@5 = 60% means 3 out of 5 retrieved docs were relevant

**3. Mean Reciprocal Rank (MRR)**
- Average position of the first relevant document
- Higher is better

**4. NDCG (Normalized Discounted Cumulative Gain)**
- Considers ranking quality (better docs should rank higher)

### Generation Quality

**1. Faithfulness**
- Is the answer supported by the retrieved documents?
- No hallucinations

**2. Answer Relevance**
- Does the answer address the question?

**3. Context Relevance**
- Are the retrieved documents actually relevant?

**4. BLEU/ROUGE**
- Compare generated answer to reference answer

### End-to-End Evaluation

**RAGAS (RAG Assessment)**
- Combines multiple metrics
- Evaluates retrieval and generation together

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevance

result = evaluate(
    dataset=test_dataset,
    metrics=[faithfulness, answer_relevance]
)
```

## Common RAG Challenges & Solutions

### Challenge 1: Poor Retrieval Quality

**Problem**: System retrieves irrelevant documents

**Solutions:**
- Improve chunking strategy (smaller/larger chunks)
- Use better embedding models
- Implement hybrid search (semantic + keyword)
- Add metadata filtering
- Fine-tune embedding model on your domain

### Challenge 2: Lost in the Middle

**Problem**: LLMs pay more attention to beginning/end of context, ignore middle

**Solutions:**
- Re-order chunks (put most relevant first and last)
- Reduce number of retrieved documents
- Use summarization before generation

### Challenge 3: Hallucinations Despite Context

**Problem**: LLM still makes things up even with retrieved docs

**Solutions:**
- Stronger system prompt: "ONLY use provided context"
- Ask LLM to quote sources
- Use structured output format
- Implement fact-checking step

### Challenge 4: Outdated Information

**Problem**: Vector DB contains old information

**Solutions:**
- Regular re-indexing schedule
- Add timestamps to documents
- Implement cache invalidation
- Hybrid approach: RAG + web search for recent info

### Challenge 5: Scalability

**Problem**: Slow with millions of documents

**Solutions:**
- Use approximate nearest neighbor search (ANN)
- Implement hierarchical indexing
- Use metadata pre-filtering
- Distributed vector databases
- GPU acceleration

### Challenge 6: Multi-hop Questions

**Problem**: Answer requires combining info from multiple documents

**Solutions:**
- Increase number of retrieved documents
- Implement iterative retrieval
- Use graph-based RAG
- Decompose question into sub-questions

## RAG vs. Fine-tuning

### When to use RAG:

✅ Need to incorporate new information frequently  
✅ Large knowledge base that changes often  
✅ Want source attribution  
✅ Limited budget (cheaper than fine-tuning)  
✅ Need to update without retraining  

### When to use Fine-tuning:

✅ Teaching model a new style or format  
✅ Domain-specific language or terminology  
✅ Information is stable (won't change frequently)  
✅ Need extremely fast responses (no retrieval overhead)  
✅ Small, focused knowledge set  

### Best of Both: RAG + Fine-tuning

Fine-tune the model on your domain, then use RAG for up-to-date facts:

```
Fine-tuned Model (knows medical terminology)
        +
RAG (retrieves latest research papers)
        =
Medical assistant with current knowledge
```

## Real-World Applications

### 1. Customer Support

```
Knowledge Base: FAQs, manuals, ticket history
Query: "My printer won't connect to WiFi"
Retrieved: Troubleshooting steps for WiFi connection
Answer: Step-by-step instructions specific to user's printer model
```

### 2. Legal Research

```
Knowledge Base: Case law, statutes, regulations
Query: "Precedents for data privacy violations in California"
Retrieved: Relevant cases, CCPA regulations
Answer: Summarized legal precedent with citations
```

### 3. Enterprise Search

```
Knowledge Base: Internal docs, emails, Slack messages
Query: "What was decided in the Q3 strategy meeting?"
Retrieved: Meeting notes, follow-up emails
Answer: Summary of key decisions and action items
```

### 4. Educational Tutoring

```
Knowledge Base: Textbooks, lecture notes, solved problems
Query: "How do I solve quadratic equations?"
Retrieved: Relevant textbook sections, example problems
Answer: Explanation with step-by-step examples
```

### 5. Medical Diagnosis Support

```
Knowledge Base: Medical literature, patient records
Query: "Treatment options for Type 2 diabetes"
Retrieved: Latest clinical guidelines, research papers
Answer: Evidence-based treatment recommendations with sources
```

## Best Practices

### 1. Document Preparation
- Clean and normalize text
- Maintain metadata (source, date, author)
- Remove duplicate content
- Structure documents logically

### 2. Chunking
- Experiment with chunk size (start with 500-1000 tokens)
- Use overlap (10-20%) to preserve context
- Respect semantic boundaries (don't split mid-sentence)
- Include surrounding context in metadata

### 3. Retrieval
- Start with k=3-5 documents
- Use hybrid search for better coverage
- Implement re-ranking for quality
- Filter by metadata when possible

### 4. Generation
- Clear system prompts
- Instruct to cite sources
- Handle "I don't know" gracefully
- Validate against retrieved context

### 5. Monitoring
- Track retrieval quality metrics
- Monitor for hallucinations
- Log failed queries
- Collect user feedback

## Tools & Frameworks

### Complete RAG Frameworks

**LangChain**
- Most popular, extensive integrations
- Good documentation
- Python and JavaScript

**LlamaIndex**
- Specialized for RAG
- Advanced indexing strategies
- Great for complex queries

**Haystack**
- Enterprise-focused
- Production-ready
- Flexible pipelines

### Vector Databases

**Managed/Cloud:**
- Pinecone (easiest to start)
- Weaviate Cloud
- Qdrant Cloud

**Self-hosted:**
- Chroma (lightweight, great for dev)
- Milvus (scalable, enterprise)
- FAISS (fast, in-memory)

### Embedding Models

**Closed-source:**
- OpenAI text-embedding-ada-002
- Cohere embeddings
- Google Vertex AI embeddings

**Open-source:**
- sentence-transformers
- instructor-xl
- BGE embeddings

## Future of RAG

**Emerging trends:**

1. **Multimodal RAG**: Retrieve images, tables, charts, not just text
2. **Graph RAG**: Use knowledge graphs for better reasoning
3. **Corrective RAG**: Self-correcting retrieval loops
4. **Adaptive RAG**: Dynamically choose retrieval strategy
5. **Long-context RAG**: As LLMs handle more tokens, retrieval strategies evolve

## Summary: Key Takeaways

1. **RAG = Retrieval + Generation**: Combine search with language model generation

2. **Core benefit**: Ground AI responses in actual documents, reducing hallucinations

3. **Main components**: Embeddings, vector database, retriever, LLM

4. **When to use**: Dynamic knowledge, domain-specific info, need citations

5. **Success factors**: Quality chunking, relevant retrieval, clear prompts

6. **Not a silver bullet**: Requires careful design and iteration

## Practice Exercises for Students

### Exercise 1: Design a RAG System
Design a RAG system for a university library. What would you index? How would you chunk? What queries would students ask?

### Exercise 2: Chunking Strategy
Given a 50-page technical manual, design a chunking strategy. Consider: fixed-size vs semantic, chunk size, overlap amount.

### Exercise 3: Evaluation
How would you evaluate if a RAG system for medical Q&A is working well? Define 3 metrics and explain why they matter.

### Exercise 4: Problem Solving
A RAG system retrieves relevant documents but the LLM still hallucinates. List 5 potential causes and solutions.

### Exercise 5: Architecture Choice
When would you choose:
- Basic RAG vs Advanced RAG?
- RAG vs Fine-tuning?
- Semantic search vs Hybrid search?

---

## Additional Resources

**Papers:**
- "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (Lewis et al., 2020)
- "REALM: Retrieval-Augmented Language Model Pre-Training" (Guu et al., 2020)

**Courses:**
- DeepLearning.AI: "Building Applications with Vector Databases"
- "LangChain for LLM Application Development"

**Documentation:**
- LangChain docs: https://python.langchain.com
- LlamaIndex docs: https://docs.llamaindex.ai
- Pinecone learning center

**Communities:**
- LangChain Discord
- r/LocalLLaMA (Reddit)
- Hugging Face forums

---

*Remember: RAG is a powerful technique, but like any tool, its effectiveness depends on thoughtful design and implementation. Start simple, measure performance, and iterate!*

<br>