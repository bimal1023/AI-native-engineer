# 03 · Retrieval & RAG

> If the right chunk isn't in the context, no model and no prompt will save the answer.

## Why This Matters

RAG is the most common LLM architecture in production and the one most often built badly. A weekend RAG demo is 40 lines: chunk, embed, cosine search, stuff into a prompt. A production RAG system is an **information retrieval system** with an LLM at the end — and IR has thirty years of hard-won technique that the vector-database-only approach throws away. The single most valuable skill in this module is decomposition: when an answer is wrong, know whether retrieval failed or generation failed, because they have entirely different fixes.

---

## Key Subtopics

### 3.1 Embeddings and vector search

An embedding model maps text to a dense vector where semantic similarity approximates geometric proximity. Retrieval is then approximate nearest-neighbour search — usually HNSW (graph-based, fast, memory-hungry) or IVF-PQ (quantized, memory-efficient, less accurate). Practical facts that matter more than the math: embeddings are **model-specific** (switching models means a full re-index), similarity is **not relevance** (an antonym-rich passage can sit close to the query), and dimension counts trade recall against memory and latency. Matryoshka-style models let you truncate dimensions for cheap first-pass search.

- [Efficient and robust approximate nearest neighbor search using HNSW](https://arxiv.org/abs/1603.09320)
- [MTEB: Massive Text Embedding Benchmark](https://arxiv.org/abs/2210.07316) · [leaderboard](https://huggingface.co/spaces/mteb/leaderboard) — a prior, not a verdict
- [Dense Passage Retrieval](https://arxiv.org/abs/2004.04906) — where dense retrieval for QA started

### 3.2 Chunking, parsing, and index design

Parsing is upstream of everything and is where most quality is lost. PDFs with multi-column layouts, tables, and headers routinely produce garbage text that no retrieval strategy can rescue — inspect your extracted text before touching embeddings. For chunking, start with structure-aware splits (by heading, section, function, or logical record) rather than fixed character counts, keep chunks in the 200–800 token range with modest overlap, and attach metadata (source, title, section path, date, permissions) to every chunk for filtering and citation. **Contextual retrieval** — prepending an LLM-generated sentence situating each chunk in its document — is one of the highest-return upgrades available.

- [Anthropic: Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) — with reported recall improvements and code
- [Pinecone: Chunking strategies](https://www.pinecone.io/learn/chunking-strategies/)
- [Docling](https://github.com/docling-project/docling), [Unstructured](https://docs.unstructured.io/), [LlamaParse](https://docs.cloud.llamaindex.ai/llamaparse/getting_started) — document parsing that handles tables and layout

### 3.3 Hybrid search and reranking

Dense retrieval is weak exactly where enterprise queries live: exact identifiers, error codes, product SKUs, rare proper nouns, acronyms. Lexical search (BM25) nails those and misses paraphrase. **Run both and fuse** — Reciprocal Rank Fusion is a three-line, tuning-free merge that reliably beats either alone. Then **rerank**: retrieve 50–100 candidates cheaply, pass them through a cross-encoder that scores each against the query jointly, and keep the top 5–10. Reranking is usually the largest single quality jump per hour of engineering in the whole pipeline.

- [Reciprocal Rank Fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) — the original RRF paper
- [Cohere Rerank](https://docs.cohere.com/docs/rerank-overview) · [bge-reranker](https://huggingface.co/BAAI/bge-reranker-v2-m3) — hosted and open cross-encoders
- [ColBERT](https://arxiv.org/abs/2004.12832) — late interaction; the middle ground between bi- and cross-encoders

### 3.4 Query understanding: rewriting, routing, filtering

The user's raw question is rarely the best search query. Techniques that pay for themselves: **rewriting** conversational follow-ups into standalone queries ("what about the second one?" → resolve the referent); **multi-query** expansion, issuing several phrasings and fusing results; **HyDE**, embedding a hypothetical answer instead of the question because answers live near answers in embedding space; **metadata filtering** to constrain by date, tenant, or permission *before* vector search; and **routing** to decide whether a query needs retrieval at all, needs SQL, or needs a different index. Permission filtering is a correctness and security requirement, not an optimization — never filter after retrieval.

- [Precise Zero-Shot Dense Retrieval without Relevance Labels](https://arxiv.org/abs/2212.10496) (HyDE)
- [Query Rewriting for Retrieval-Augmented LLMs](https://arxiv.org/abs/2305.14283)
- [LlamaIndex: query transformations & routers](https://docs.llamaindex.ai/en/stable/optimizing/advanced_retrieval/query_transformations/)

### 3.5 RAG evaluation and failure modes

Evaluate the two stages separately or you will optimize blind.

- **Retrieval**: recall@k, precision@k, MRR, nDCG against a labeled query→relevant-chunk set. Build this set by hand from 50–100 real queries; it's a day of work and it makes every subsequent decision measurable.
- **Generation**: faithfulness/groundedness (is every claim supported by retrieved context?), answer relevance, and citation correctness.

Know the canonical failure modes: the chunk was never retrieved; it was retrieved but ranked below the cutoff; it was in context but ignored; the model answered from parametric memory instead of context; the index is stale; the answer is right but uncited. Each has a different fix.

- [Retrieval-Augmented Generation for Knowledge-Intensive NLP](https://arxiv.org/abs/2005.11401) — the original RAG paper
- [Seven Failure Points When Engineering a RAG System](https://arxiv.org/abs/2401.05856)
- [Ragas](https://docs.ragas.io/) — faithfulness, context precision/recall metrics out of the box

---

## Tools & Frameworks Used in Industry

| Purpose | Tools |
|---|---|
| Vector storage | [pgvector](https://github.com/pgvector/pgvector), [Qdrant](https://qdrant.tech/), [Weaviate](https://weaviate.io/), [Milvus](https://milvus.io/), [Pinecone](https://www.pinecone.io/), [Chroma](https://www.trychroma.com/), [LanceDB](https://lancedb.com/), [turbopuffer](https://turbopuffer.com/) |
| Lexical / hybrid engines | [Elasticsearch](https://www.elastic.co/), [OpenSearch](https://opensearch.org/), [Vespa](https://vespa.ai/), Postgres full-text search |
| Embedding models | OpenAI `text-embedding-3`, [Cohere Embed](https://docs.cohere.com/docs/cohere-embed), [Voyage AI](https://docs.voyageai.com/), [BGE](https://huggingface.co/BAAI), [E5](https://huggingface.co/intfloat), [Nomic Embed](https://www.nomic.ai/blog/posts/nomic-embed-text-v1) |
| Reranking | Cohere Rerank, `bge-reranker-v2-m3`, Voyage rerank, [RankGPT](https://github.com/sunnweiwei/RankGPT) |
| Orchestration | [LlamaIndex](https://docs.llamaindex.ai/), [LangChain](https://python.langchain.com/), [Haystack](https://haystack.deepset.ai/) |
| Ingestion & parsing | Docling, Unstructured, LlamaParse, [Firecrawl](https://www.firecrawl.dev/), [Reducto](https://reducto.ai/) |
| RAG evaluation | [Ragas](https://docs.ragas.io/), [TREC-style](https://trec.nist.gov/) labeled sets, [BEIR](https://github.com/beir-cellar/beir) |

> **Start with pgvector** if you already run Postgres. Transactional consistency with your source data, metadata filters in SQL, one system to operate. Move to a dedicated vector DB when scale or feature needs justify it — not before.

---

## Hands-On Project: Cited Answers Over a Real Corpus

**Build a question-answering system over a corpus you actually care about, and improve it by measurement, not intuition.**

Requirements:
1. Ingest a real corpus (500+ documents): your company's docs, a library's source + docs, arXiv papers in one area, or public filings.
2. Hand-label 50 evaluation queries with the chunk(s) that should be retrieved. This is the core asset — do not skip it.
3. Ship a **baseline**: fixed-size chunks, dense-only retrieval, top-5, stuffed into a prompt. Record recall@5 and faithfulness.
4. Then add one change at a time, re-measuring after each: structure-aware chunking → metadata filters → BM25 + RRF hybrid → cross-encoder reranking → contextual retrieval → query rewriting.
5. Require inline citations with source and section, and verify citations resolve to real chunks.
6. Publish a table: variant × recall@5 × faithfulness × p95 latency × cost per query.

**Done when:** you have a chart showing recall improving across variants, and you can state which change earned its complexity and which didn't.

**Stretch:** add a router that sends metadata-shaped questions ("how many docs mention X since March?") to SQL instead of vector search.

---

## Common Pitfalls

- **Evaluating end-to-end only.** If you can't separate retrieval failure from generation failure, every fix is a guess.
- **Skipping the labeled retrieval set.** One day of labeling buys months of measurable iteration.
- **Fixed-size chunking on structured documents.** Splitting mid-table or mid-function destroys the meaning you're trying to retrieve.
- **Never looking at your parsed text.** Print 20 random chunks. The PDF extractor is worse than you think.
- **Dense-only retrieval.** Guaranteed failure on error codes, IDs, SKUs, and rare proper nouns. Add BM25.
- **No reranker.** Usually the cheapest large quality win still on the table.
- **Filtering permissions after retrieval.** That's a data leak. Filter in the query.
- **Swapping the embedding model without re-indexing.** Vectors from different models are not comparable; results become subtly, silently wrong.
- **No freshness strategy.** Deleted and updated source documents must propagate to the index, or you serve confident stale answers.
- **Answers without citations.** Users can't verify, and you can't debug.
- **Reaching for RAG when the corpus fits in context** — or when the real problem is a missing database query.

---

## Progress

- [ ] 3.1 Embeddings and vector search
- [ ] 3.2 Chunking, parsing, and index design
- [ ] 3.3 Hybrid search and reranking
- [ ] 3.4 Query understanding: rewriting, routing, filtering
- [ ] 3.5 RAG evaluation and failure modes
- [ ] **Project:** cited answers over a real corpus

---

[← 02 · Prompting & Context Engineering](../02-prompting-and-context-engineering/README.md) · [Back to root](../README.md) · [Next: 04 · Agents & Tool Use →](../04-agents-and-tool-use/README.md)
