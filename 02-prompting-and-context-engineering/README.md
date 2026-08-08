# 02 · Prompting & Context Engineering

> Prompting is what you write. Context engineering is what you decide to put in the window — and what you leave out.

## Why This Matters

The industry moved from "prompt engineering" as clever phrasing to **context engineering**: treating the context window as a scarce, expensive, curated resource assembled at runtime from instructions, retrieved documents, tool results, conversation history, and memory. Phrasing still matters, but the bigger wins come from what you include, in what order, in what format, and what you compact away. This module is also where the discipline gap shows most clearly — most teams have prompts in string literals with no version, no test, and no owner.

---

## Key Subtopics

### 2.1 Prompt anatomy and instruction following

A production prompt has a stable structure: role and task in the system prompt, then background/context, then examples, then the specific request, then explicit output-format instructions. Order matters for two reasons — models attend most reliably to the beginning and end of the window, and a stable prefix is what makes prompt caching pay off. Use clear delimiters (XML tags or markdown fences) to separate instructions from data; this is both a quality and a security boundary. Prefer positive instructions ("respond in ≤3 sentences") over negations ("don't be verbose").

- [Anthropic: Prompt engineering overview](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview) — the most practical vendor guide
- [OpenAI: Prompt engineering guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic: Use XML tags](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags) — structure beats prose for complex inputs

### 2.2 Few-shot examples and reasoning elicitation

Few-shot examples teach *format and edge-case handling* far more efficiently than instructions do. Three to five diverse examples — including the tricky and null cases — usually beat a page of rules. For multi-step problems, chain-of-thought ("work through it step by step before answering") improves accuracy on non-reasoning models; self-consistency (sample *n*, take the majority) improves it further at *n*× the cost. Important 2025–26 caveat: with **reasoning models**, explicit CoT instructions are redundant or actively harmful — give them the goal and constraints, set a thinking budget, and get out of the way.

