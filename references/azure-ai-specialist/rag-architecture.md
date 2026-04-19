# RAG Architecture Patterns

## Vector Store Selection

| Store | Best For | Vector Search | Operational Data | Cost Model |
|-------|----------|---------------|------------------|------------|
| Azure AI Search | Enterprise RAG, hybrid search | DiskANN, HNSW | Separate indexer pipeline | Per-unit pricing, tiered |
| Cosmos DB NoSQL | Real-time RAG + operational data | IVF, HNSW, DiskANN | Same database | RU-based |
| Cosmos DB MongoDB vCore | MongoDB-compatible apps | HNSW | Same database | vCore-based |
| Azure SQL | Existing SQL Server workloads | HNSW (preview) | Same database | DTU/vCore |
| External (Qdrant, Pinecone, etc.) | Multi-cloud, specialized needs | Varies | Separate | Varies |

## RAG Decision Tree

```
What kind of RAG do you need?
├─ Enterprise search over documents?
│  ├─ Need agentic retrieval (multi-query, parallel)? → AI Search agentic retrieval
│  ├─ Need classic single-query RAG? → AI Search classic
│  └─ Need SharePoint/Teams content? → AI Search + SharePoint connector
├─ RAG + operational data in same database?
│  ├─ Global distribution needed? → Cosmos DB NoSQL with vector search
│  ├─ MongoDB compatibility? → Cosmos DB MongoDB vCore
│  └─ SQL Server ecosystem? → Azure SQL with vector search
├─ Agent memory + RAG combined?
│  └─ Cosmos DB (vector search + change feed for real-time updates)
└─ Multi-modal RAG (images, PDFs)?
   └─ AI Search with skillsets (OCR, image analysis, document extraction)
```

## AI Search Relevance Optimization

1. **Hybrid queries** (keyword + vector) for maximum recall — always prefer over pure vector
2. **Semantic ranking** for re-ranking results by semantic relevance
3. **Chunking strategy matters**: 512-1024 tokens per chunk, with overlap (10-15%)
4. **Scoring profiles** to boost specific fields (title > body, recent > old)
5. **Vector weighting** to balance keyword vs vector relevance
6. **Minimum thresholds** to filter low-confidence results before sending to LLM
