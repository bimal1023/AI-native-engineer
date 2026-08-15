# Roadmap: What to Learn, In What Order

> Seven modules is a lot. This page tells you which ones *you* need, in what order, and how to know when you're done. [Back to root](README.md)

The modules are numbered foundational → advanced, but almost nobody should read all seven straight through. What you need depends on what you're trying to do. Pick a path below, follow it, and ignore the rest until you need it.

---

## Start Here: Which Path Are You On?

| If you… | Follow | Time |
|---|---|---|
| Need to ship an AI feature at work, soon | [Path A](#path-a--ship-one-feature-fast) | ~3 weeks |
| Want an AI engineer role | [Path B](#path-b--the-full-ai-engineer) | ~3 months |
| Work on search, docs, or knowledge products | [Path C](#path-c--retrieval-specialist) | ~6 weeks |
| Are building agents or agentic tooling | [Path D](#path-d--agent-developer) | ~6 weeks |
| Own infrastructure, platform, or cost | [Path E](#path-e--ai-platform--infra) | ~6 weeks |
| Are interviewing for AI engineering roles | [Path F](#path-f--interview-prep) | ~4 weeks |

All paths start with [Module 01](01-llm-fundamentals/README.md). There's no version of this where you skip fundamentals and it goes well.

---

## Module Dependencies

```
                    01 · Fundamentals
                            │
                            ▼
                    02 · Prompting & Context
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
        03 · RAG      04 · Agents    06 · Infra
              └─────────────┼─────────────┘
                            ▼
                    05 · Evals & Observability
                            │
                            ▼
                    07 · Emerging  →  Capstone
```

**Read this graph with one correction:** 05 is drawn last because it *depends* on having something to measure, but you should start it early — as soon as you have any working feature. The repo's core argument is that evals come before optimization, not after.

---

## Time Estimates

Honest ranges for reading plus the module project, at a working engineer's pace.

| Module | Reading | Project | Total |
|---|---|---|---|
| [01 · Fundamentals](01-llm-fundamentals/README.md) | 4–6 h | 6 h | ~10 h |
| [02 · Prompting & Context](02-prompting-and-context-engineering/README.md) | 3 h | 6 h | ~9 h |
| [03 · Retrieval & RAG](03-retrieval-and-rag/README.md) | 4 h | 12 h | ~16 h |
| [04 · Agents & Tool Use](04-agents-and-tool-use/README.md) | 4 h | 12 h | ~16 h |
| [05 · Evals & Observability](05-evaluation-and-observability/README.md) | 3 h | 10 h | ~13 h |
| [06 · Deployment & Infra](06-deployment-and-ai-infra/README.md) | 4 h | 10 h | ~14 h |
| [07 · Emerging Topics](07-emerging-topics/README.md) | 3 h | 4 h | ~7 h |
| [Capstone](capstone.md) | — | 40–60 h | ~50 h |

Full repo including capstone: **roughly 135 hours**. At 8 hours a week that's four months; at 20 hours a week, seven weeks.

---

## The Paths

### Path A — Ship one feature fast

**For:** an engineer with a deadline and a product manager asking when it'll be done.

1. [01 · Fundamentals](01-llm-fundamentals/README.md) — §1.1, 1.2, 1.6, 1.7, 1.8 only. Skip architecture variants and quantization.
2. [02 · Prompting & Context](02-prompting-and-context-engineering/README.md) — all of it, especially structured output.
3. [05 · Evals](05-evaluation-and-observability/README.md) — §5.1 and 5.4. Build the golden set before you optimize anything.
4. [03 · RAG](03-retrieval-and-rag/README.md) — only if the feature needs your own data.
5. [soft-skills §2](soft-skills.md#2-cost-and-latency-tradeoffs) — so you can answer "what will this cost?"

**Skip for now:** agents, infra, emerging topics.

**Done when:** the feature ships with a schema, an eval set, tracing, and a cost-per-request number you can quote from memory.

### Path B — The full AI engineer

**For:** changing roles, or wanting genuine breadth.

Straight through 01 → 07, then the [capstone](capstone.md). Two deviations from strict order:

- Start [05](05-evaluation-and-observability/README.md) right after 02 rather than waiting. Build the eval harness on your prompting project, then reuse it for every module after.
- Do [07](07-emerging-topics/README.md) last, and treat it as a habit rather than a module — reread it quarterly against [hot-topics.md](hot-topics.md).

**Done when:** the capstone is shipped and written up with real numbers.

### Path C — Retrieval specialist

**For:** search, documentation products, internal knowledge tools, anything where "find the right thing" is the product.

1. [01](01-llm-fundamentals/README.md) — §1.2, 1.3 (embeddings), 1.7 (hallucination)
2. [02](02-prompting-and-context-engineering/README.md) — §2.3, 2.4
3. [03](03-retrieval-and-rag/README.md) — **all of it, twice**. This is your module.
4. [05](05-evaluation-and-observability/README.md) — §5.1, 5.5. Retrieval evals are the whole game.
5. [07](07-emerging-topics/README.md) — §7.4, page-image retrieval is actively changing ingestion.

**Done when:** you can show a recall@5 improvement chart with each change attributed.

### Path D — Agent developer

**For:** building agentic products, internal automation, or agent tooling.

1. [01](01-llm-fundamentals/README.md) — §1.6, 1.7, 1.8. Knowing the failure surface matters more here than anywhere.
2. [02](02-prompting-and-context-engineering/README.md) — §2.3, 2.4. Context compaction is load-bearing for agents.
3. [04](04-agents-and-tool-use/README.md) — **all of it**. Read [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) before writing any code.
4. [05](05-evaluation-and-observability/README.md) — §5.3, 5.5. Trajectory evaluation, not just final answers.
5. [soft-skills §3](soft-skills.md#3-prompt-injection-and-security-awareness) — **non-negotiable.** Agents with write access are the highest-risk thing in this repo.
6. [03](03-retrieval-and-rag/README.md) — when your agent needs to search.

**Done when:** 20/20 agent runs terminate within budget and a planted prompt injection fails safely.

### Path E — AI platform & infra

**For:** owning the serving layer, the gateway, the GPU bill, or the internal AI platform.

1. [01](01-llm-fundamentals/README.md) — §1.4, 1.6, 1.9. MoE, KV cache, and quantization are your daily vocabulary.
2. [06](06-deployment-and-ai-infra/README.md) — **all of it**. This is your module.
3. [05](05-evaluation-and-observability/README.md) — §5.3, 5.4. You'll own the tracing and the CI gate.
4. [02](02-prompting-and-context-engineering/README.md) — §2.4, specifically prompt caching. It's the biggest cost lever you control.
5. [04](04-agents-and-tool-use/README.md) — §4.5, since agent workloads break capacity planning.

**Done when:** you can state $/1M tokens and p95 latency at three concurrency levels, from your own load test.

### Path F — Interview prep

**For:** interviewing for AI engineer / ML engineer roles in the next month.

1. [01](01-llm-fundamentals/README.md) — all of it. This is where the "do they actually understand it" questions come from.
2. [02](02-prompting-and-context-engineering/README.md) and [03](03-retrieval-and-rag/README.md) — RAG design is the most common AI system design question.
3. [05](05-evaluation-and-observability/README.md) — "how would you know if it's working?" is asked constantly, and most candidates have no answer.
4. [06 §6.3](06-deployment-and-ai-infra/README.md#63-cost-and-latency-engineering) — cost and latency reasoning.
5. [soft-skills §4](soft-skills.md#4-communicating-ai-system-limitations) — explaining limitations is half the interview.

**Prepare specifically:** design a RAG system out loud in 45 minutes. Explain how you'd evaluate it. Explain what you'd do when it returns a wrong answer. Those three questions cover most AI system design rounds.

---

## Levels: How to Know Where You Are

**Level 1 — Competent API user** *(after 01–02)*
- [ ] Can estimate cost and latency for a feature before building it
- [ ] Get reliable structured output with schema validation and retries
- [ ] Know why the model fails at the things it fails at

**Level 2 — Can ship something that holds up** *(after 05, plus 03 or 04)*
- [ ] Have a labeled eval set and a CI gate on it
- [ ] Can attribute a quality change to a specific prompt or model version
- [ ] Every request is traced with tokens, cost, and latency

**Level 3 — Can build the hard things** *(after 03 and 04)*
- [ ] Can decompose a wrong answer into retrieval failure vs generation failure
- [ ] Can design an agent loop that always terminates and recovers from tool errors
- [ ] Can name which leg of the lethal trifecta your system breaks

**Level 4 — Can run it in production** *(after 06)*
- [ ] Know your p95, your cost per request, and your failure behaviour under load
- [ ] Have a fallback path and have tested it
- [ ] Can defend build-vs-buy with a break-even volume

**Level 5 — Can evaluate the frontier** *(after 07 + capstone)*
- [ ] Can reproduce a new capability and measure it against your own baseline
- [ ] Can say "not yet, and here's the condition that would change my answer"
- [ ] Have shipped one complete system and written up what you'd do differently

---

## What to Skip

Genuinely optional unless it's your job:

- **Training or fine-tuning a model from scratch.** Interesting; almost never the answer. See [§6.4](06-deployment-and-ai-infra/README.md#64-fine-tuning-vs-rag-vs-prompting).
- **The math behind backpropagation.** You need the mental model, not the derivation.
- **Every new framework.** Learn the pattern; frameworks churn. [hot-topics.md](hot-topics.md) tags which is which.
- **Benchmark leaderboards.** A prior for what to test, never evidence about your data.
- **Building your own vector database.** Use pgvector.

And two time-sinks that feel productive but aren't: collecting prompt libraries you never test, and reading about agents without building one that terminates.

---

## If You Only Have…

**A weekend:** [01](01-llm-fundamentals/README.md) §1.1–1.2 and 1.6–1.8, then [02](02-prompting-and-context-engineering/README.md), then build the structured extraction project. You'll be meaningfully more dangerous by Sunday.

**A month:** Path A, plus the [module 05](05-evaluation-and-observability/README.md) project. Evals are what separate you from everyone else who did a weekend.

**Three months:** Path B, all seven modules, capstone shipped and written up.

**Ongoing, already working in this:** [hot-topics.md](hot-topics.md) quarterly, four hours a month, and one capability memo per quarter.

---

## Progress

- [ ] Picked a path and wrote down which one
- [ ] Level 1 checklist complete
- [ ] Level 2 checklist complete
- [ ] Level 3 checklist complete
- [ ] Level 4 checklist complete
- [ ] Level 5 checklist complete

---

[← Back to root](README.md) · [hot-topics.md](hot-topics.md) · [soft-skills.md](soft-skills.md) · [capstone.md](capstone.md)