- [Chain-of-Thought Prompting Elicits Reasoning](https://arxiv.org/abs/2201.11903)
- [Self-Consistency Improves Chain of Thought Reasoning](https://arxiv.org/abs/2203.11171)
- [Anthropic: Extended thinking tips](https://docs.claude.com/en/docs/build-with-claude/extended-thinking) — how prompting changes for reasoning models

### 2.3 Structured output and schema enforcement

Any prompt whose output feeds another system needs a schema, not a hope. Three mechanisms, in increasing order of reliability: prompt-and-parse (fragile), **tool/function calling** with a JSON Schema (good — the model is trained for it), and **constrained decoding**, where the sampler is masked so only grammar-valid tokens can be emitted (strongest). Always validate against your schema on receipt, and always have a retry path that feeds the validation error back to the model. Design schemas so a partial answer is expressible — give the model a `null` or `"insufficient_evidence"` option, or it will invent a value to satisfy a required field.

- [OpenAI: Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs)
- [Anthropic: Tool use / JSON output](https://docs.claude.com/en/docs/agents-and-tools/tool-use/overview)
- [Instructor](https://python.useinstructor.com/) and [Outlines](https://dottxt-ai.github.io/outlines/) — Pydantic-typed responses and grammar-constrained generation

### 2.4 Context engineering: assembly, compaction, and caching

Context is a budget. Spend it deliberately.

- **Assembly** — build the window at runtime from system prompt + retrieved chunks + tool results + history. Put stable content first (cacheable), volatile content last.
- **Compaction** — as conversations or agent loops grow, summarize older turns, drop superseded tool output, and keep a durable scratchpad *outside* the window (a file, a store) rather than carrying everything forward.
- **Position** — put the critical instruction near the end when the context is long; middle content is recalled worst.
- **Caching** — provider prompt caching can cut input cost by up to ~90% and TTFT substantially on repeated prefixes, but only if the prefix is byte-identical. Never put a timestamp or request ID at the top of a cached prompt.

Long context does not eliminate retrieval. Filling 200K tokens is slow, expensive, and measurably degrades precision versus feeding 8K of well-chosen context.

- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Lost in the Middle](https://arxiv.org/abs/2307.03172)
- [Anthropic: Prompt caching](https://docs.claude.com/en/docs/build-with-claude/prompt-caching)

### 2.5 Prompt versioning, testing, and optimization

Prompts are production code with a worse blast radius. Minimum bar: prompts live in version-controlled files (not inline strings), every prompt has an ID and a version, every logged trace records which prompt version and model version produced it, and no prompt change ships without running the eval set. Above that bar sits **programmatic optimization** — DSPy and similar frameworks treat prompts as parameters compiled against a metric, so you optimize few-shot selection and instructions automatically instead of by hand. Pair this with [Module 05](../05-evaluation-and-observability/README.md); prompt iteration without an eval set is a random walk.

- [DSPy](https://dspy.ai/) — programming, not prompting; optimizers that compile prompts against a metric
- [Anthropic: prompt improver & templates](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/prompt-improver)
- [Hamel Husain: Your AI product needs evals](https://hamel.dev/blog/posts/evals/) — why prompt iteration without measurement fails

---

## Tools & Frameworks Used in Industry

| Purpose | Tools |
|---|---|
| Typed / validated output | [Instructor](https://python.useinstructor.com/), [Pydantic AI](https://ai.pydantic.dev/), [Outlines](https://dottxt-ai.github.io/outlines/), [BAML](https://docs.boundaryml.com/) |
| Prompt optimization | [DSPy](https://dspy.ai/), [TextGrad](https://github.com/zou-group/textgrad) |
| Prompt management & versioning | [Langfuse](https://langfuse.com/), [LangSmith](https://docs.smith.langchain.com/), [Braintrust](https://www.braintrust.dev/), [PromptLayer](https://www.promptlayer.com/) |
| Templating | Jinja2, [LangChain](https://python.langchain.com/) prompt templates, plain f-strings + files |
| Prompt testing | [promptfoo](https://www.promptfoo.dev/), [DeepEval](https://docs.confident-ai.com/), pytest + a fixed input set |
| Context/token accounting | `tiktoken`, provider token-count endpoints, custom budget middleware |

---

## Hands-On Project: Structured Extraction Pipeline

**Build a service that turns messy documents into validated, typed records — and prove it's reliable.**

Requirements:
1. Pick a real corpus with genuine messiness (receipts, job postings, arXiv abstracts, changelogs). Target ~50 documents with hand-labeled ground truth.
2. Define a Pydantic schema with required fields, optional fields, and an explicit `null`/`unknown` path. No field should be unfillable-but-required.
3. Extract using tool calling or structured outputs; validate on receipt; on validation failure, retry once with the error message appended. Log every retry.
4. Store prompts as versioned files with IDs. Log `prompt_version`, `model`, tokens, and cost per call.
5. Report field-level accuracy, schema-validity rate, hallucination rate on absent fields, and cost per document.
6. Run three variants — zero-shot, 5-shot, and cached-prefix — and publish a comparison table.

**Done when:** schema-validity is ≥99%, you can attribute every quality change to a specific prompt version, and you can show the cost delta from prompt caching.

**Stretch:** re-implement the extractor in DSPy and see whether an optimizer beats your hand-tuned prompt on the same eval set.

---

## Common Pitfalls

- **Prompts as inline string literals.** Unversioned, untested, unattributable. Move them to files on day one.
- **Iterating on prompts without an eval set.** You'll fix example A, silently break example B, and call it progress.
- **Overstuffing few-shot examples.** More than ~5 usually adds cost and biases the model toward the examples' surface form. Diversity beats volume.
- **Cache-busting your own prefix.** A timestamp, UUID, or reordered tool list at the top of the prompt zeroes out your cache hit rate.
- **Required fields with no escape hatch.** If the schema demands `due_date` and the document has none, the model invents one.
- **Mixing untrusted data with instructions.** Retrieved documents and user content must be clearly delimited and treated as data — see [soft-skills.md](../soft-skills.md#3-prompt-injection-and-security-awareness).
- **Forcing chain-of-thought onto reasoning models.** They already think; extra instructions can degrade output.
- **Assuming long context replaces retrieval.** It's slower, pricier, and less precise than good retrieval ([Module 03](../03-retrieval-and-rag/README.md)).
- **Prompting around a data problem.** If ground truth is ambiguous to a human labeler, no prompt fixes it.

---

## Progress

- [ ] 2.1 Prompt anatomy and instruction following
- [ ] 2.2 Few-shot examples and reasoning elicitation
- [ ] 2.3 Structured output and schema enforcement
- [ ] 2.4 Context engineering: assembly, compaction, and caching
- [ ] 2.5 Prompt versioning, testing, and optimization
- [ ] **Project:** structured extraction pipeline

---

[← 01 · LLM Fundamentals](../01-llm-fundamentals/README.md) · [Back to root](../README.md) · [Next: 03 · Retrieval & RAG →](../03-retrieval-and-rag/README.md)
