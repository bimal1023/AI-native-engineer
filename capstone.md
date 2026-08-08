# Capstone: Agentic RAG With Evals and Observability

> One system that exercises every module. Build it, measure it, break it, write it up. [Back to root](README.md)

## The Brief

Build a **production-shaped research assistant** over a real document corpus: an agent that plans a multi-step answer, retrieves through a hybrid pipeline with reranking, uses tools, cites everything, runs inside hard budgets, is fully traced, and is defended by an eval suite that gates CI.

The point isn't the feature — plenty of people have built a chatbot over PDFs. The point is everything around it: the measurements, the gates, the failure handling, the cost table, and the security posture. That's the difference the repo exists to teach, and it's what makes this a portfolio piece rather than a demo.

**Time:** 3–4 weeks part-time · **Prerequisites:** Modules [01](01-llm-fundamentals/README.md)–[06](06-deployment-and-ai-infra/README.md) · **Deliverable:** a public repo with a README containing real numbers

---

## Pick a Real Domain

Choose a corpus you genuinely care about — motivation matters over four weeks, and domain knowledge makes you a better labeler.

- Internal engineering docs, runbooks, and incident reports
- An open-source project's source + docs + issue history
- A research area's arXiv papers (200–1000 of them)
- Public filings, legislation, or regulatory guidance
- Your own notes, saved articles, and bookmarks

**Requirements:** 500+ documents, genuinely messy (mixed formats, tables, code, inconsistent structure), and questions you actually want answered.

---

## System Requirements

