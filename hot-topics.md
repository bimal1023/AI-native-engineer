# Hot Topics vs. Stable Fundamentals

> **Last reviewed:** August 2026 · Review cadence: quarterly · [Back to root](README.md)

Not all of this repo ages at the same rate. Some of it has been true since 2017 and will be true in 2030; some of it changed twice this year. This file tells you where to spend your re-reading time — and, just as importantly, where *not* to.

**Velocity key**

| | Meaning | Revisit |
|---|---|---|
| 🔥 | High velocity — materially changed in the last 6–12 months | Monthly |
| 🌤 | Moderate — practices consolidating, details still shifting | Quarterly |
| 🪨 | Stable — learn once, revisit rarely | Yearly, or never |

---

## 🔥 High Velocity — Check Monthly

| Area | Module | What changed recently | Why it matters |
|---|---|---|---|
| **Reasoning models & thinking budgets** | [07](07-emerging-topics/README.md#71-reasoning-models-and-test-time-compute) | Test-time compute went from research result to a per-request dial across every major provider; open reproductions (R1-style) commoditized the technique | Changes model selection, cost modeling, and streaming UX for every multi-step task |
| **Agentic coding & harness design** | [07](07-emerging-topics/README.md#72-agentic-coding-and-swe-agents) | The bottleneck moved from model capability to harness design — context strategy, sub-agents, verification loops, approval UX | Directly changes your own workflow and how your team reviews code |
| **MCP and agent protocols** | [04](04-agents-and-tool-use/README.md#44-environments-and-protocols-mcp-sandboxes-computer-use), [07](07-emerging-topics/README.md#73-protocols-and-interoperability) | MCP went from new spec to de-facto standard; auth, scoping, and server-trust practices are still being written. A2A is earlier | Determines how you integrate tools; the security guidance is genuinely unsettled |
| **Agent security & prompt injection** | [04](04-agents-and-tool-use/README.md#45-reliability-budgets-retries-human-in-the-loop), [soft-skills](soft-skills.md#3-prompt-injection-and-security-awareness) | New attack classes arrive faster than defenses; no reliable general mitigation exists — architecture is the control | Agents with write access and untrusted input are the highest-risk thing in this repo |
| **Small-model capability & distillation** | [07](07-emerging-topics/README.md#75-small-models-distillation-and-on-device) | The 1–8B tier keeps absorbing tasks that needed frontier models; distillation recipes are now routine | The largest available cost lever, re-evaluate every few months |
| **Pricing, model versions, context limits** | [01](01-llm-fundamentals/README.md#15-model-selection-scaling-laws-and-reasoning-models), [06](06-deployment-and-ai-infra/README.md#63-cost-and-latency-engineering) | Constantly. Prices fall, models deprecate, limits move | Any cost or model-choice conclusion older than ~3 months should be re-verified against live docs |
| **Multimodal document AI & realtime voice** | [07](07-emerging-topics/README.md#74-multimodal-realtime-and-computer-use) | Page-image retrieval competing with text-extraction pipelines; speech-to-speech collapsed the STT→LLM→TTS chain | Rewrites ingestion architecture in [03](03-retrieval-and-rag/README.md) and makes voice agents viable |

---

## 🌤 Moderate Velocity — Check Quarterly

| Area | Module | State of play |
|---|---|---|
| **Context engineering** | [02](02-prompting-and-context-engineering/README.md#24-context-engineering-assembly-compaction-and-caching) | The framing ("engineer the window, not the sentence") has stabilized; specific compaction and memory patterns are still consolidating |
| **Agent architectures** | [04](04-agents-and-tool-use/README.md#42-agent-architectures-workflows-vs-agents) | The workflow-vs-agent taxonomy has held up well; multi-agent guidance is still shifting, mostly toward "less than you think" |
| **Eval tooling** | [05](05-evaluation-and-observability/README.md) | Eval *methodology* is stable; the vendor landscape churns and consolidates constantly. Learn the method, hold tools loosely |
| **Observability standards** | [05](05-evaluation-and-observability/README.md#53-tracing-metrics-and-otel-genai-conventions) | OTel GenAI semantic conventions are converging but still evolving; adopting them now is still the right call |
| **Serving optimizations** | [06](06-deployment-and-ai-infra/README.md#61-serving-and-inference-optimization) | vLLM/SGLang are stable choices; quantization formats, speculative decoding, and disaggregated prefill/decode keep improving |
| **Embedding & reranking models** | [03](03-retrieval-and-rag/README.md#31-embeddings-and-vector-search) | New SOTA models land regularly, but the *architecture* (hybrid + rerank) hasn't changed. Re-benchmark models, don't redesign |
| **Fine-tuning economics** | [06](06-deployment-and-ai-infra/README.md#64-fine-tuning-vs-rag-vs-prompting) | LoRA/QLoRA are settled technique; when fine-tuning beats prompting keeps moving as base models improve |
| **AI governance & regulation** | [07](07-emerging-topics/README.md#76-safety-interpretability-and-governance) | EU AI Act obligations phasing in through 2026–2027; enterprise procurement requirements tightening |

---

## 🪨 Stable Fundamentals — Learn Once

| Area | Module | Why it's durable |
|---|---|---|
| **Transformer architecture & attention** | [01](01-llm-fundamentals/README.md#11-transformers-and-the-attention-mechanism) | Essentially unchanged since 2017. Efficiency variants come and go; the core mechanism holds |
| **Tokenization** | [01](01-llm-fundamentals/README.md#12-tokenization-and-the-context-window) | BPE-family tokenizers, ~4 chars/token for English, code and non-Latin cost more. Won't change |
| **Autoregressive decoding & sampling** | [01](01-llm-fundamentals/README.md#14-inference-and-decoding-sampling-params-kv-cache-streaming) | Temperature, top-p, KV cache, TTFT vs. per-token latency — the mechanics are fixed |
| **Training lifecycle** | [01](01-llm-fundamentals/README.md#13-the-training-lifecycle-pretraining--sft--preference-tuning) | Pretrain → SFT → preference-tune. Algorithms change; the shape doesn't |
| **Information retrieval fundamentals** | [03](03-retrieval-and-rag/README.md) | BM25, precision/recall, nDCG, hybrid fusion, cross-encoder reranking — decades old and still correct |
| **Eval methodology** | [05](05-evaluation-and-observability/README.md#51-eval-fundamentals-and-error-analysis) | Error analysis → taxonomy → labeled set → measure → gate. This is the most durable skill in the repo |
| **Distributed-systems discipline** | [04](04-agents-and-tool-use/README.md#45-reliability-budgets-retries-human-in-the-loop), [06](06-deployment-and-ai-infra/README.md#65-reliability-and-operations) | Timeouts, idempotency, backoff with jitter, circuit breakers, least privilege, SLOs. Unchanged and non-negotiable |
| **Cost/latency reasoning** | [06](06-deployment-and-ai-infra/README.md#63-cost-and-latency-engineering) | Prices change; the *levers* (tier, cache, cap output, batch) don't |
| **Structured output discipline** | [02](02-prompting-and-context-engineering/README.md#23-structured-output-and-schema-enforcement) | Schema + validate + retry has been the right answer since function calling existed |

---

## Where to Spend Your Attention

If you have **four hours a month** for staying current, spend it roughly like this:

- **2h** — one 🔥 area, hands-on. Reproduce something, measure it against your own baseline, write it down.
- **1h** — provider changelogs and docs diffs for whatever you run in production (pricing, deprecations, new params).
- **1h** — one primary source: a paper or an engineering post, read properly rather than five skimmed threads.

Spend **zero** hours on: model-release Twitter discourse, benchmark leaderboard reshuffles, "X is dead" posts, and new framework announcements with no production users. If something matters, it will still be there in three months with real write-ups attached.

## Signals Worth Following

- **Provider engineering blogs** — [Anthropic Engineering](https://www.anthropic.com/engineering), [OpenAI](https://openai.com/news/), [Google DeepMind](https://deepmind.google/discover/blog/)
- **Practitioners** — [Hamel Husain](https://hamel.dev/), [Eugene Yan](https://eugeneyan.com/), [Simon Willison](https://simonwillison.net/), [Chip Huyen](https://huyenchip.com/blog/)
- **Infra** — [vLLM blog](https://blog.vllm.ai/), [Hugging Face blog](https://huggingface.co/blog)
- **Specs** — [MCP changelog](https://modelcontextprotocol.io/), [OTel GenAI semconv](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- **Papers** — [arXiv cs.CL](https://arxiv.org/list/cs.CL/recent) via a filter or digest, not raw

---

## Keeping This File Honest

When you review, update the date at the top and use this template per change:

```markdown
| **Area name** | [module](path) | What actually changed, in one sentence | Why an engineer should care |
```

Rules: **date every claim**, demote aggressively (most 🔥 entries should become 🌤 within a year), and delete entries that turned out not to matter rather than leaving them as clutter. A hot-topics file that only grows is a stale one.

---

[← Back to root](README.md) · [soft-skills.md](soft-skills.md) · [capstone.md](capstone.md)
