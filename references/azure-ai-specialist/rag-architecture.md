# RAG Architecture Patterns

## Basic RAG Pattern

The fundamental Retrieval-Augmented Generation flow:

```
User Query
  → Generate embedding (text-embedding-3-small/large)
  → Vector search against index (Azure AI Search)
  → Retrieve top-K relevant chunks
  → Inject chunks as context into LLM prompt
  → LLM generates grounded response
  → Return response with citations
```

**Why RAG**: LLMs have training data cutoffs and lack your proprietary knowledge. RAG grounds responses in your actual data, reducing hallucinations and enabling domain-specific answers.

## Azure AI Search as Vector Store

Azure AI Search is the recommended vector store for enterprise RAG on Azure.

### Hybrid Search (Recommended Default)
Combines three retrieval methods for maximum recall:
1. **Keyword search (BM25)**: Traditional full-text matching — catches exact terms, acronyms, identifiers
2. **Vector search**: Semantic similarity via embeddings — catches meaning even with different wording
3. **Semantic ranking**: Neural re-ranker applied on top of combined results — reorders by semantic relevance

Always use hybrid search. Pure vector search misses exact-match queries; pure keyword search misses semantic meaning.

### Index Configuration
- **Vector fields**: Store embeddings alongside text content
- **Filterable fields**: Add metadata fields (source, date, category) for filtered search
- **Searchable fields**: Enable full-text search on text content fields
- **Vector algorithms**: HNSW (default, good recall) or exhaustive KNN (exact, slower)
- **Dimensions**: Match your embedding model (1536 for text-embedding-3-small, 3072 for large)

### Integrated Vectorization Pipeline
Azure AI Search can handle the full ingestion pipeline:
- **Data sources**: Blob Storage, Cosmos DB, SQL Database, SharePoint, Azure Data Lake
- **Skillsets**: Built-in skills for OCR, image analysis, document cracking, language detection, PII redaction
- **Indexers**: Scheduled or on-demand execution, change tracking for incremental updates
- **Embedding**: Integrated embedding skill calls Azure OpenAI — no external pipeline needed

## Chunking Strategies

Chunking determines how documents are split before embedding. This is the single most impactful RAG design decision.

### Fixed-Size Chunking
- Split text into chunks of N tokens (e.g., 512 or 1024) with overlap
- **Overlap**: 10-15% of chunk size to preserve context at boundaries
- **Pros**: Simple, predictable, works for most content
- **Cons**: May split mid-sentence or mid-concept
- **When to use**: Default starting point for most content

### Semantic Chunking
- Split at natural semantic boundaries (paragraphs, topic shifts)
- Uses embedding similarity to detect topic boundaries
- **Pros**: More meaningful chunks, better retrieval relevance
- **Cons**: Variable chunk sizes, more complex to implement
- **When to use**: Long-form narrative content, articles, documentation

### Document-Structure-Aware Chunking
- Respect document structure (headings, sections, tables, lists)
- Keep tables and structured data intact within single chunks
- Preserve heading hierarchy as chunk metadata
- **Pros**: Maintains document context, tables stay intact
- **Cons**: Requires document parsing, format-specific logic
- **When to use**: Technical documents, reports, structured content with tables

### Chunking Best Practices
- **Chunk size**: Start with 512-1024 tokens. Smaller = more precise retrieval, larger = more context per chunk.
- **Overlap**: 10-15% of chunk size prevents losing context at boundaries
- **Metadata**: Always store source document, section heading, page number alongside chunk
- **Test empirically**: Optimal chunk size depends on your content and queries — benchmark different sizes

## Embedding Pipeline

```
Documents (PDF, DOCX, HTML, etc.)
  → Document cracking (extract text, tables, images)
  → Chunking (fixed-size, semantic, or structure-aware)
  → Metadata extraction (source, date, section, author)
  → Embedding generation (text-embedding-3-small or large)
  → Index in Azure AI Search (vector + text + metadata)
```

### Embedding Model Selection
- **text-embedding-3-small**: Default choice. ~$0.02/1M tokens. 1536 dimensions. Good quality for most applications.
- **text-embedding-3-large**: Use when small model quality is demonstrably insufficient. ~$0.13/1M tokens. 3072 dimensions.
- **Dimension reduction**: Both models support reducing dimensions (e.g., 256 or 512) for storage/cost savings with modest quality trade-off.

### Volume Planning
Large corpus embedding can be expensive. Example: 1M documents × 500 tokens average = 500M tokens = ~$10 with small model. Plan for initial bulk embedding plus incremental updates.

## Advanced RAG Patterns

### Query Expansion
- Rewrite the user query into multiple search queries for broader recall
- Use the LLM to generate alternate phrasings of the question
- Merge results from all queries before sending to the LLM

### Hypothetical Document Embedding (HyDE)
- Ask the LLM to generate a hypothetical answer to the question
- Embed the hypothetical answer instead of the raw question
- Search with the hypothetical answer embedding (often retrieves better matches)
- Useful when user queries are short or vague

### Re-Ranking
- Retrieve a larger initial set (top-50) with hybrid search
- Re-rank with semantic ranker or a cross-encoder model to get top-5
- Higher quality results at the cost of additional compute

### Multi-Index Retrieval
- Search across multiple indexes (e.g., documentation + knowledge base + FAQ)
- Merge and re-rank results across indexes
- Useful when knowledge is spread across different data sources

## Evaluation

Evaluate RAG quality using these metrics (available in Azure AI Foundry):

| Metric | What It Measures | Why It Matters |
|--------|-----------------|----------------|
| Groundedness | Is the response supported by the retrieved context? | Prevents hallucination |
| Relevance | Does the response answer the user's question? | Ensures usefulness |
| Coherence | Is the response logically consistent and well-structured? | Readability |
| Fluency | Is the response grammatically correct and natural? | User experience |
| Retrieval quality | Are the retrieved chunks relevant to the query? | Upstream quality check |

### Evaluation Best Practices
- Build a test dataset of question-answer-context triples from your domain
- Run evaluations on every change to chunking, prompts, or model versions
- Use Azure AI Foundry evaluation tools for automated scoring
- Supplement with human evaluation for subjective quality

## Architecture Overview

```
[User App] → [Azure OpenAI] ← generates response using context
                  ↑
            [Prompt with retrieved chunks]
                  ↑
          [Azure AI Search] ← hybrid search (keyword + vector + semantic)
                  ↑
          [Embedding of user query via Azure OpenAI]
```

Key integration points:
- App calls Azure OpenAI for both embedding generation and chat completion
- Azure AI Search stores vectors, text, and metadata
- System prompt instructs the LLM to answer only from provided context and cite sources
- Content filtering applies to both the query and the generated response
