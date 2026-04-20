# Azure AI Search Indexing & Hybrid Search Configuration

Patterns for configuring Azure AI Search as a vector store for RAG and enterprise search.

## Index Design

### Field Configuration for RAG
A production RAG index needs these field types:
```json
{
  "fields": [
    { "name": "id", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "content", "type": "Edm.String", "searchable": true, "analyzer": "en.microsoft" },
    { "name": "content_vector", "type": "Collection(Edm.Single)", "searchable": true,
      "dimensions": 1536, "vectorSearchProfile": "default-profile" },
    { "name": "title", "type": "Edm.String", "searchable": true, "filterable": true },
    { "name": "source", "type": "Edm.String", "filterable": true, "facetable": true },
    { "name": "chunk_id", "type": "Edm.Int32", "filterable": true, "sortable": true },
    { "name": "last_updated", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true },
    { "name": "category", "type": "Edm.String", "filterable": true, "facetable": true }
  ]
}
```
- Mark metadata fields as `filterable` for scoped retrieval (e.g., search only within a specific source)
- Use `facetable` for fields you need to aggregate on (e.g., document categories)
- Match `dimensions` to your embedding model: 1536 for text-embedding-3-small, 3072 for text-embedding-3-large

### Vector Search Configuration
```json
{
  "vectorSearch": {
    "algorithms": [
      { "name": "hnsw-default", "kind": "hnsw",
        "hnswParameters": { "m": 4, "efConstruction": 400, "efSearch": 500, "metric": "cosine" } }
    ],
    "profiles": [
      { "name": "default-profile", "algorithm": "hnsw-default" }
    ]
  }
}
```
- **HNSW** (default): Fast approximate nearest neighbor — good recall with configurable speed/accuracy tradeoff
- **Exhaustive KNN**: Exact nearest neighbor — slower but 100% recall. Use for small indexes or as a quality baseline
- **`efSearch`**: Higher values = better recall, slower queries. Start at 500, tune based on quality/latency requirements
- **`metric`**: Use `cosine` for OpenAI embedding models

## Hybrid Search Configuration

### Why Hybrid Search Is the Default
Hybrid search combines three retrieval methods:
1. **BM25 keyword search**: Catches exact terms, acronyms, IDs, product codes
2. **Vector similarity search**: Catches semantic meaning even with different wording
3. **Semantic ranker** (optional): Neural re-ranker that reorders combined results by relevance

**Always start with hybrid search**. Pure vector search fails on exact-match queries. Pure keyword search fails when users describe concepts differently than the documents.

### Query Configuration
```python
results = search_client.search(
    search_text=user_query,                    # BM25 keyword search
    vector_queries=[VectorizedQuery(
        vector=query_embedding,                # Vector search
        k_nearest_neighbors=50,
        fields="content_vector"
    )],
    query_type=QueryType.SEMANTIC,             # Enable semantic ranker
    semantic_configuration_name="default",
    top=5,                                     # Final results to return
    filter="category eq 'technical-docs'"      # Optional metadata filter
)
```

### Semantic Ranker Considerations
- Billed per query, separate from index storage costs — can be significant at scale
- Processes the top 50 results from initial retrieval — setting `k_nearest_neighbors > 50` doesn't improve ranking
- Use selectively: enable for user-facing queries, skip for internal/batch retrieval operations
- Requires a semantic configuration pointing to the content field and optionally title/keyword fields

## Integrated Vectorization Pipeline

Azure AI Search can handle the entire document ingestion pipeline:

### Indexer + Skillset Pattern
```
Data Source (Blob Storage / Cosmos DB / SQL)
  → Indexer (scheduled or on-demand)
    → Skillset (chunking + embedding)
      → Index (text + vectors + metadata)
```

### Key Skills
- **Text Split skill**: Chunks documents by token count with configurable overlap
- **Azure OpenAI Embedding skill**: Generates vectors using your Azure OpenAI deployment
- **Custom Web API skill**: Call custom logic for domain-specific processing (PII scrubbing, entity extraction)

### Chunking Configuration
```json
{
  "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
  "textSplitMode": "pages",
  "maximumPageLength": 2000,
  "pageOverlapLength": 500,
  "unit": "characters"
}
```
- Start with ~2000 characters (~500 tokens) per chunk with 500-character overlap
- Adjust based on your content type: technical docs may need larger chunks for context; FAQ-style content works with smaller chunks
- Test retrieval quality with representative queries after any chunking changes

## Incremental Indexing

### Change Detection Strategies
- **Blob Storage**: Use `LastModified` timestamp for automatic change detection
- **Cosmos DB**: Use change feed for near-real-time updates
- **SQL**: Use high-watermark column (e.g., `last_modified` datetime) or SQL change tracking
- **Reset**: If you change the skillset (chunking strategy, embedding model), you must reset and re-run the indexer

### Index Aliasing for Zero-Downtime Updates
When re-indexing (e.g., embedding model change), create a new index and swap the alias:
1. Create new index with updated schema/embeddings
2. Run full indexer against new index
3. Swap alias to point to new index
4. Delete old index after validation
