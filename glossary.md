# Glossary

> Every term this repo uses, defined once. The most useful section is [Commonly Confused](#commonly-confused) — those pairs cause more production bugs than anything else here. [Back to root](README.md)

---

## Model & Architecture

**Attention** — the mechanism letting each token weigh every previous token. Quadratic in sequence length, which is why long context is expensive. [§1.1](01-llm-fundamentals/README.md#11-transformers-and-the-attention-mechanism)

**Decoder-only** — the architecture behind every modern chat LLM. Generates left to right, one token at a time.

**Embedding** — a vector representation of text. Model-specific: vectors from different models are not comparable, so switching embedding models means re-indexing everything.

**GQA (Grouped-Query Attention)** — groups of attention heads share keys/values, shrinking the KV cache. Why two models with equal parameter counts can have very different memory footprints.

**Logits** — raw pre-softmax scores over the vocabulary, one per token.

**Logprobs** — log-probabilities of chosen tokens. An underused, nearly free confidence signal for routing and abstention.

**MoE (Mixture of Experts)** — many "expert" blocks with a router activating only a few per token. **Total parameters** drive memory; **active parameters** drive cost and speed.

**RoPE (Rotary Position Embedding)** — encodes token position by rotating query/key vectors. What makes context-window extension possible, and why extended context underperforms native context.

**Temperature** — flattens (>1) or sharpens (<1) the output distribution. Tune this *or* `top_p`, never both.

**Top-p (nucleus sampling)** — truncates sampling to the smallest set of tokens summing to probability *p*.

---

## Tokens & Context

**Token** — the unit a model actually sees. ~4 characters per token for English prose; far fewer for code, JSON, and non-Latin scripts.

**BPE (Byte-Pair Encoding)** — the tokenization family used by essentially all current models.

**Context window** — total tokens the model can process, **input plus output combined**.

**Max output tokens** — the separate, usually much smaller cap on generated tokens. Truncated JSON is almost always this, not the context window.

**Effective context** — the range within which recall actually holds up, typically well below the advertised limit.

**Lost in the middle** — measured degradation in recall for content buried mid-context. Put critical instructions near the end.

**KV cache** — stored attention keys/values for already-processed tokens, making each new token O(1). Grows with context length and concurrency; often the real memory constraint.

**Prompt caching** — provider-side reuse of a repeated prompt prefix. Up to ~90% off input cost, but only if the prefix is byte-identical. A timestamp at the top of your prompt zeroes it out.

---

## Prompting & Context Engineering

**Context engineering** — deciding what goes into the window at runtime, and what to leave out. The discipline that replaced "prompt engineering."

**Few-shot** — including examples in the prompt. Teaches format and edge cases far more efficiently than instructions. 3–5 diverse examples usually beats a page of rules.

**Chain of thought (CoT)** — prompting the model to reason step by step before answering. Redundant or harmful with reasoning models, which already do this internally.

**Structured output** — constraining generation to a schema, via tool calling or constrained decoding. Always validate on receipt and have a retry path.

**Constrained decoding** — masking the sampler so only grammar-valid tokens can be emitted. The strongest structured-output guarantee.

**Compaction** — summarizing or dropping older context as a conversation or agent loop grows.

**Prompt injection** — instructions hidden in untrusted content that the model follows. No prompt fixes this; mitigation is architectural. [soft-skills §3](soft-skills.md#3-prompt-injection-and-security-awareness)

**Lethal trifecta** — private data access + untrusted content + an outbound channel. Break any one leg.

---

## Retrieval & RAG

**RAG** — retrieval-augmented generation: fetch relevant context, then generate from it.

**Chunking** — splitting documents for indexing. Structure-aware beats fixed-size; 200–800 tokens is the usual range.

**Contextual retrieval** — prepending an LLM-generated sentence situating each chunk in its document. One of the highest-return upgrades available.

**BM25** — lexical ranking from the 1990s. Still beats dense retrieval on exact identifiers, error codes, SKUs, and rare proper nouns.

**Hybrid search** — running lexical and dense retrieval together and fusing results.

**RRF (Reciprocal Rank Fusion)** — a tuning-free way to merge ranked lists. Three lines, reliably beats either input alone.

**Reranking** — a cross-encoder scoring query and document jointly. Retrieve 50–100 cheaply, rerank to the top 5–10. Usually the single largest quality jump per hour of work.

**Cross-encoder / bi-encoder** — scores a pair jointly (accurate, slow) vs embeds each side independently (fast, searchable at scale).

**HNSW** — the graph-based approximate nearest-neighbour index most vector databases use.

**HyDE** — embedding a *hypothetical answer* instead of the question, because answers live near answers in embedding space.

**recall@k / precision@k / nDCG / MRR** — retrieval metrics. Recall@k is the one to start with: did the right chunk appear in the top k?

**Faithfulness / groundedness** — whether every claim in the answer is supported by retrieved context. A generation metric, measured separately from retrieval.

---

## Agents

**Agent** — a loop where the model chooses the next action. Use only when the number of steps can't be known in advance.

**Workflow** — LLM calls on predefined paths. Predictable, cheap, testable. Usually the right answer.

**Tool calling** — the model returns a structured call; **your code** executes it and returns the result. That boundary is where all your control lives.

**MCP (Model Context Protocol)** — an open standard for connecting models to tools and data. Solves *how* things connect, not *what* to connect.

**Trajectory** — the full sequence of steps an agent took. Evaluate this, not just the final answer.

**Idempotency key** — makes a retried mutating call safe. Without it, retries send the email twice.

**Step budget** — a hard cap on iterations. Every agent must terminate.

**Sandbox** — isolated execution for model-generated code. Assume that code is untrusted.

---

## Evaluation & Observability

**Eval** — a labeled dataset plus scoring, used to detect regressions. The skill that most separates AI engineers from people who call AI APIs.

**Error analysis** — reading 50–100 real outputs and writing down what's wrong. Do this *before* choosing metrics.

**Golden set** — a frozen labeled set used for regression testing in CI.

**LLM-as-judge** — using a model to grade output. Useless until calibrated against human labels (aim for ≥80% agreement).

**Calibration** — how well stated confidence matches actual correctness.

**Trace / span** — one request, and one operation within it. Every LLM span should carry model, prompt version, tokens, cost, and latency.

**OTel GenAI semantic conventions** — standardized attribute names for LLM telemetry, so you're not locked to one vendor.

**Guardrails** — input/output filters running in production. Online evals with teeth.

**Regression gate** — CI failing the build when quality drops below a committed baseline.

---

## Serving & Infrastructure

**TTFT (time to first token)** — dominated by input length and prefill. What the user actually feels when streaming.

**TPOT / ITL** — per-output-token time. Memory-bandwidth bound.

**Prefill vs decode** — reading the input (compute-bound) vs generating output (bandwidth-bound). Optimized differently.

**Continuous batching** — merging incoming requests into a running batch. The largest single throughput win in serving.

**PagedAttention** — non-contiguous KV cache storage that eliminates memory fragmentation. The core vLLM idea.

**Speculative decoding** — a small draft model proposes tokens a large model verifies in parallel. Cuts latency without changing output.

**Quantization** — reducing weight precision (FP8, INT4) to fit more model in less memory. 8-bit is near-lossless; below 4-bit degrades noticeably, and long-context reasoning degrades first.

**AI gateway** — a service between your app and model providers holding retries, fallbacks, caching, rate limits, and logging. App code should never call a provider SDK directly.

**Model cascade / tiering** — routing easy requests to a small model and escalating only when needed. Often 60–80% cost reduction.

**LoRA / QLoRA** — training a small number of adapter parameters instead of the full model. Makes fine-tuning affordable and lets you serve many adapters on one base model.

**Batch API** — asynchronous bulk processing at roughly half price, when latency doesn't matter.

---

## Commonly Confused

The pairs that actually cause bugs. If you only read one section, read this one.

| These get mixed up | The distinction |
|---|---|
| **Context window** vs **max output tokens** | Total budget vs the cap on generation alone. Truncated output is almost always the second. |
| **Total** vs **active parameters** | Memory footprint vs cost and speed. With MoE these differ by 10×. |
| **Latency** vs **throughput** | Time per request vs requests per second. Batching improves one and worsens the other. |
| **TTFT** vs **total latency** | What the user feels while streaming vs when the job finishes. Quote both. |
| **Temperature** vs **top-p** | Two ways to shape the same distribution. Tune one; changing both makes results uninterpretable. |
| **Agent** vs **workflow** | Model decides the path vs the path is predefined. Most "agents" should be workflows. |
| **Fine-tuning** vs **RAG** | Teaching form and style vs supplying facts. Fine-tuning does not reliably teach facts. |
| **Embedding model** vs **vector database** | The thing that makes vectors vs the thing that stores and searches them. |
| **Semantic similarity** vs **relevance** | Close in vector space vs actually answers the question. Not the same, which is why reranking exists. |
| **Retrieval failure** vs **generation failure** | The chunk was never found vs it was found and ignored. Different fixes; measure separately. |
| **Authentication** vs **authorization** | Who you are vs what you may do. |
| **Prompt injection** vs **jailbreak** | Third-party instructions in your data vs the user trying to bypass your rules. |
| **Hallucination** vs **wrong retrieval** | The model invented it vs you fed it the wrong document. Check the trace before blaming the model. |
| **Reasoning tokens** vs **output tokens** | Hidden thinking is still billed as output, and can be 10× the visible answer. |
| **`O(1)` hash lookup** — average vs worst | Say "average." Worst case is `O(n)`. |

---

## Acronym Index

AI RMF · BM25 · BPE · CoT · CAP · DPO · GQA · HNSW · HyDE · ITL · KV · LLM · LoRA · MCP · MoE · MQA · MRR · nDCG · OTel · PEFT · QLoRA · RAG · RLHF · RoPE · RRF · SFT · SLO · TPOT · TTFT · vLLM

---

*Missing a term? That's a good first contribution — see [Contributions Welcome](README.md#contributions-welcome).*

[← Back to root](README.md) · [roadmap.md](roadmap.md) · [hot-topics.md](hot-topics.md) · [soft-skills.md](soft-skills.md)
