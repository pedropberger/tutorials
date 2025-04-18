
## Smaller Indices, Smaller Problems: Lessons from My Initial Journey with Elasticsearch for RAG

Retrieval-Augmented Generation (RAG) systems represent a powerful fusion of large language models (LLMs) and information retrieval. The core idea is simple yet effective: instead of relying solely on the LLM's pre-trained knowledge, we first retrieve relevant documents from a corpus and then provide this context to the LLM to generate a more accurate and grounded response. My journey into building a RAG system led me straight to Elasticsearch, renowned for its text search capabilities and increasingly, its vector search prowess. However, like many initial forays into a powerful tool, my first approach wasn't necessarily the most optimal. Here I want to share some of what I've learned and how I've evolved. This text is a collection of my notes as I struggled to deploy an optimized app ([this one](https://github.com/mpes-uis/rag_gampes)).

![Im not soo good building AI images...](https://github.com/user-attachments/assets/bf11a426-7488-4de8-b4b0-5abede418f42)


**The Initial Approach: One Index to Rule Them All**

Coming from a relational database background, my initial dataset was structured with clear tables and relationships. I had documents, their constituent pages, identifiers, and critically for RAG, embedding vectors representing the semantic meaning of the text chunks.

Leveraging Elasticsearch's famed schema flexibility seemed like a godsend. Why bother defining rigid structures upfront? My initial strategy was straightforward: consolidate *everything* related to a retrievable unit into a single Elasticsearch index. A typical document in this index looked something like this:

```json
{
  "document_id": "doc_123",
  "document_title": "Research Paper on AI Ethics",
  "full_text": "The complete text of the research paper...",
  "page_number": 5,
  "page_content": "Content specific to page 5...",
  "source_db_id": 4567,
  "embedding_model": "text-embedding-ada-002",
  "embedding_vector": [0.123, -0.456, ..., 0.789] // Dense vector
}
```

This approach had immediate appeal:

1.  **Simplicity:** One index definition, one ingestion pipeline. Easy to get started.
2.  **Flexibility:** No need to pre-define every field rigorously. Elasticsearch would handle new fields gracefully.
3.  **Co-location:** All information potentially needed for retrieval or display was in one place. A single query could fetch everything.

**The Cracks Begin to Show**

While functional, this monolithic approach quickly revealed its limitations, especially as the dataset grew and requirements evolved:

1.  **Performance Bottlenecks:**
    *   **Indexing:** Ingesting large documents containing potentially lengthy `full_text` fields alongside dense vectors and other metadata made indexing slower than necessary, especially when only page-level chunks were truly needed for vector search.
    *   **Querying:** Performing a k-Nearest Neighbor (k-NN) search on the `embedding_vector` field meant Elasticsearch still had to deal with potentially large documents, even if only the vector and a small identifier were strictly needed for the similarity search phase. Similarly, text searches might inadvertently load large vector fields into memory, even if not directly queried.
2.  **Inefficiency in Updates:** If I only needed to update the `page_content` for a specific page, I still had to re-index the entire large document associated with that page entry. This was wasteful.
3.  **Embedding Model Experimentation Nightmare:** This was the biggest pain point. RAG performance is highly sensitive to the quality of embeddings. I wanted to test different embedding models (e.g., OpenAI's Ada, Cohere's Embed, a custom Sentence-BERT model). With the single-index approach, this meant either:
    *   Creating entirely new, massive indices duplicating all the text content, just with different embedding vectors.
    *   Adding *multiple* embedding vector fields to the *same* document (e.g., `embedding_vector_ada`, `embedding_vector_cohere`), making documents even larger and queries more complex (requiring conditional logic on which vector field to use). Both options were highly inefficient in terms of storage and management.

**Evolution: Embracing Segmentation with Multiple Indices**

The challenges clearly pointed towards a need for specialization. I decided to break down the monolithic index into several smaller, purpose-built indices:

1.  **`documents_index`:**
    *   Fields: `document_id`, `document_title`, `full_text`, `source_db_id`.
    *   Purpose: Store the core document metadata and the complete text, optimized for full-text search or retrieval of the entire document content *after* initial identification.

2.  **`pages_index`:**
    *   Fields: `page_id` (unique identifier for the chunk), `document_id` (foreign key), `page_number`, `page_content`.
    *   Purpose: Store the chunked text content used for generating embeddings and potentially for displaying snippets. Optimized for text search on smaller chunks.

3.  **`embeddings_index`:**
    *   Fields: `embedding_id` (can be same as `page_id` or a unique ID), `reference_id` (linking back to `page_id` or `document_id`), `embedding_vector` (dense vector field, configured for k-NN search), `embedding_model_name`.
    *   Purpose: Exclusively store embedding vectors and necessary identifiers. Highly optimized for vector search.

**Observed Benefits of Segmentation**

This segmented approach yielded significant improvements:

1.  **Improved Query Performance:**
    *   Vector search (k-NN) queries hit the `embeddings_index`, which contained only lean documents (ID, vector, model name). This was noticeably faster as less data needed to be loaded and processed per document during the search.
    *   Text searches on page content hit the `pages_index`, which also contained smaller, more focused documents than the original monolithic index.
    *   Retrieving full context required a two-step process (k-NN search on `embeddings_index` -> get `reference_id`s -> fetch corresponding documents from `pages_index` or `documents_index`), but the overall latency was often lower due to the speedup in the initial, most intensive k-NN step. Client-side joins or simple follow-up queries based on IDs are generally fast.
2.  **Faster Indexing and Updates:** Updating page content only required re-indexing a small document in `pages_index`. Generating and indexing new embeddings only affected the `embeddings_index`.
3.  **Simplified Embedding Experimentation:** This was the game-changer. To test a new embedding model (e.g., `bge-large-en`), I simply:
    *   Generated embeddings for the content in `pages_index` using the new model.
    *   Indexed these new embeddings into the *same* `embeddings_index`, just setting the `embedding_model_name` field appropriately (e.g., `bge-large-en`).
    *   Alternatively, and perhaps cleaner, I could create a *new, separate* embedding index (e.g., `embeddings_bge_large_index`) containing only vectors from that model.
    *   My application logic could then choose which embedding index (or which subset of documents within the single embedding index, filtered by `embedding_model_name`) to query for the k-NN search, without ever touching the source text indices (`documents_index`, `pages_index`). No data duplication, minimal overhead.

**The 'Divide and Conquer' Principle in Elasticsearch**

This journey perfectly illustrates the 'divide and conquer' principle applied to Elasticsearch index design. By breaking down a complex, multi-purpose index into smaller, specialized indices, we gained:

*   **Organization:** Clear separation of concerns. Each index has a well-defined purpose and structure.
*   **Optimization:** Each index can be tuned specifically for its primary workload. The `embeddings_index` can have mappings optimized for `dense_vector` fields and k-NN search, while the `pages_index` can be tuned for text search relevance (`BM25`) and aggregations. Shard counts and hardware allocation can potentially be tailored per index based on size and query load.
*   **Modularity & Maintainability:** Easier to manage, update, and experiment with different parts of the system independently.

**Caution: The Pitfall of Over-Fragmentation**

However, the 'divide and conquer' principle has its limits. While segmentation was beneficial in my case, excessive fragmentation – breaking data down into *too many* small indices – can be detrimental in Elasticsearch. This is analogous to excessive normalization in relational databases, a practice generally discouraged in Elasticsearch for several reasons:

1.  **Query Complexity:** Elasticsearch performs best when querying data within a single index or a small number of indices. Queries spanning numerous indices (using multi-index searches or requiring extensive client-side joins based on IDs retrieved from multiple preliminary queries) increase complexity and can negate performance gains, especially if intermediate result sets are large. Operations like `_msearch` help, but managing queries across dozens of indices becomes unwieldy.
2.  **Relevance Scoring Challenges:** Combining relevance scores from text searches across many different indices can be complex and may not produce intuitive results compared to searching within a single, more comprehensive index where term statistics (like IDF) are calculated across the relevant corpus.
3.  **Management Overhead:** Every index, and more importantly, every *shard*, consumes resources (memory for metadata, file handles, CPU for cluster state management). Having hundreds or thousands of tiny indices/shards ("shard overallocation") can strain the cluster master node and lead to inefficiency and instability. It also complicates monitoring, backup, and maintenance routines.

The key is finding the right balance – segmenting where there are clear benefits in terms of data type, update patterns, or query optimization, but avoiding fragmentation purely for the sake of separation if the data is frequently accessed together.

**When a Single Index (Still) Shines**

Despite my positive experience with segmentation, the initial monolithic approach isn't inherently wrong. It remains a valid and efficient strategy in specific scenarios:

1.  **Simple Datasets & Queries:** For smaller datasets with relatively few fields and straightforward query patterns (e.g., searching across text and filtering by a few metadata fields simultaneously), a single index is often simpler and perfectly adequate.
2.  **Tightly Coupled Data Access:** If the vast majority of queries *require* fetching and filtering/searching across most fields within the document simultaneously (e.g., displaying a product listing with title, description, specs, price, *and* using an embedding for similarity), keeping everything in one document can be more efficient than performing multiple lookups across indices. Elasticsearch is very fast at filtering and retrieving fields from the *same* document.
3.  **Rapid Prototyping:** When speed of development is paramount and performance/scalability concerns are secondary (e.g., building an initial proof-of-concept), the simplicity of a single index is advantageous.
4.  **Read-Heavy Workloads with Infrequent Updates:** If the data rarely changes and the primary workload involves complex searches across multiple facets of the same logical entity, the denormalized single-index approach minimizes query-time complexity.

**Conclusion**

My initial journey with Elasticsearch for RAG taught me a valuable lesson: while the platform's flexibility allows for a simple "dump everything here" approach, thoughtful index design pays significant dividends. Segmenting data into smaller, specialized indices based on data type and access patterns led to tangible improvements in query performance, update efficiency, and crucially, the ability to experiment with different embedding models without massive data duplication. Applying the 'divide and conquer' principle brought clarity and optimization. However, it's essential to temper this with caution against over-fragmentation, recognizing that sometimes, particularly for simpler or tightly coupled use cases, a single comprehensive index remains the most practical approach. Like many aspects of system design, the optimal Elasticsearch indexing strategy depends heavily on the specific requirements, data characteristics, and query patterns of the application.

---