| Layer | What it must do | Module |
|---|---|---|
| **Ingestion** | Structure-aware parsing and chunking, rich metadata, incremental re-index on change | [03](03-retrieval-and-rag/README.md) |
| **Retrieval** | Hybrid BM25 + dense with RRF, cross-encoder reranking, metadata filters | [03](03-retrieval-and-rag/README.md) |
| **Agent** | Multi-step planning, 4–6 tools, context compaction, hard step/time/cost budgets | [04](04-agents-and-tool-use/README.md) |
| **Generation** | Structured output, inline citations that resolve, explicit abstention path | [02](02-prompting-and-context-engineering/README.md) |
| **Evaluation** | Labeled retrieval set + golden answer set, calibrated judge, CI gate | [05](05-evaluation-and-observability/README.md) |
| **Observability** | Full traces with model/prompt versions, tokens, cost, TTFT; live dashboard | [05](05-evaluation-and-observability/README.md) |
| **Serving** | Streaming API, gateway with fallback, caching, rate limits, load-tested | [06](06-deployment-and-ai-infra/README.md) |
| **Security** | Least-privilege tools, approval gate, injection tests in CI | [04](04-agents-and-tool-use/README.md), [soft-skills](soft-skills.md#3-prompt-injection-and-security-awareness) |

---

## Milestones

### M0 · Scope and baseline *(2–3 days)*
- [ ] Pick the corpus; write a one-page spec: users, 10 example questions, acceptable failure rate, cost ceiling
- [ ] Ingest raw documents into storage with source metadata
- [ ] **Read 20 randomly sampled parsed chunks by hand** — fix parsing before anything else
- [ ] Ship a deliberately naive baseline: fixed chunks, dense-only top-5, single LLM call, no agent
- [ ] Record baseline cost per query and eyeball 10 answers

### M1 · Evaluation foundation *(4–5 days — do this before optimizing)*
- [ ] Write 50–100 real questions across easy / multi-hop / unanswerable / adversarial
- [ ] Label the chunk(s) each question should retrieve → **retrieval eval set**
- [ ] Write reference answers for 50 → **golden answer set**
- [ ] Implement deterministic checks: schema validity, citation resolution, abstention correctness on unanswerable questions
- [ ] Build an LLM judge for faithfulness and answer relevance
- [ ] **Calibrate the judge:** hand-label 50, report agreement, iterate the rubric to ≥80%
- [ ] Run the full suite against the M0 baseline and commit the numbers as `baseline.json`

### M2 · Retrieval quality *(4–5 days)*
- [ ] Structure-aware chunking with section paths in metadata — measure
- [ ] Add BM25 and fuse with RRF — measure
- [ ] Add cross-encoder reranking over top-50 → top-8 — measure
- [ ] Add contextual retrieval (LLM-generated chunk context) — measure
- [ ] Add query rewriting for conversational follow-ups — measure
- [ ] Publish a variant × recall@5 × nDCG × latency × cost table
- [ ] **Write down which changes didn't earn their complexity** and remove them

### M3 · Agent layer *(4–5 days)*
- [ ] Design 4–6 tools with precise schemas: `search_corpus`, `fetch_document`, `write_note`, `read_notes`, one external tool, one approval-gated action
- [ ] Implement the agent loop with hard budgets: ≤15 steps, ≤3 min, ≤$0.25 per query
- [ ] Context compaction: summarize completed sub-goals, keep detail in notes
- [ ] Durable state — a killed process resumes mid-run
- [ ] Structured final answer with inline citations that resolve to real chunks
- [ ] Explicit abstention when evidence is insufficient
- [ ] Add trajectory evaluation: tool-choice accuracy, steps-to-completion, termination-reason distribution
- [ ] Compare agent vs. M2 single-pass pipeline on the same eval set — **be honest if the simpler one wins on most queries**

### M4 · Observability and CI *(3–4 days)*
- [ ] OTel-compatible tracing on every LLM call, retrieval, and tool invocation
- [ ] Every span carries: model, prompt version, tokens (in/out/cached/reasoning), cost, TTFT, total latency, stop reason
- [ ] Dashboard: cost per query, p50/p95/p99 latency, error rate, cache hit rate, abstention rate, budget-exhaustion rate
- [ ] GitHub Action runs the golden set on every PR touching prompts, models, tools, or retrieval config
- [ ] CI fails on accuracy regression >2% or cost regression >20% vs. `baseline.json`
- [ ] Feedback route: thumbs-down traces land in a review queue, then flow into the eval set

### M5 · Serving and hardening *(3–4 days)*
- [ ] Streaming SSE API — verify TTFT end-to-end through your proxy, not just locally
- [ ] Gateway layer: timeouts, jittered retries, circuit breaker, per-key rate limits, exact-match cache
- [ ] Model fallback to a second provider; run the eval set against the fallback path too
- [ ] Model tiering: route simple lookups to a small model, escalate on complexity — report the cost delta
- [ ] Prompt caching on the stable prefix — report the cost and TTFT delta
- [ ] Load test at 1 / 10 / 50 concurrent users; report percentiles and error rates
- [ ] **Security pass:** least-privilege credentials, approval gate on the mutating tool, allowlisted outbound destinations
- [ ] **Red-team:** plant prompt injections in 5 corpus documents; add them as permanent CI test cases; demonstrate they fail safely

### M6 · Write-up *(2 days — this is the part people read)*
- [ ] Architecture diagram and a short "why this design" section
- [ ] Results table: baseline → final, on every metric
- [ ] The retrieval-variant table from M2
- [ ] Cost breakdown per query and projected monthly cost at 10K queries/day
- [ ] Latency percentiles at each concurrency level
- [ ] **A failure-modes section** — what it still gets wrong, with examples
- [ ] **A "what I'd do differently" section** — this is the strongest signal of engineering maturity in the whole repo
- [ ] A capability statement in the style of [soft-skills §4](soft-skills.md#4-communicating-ai-system-limitations)
- [ ] Reproducible setup: one command to ingest, one to eval, one to serve

---

## Acceptance Criteria

Ship it when all of these are true:

- [ ] Retrieval recall@5 improved **measurably** over the M0 baseline, with the number published
- [ ] Faithfulness ≥90% on the golden set, judged by a judge with ≥80% human agreement
- [ ] 100% of citations resolve to real chunks
- [ ] Unanswerable questions get abstention, not invention, ≥90% of the time
- [ ] Every agent run terminates within budget — no exceptions across 100 runs
- [ ] A quality-regressing PR is blocked by CI, demonstrated with a screenshot
- [ ] Planted prompt injections fail safely and are permanent test cases
- [ ] Any run is replayable from its trace
- [ ] Cost per query and p95 latency are stated as numbers in the README

---

## Stretch Goals

- **Self-host** the generation model with vLLM and publish self-hosted vs. API cost at your real volume
- **Distill** the reranker or router into a small fine-tuned model and show quality/cost parity
- **Multi-hop evaluation** — questions requiring 3+ retrieval steps, scored on trajectory
- **Expose the corpus as an MCP server** so any MCP client can query it
- **Multimodal ingestion** — page-image retrieval for documents where text extraction fails
- **Multi-tenant** with permission-filtered retrieval and a test proving cross-tenant isolation

---

## How to Present It

Recruiters and senior engineers skim. Put the evidence where they'll see it.

1. **Lead with the results table**, not the architecture. Numbers first, prose second.
2. **Show one honest failure.** A documented failure mode signals more competence than a flawless demo, because everyone knows the flawless demo isn't real.
3. **Link the traces.** A screenshot of a real trace with cost and token counts proves you actually operated this.
4. **Write the "what I'd do differently" section properly.** It's the most-read paragraph in any good engineering write-up.
5. **Keep a 60-second demo video** at the top — the eval harness and dashboard, not just the chat UI.

---

## Progress

- [ ] M0 · Scope and baseline
- [ ] M1 · Evaluation foundation
- [ ] M2 · Retrieval quality
- [ ] M3 · Agent layer
- [ ] M4 · Observability and CI
- [ ] M5 · Serving and hardening
- [ ] M6 · Write-up
- [ ] All acceptance criteria met
- [ ] Published and shared

---

[← 07 · Emerging Topics](07-emerging-topics/README.md) · [Back to root](README.md) · [hot-topics.md](hot-topics.md) · [soft-skills.md](soft-skills.md)
