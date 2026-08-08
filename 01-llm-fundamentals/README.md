# 01 · LLM Fundamentals

> What the model is actually doing, and how to choose one on evidence instead of vibes.

## Why This Matters

Almost every production LLM bug traces back to a fundamentals gap. Truncated output? Output-token limit, not context window. Non-deterministic results at `temperature=0`? Batching and floating-point nondeterminism, not a bug in your code. Costs 4× the estimate? You priced input tokens and forgot output tokens are 3–5× more expensive. You don't need to train a model, but you need an accurate mental model of tokens in → tokens out, and what that costs in money and milliseconds.

---

## Key Subtopics

### 1.1 Transformers and the attention mechanism

A decoder-only transformer predicts the next token by letting each position attend to all previous positions. Stack that ~30–100 times with feed-forward layers in between and you get an LLM. The parts worth internalizing: self-attention is *quadratic* in sequence length (why long context is expensive), the model is *stateless* between calls (why you resend the whole conversation every turn), and generation is *autoregressive* (why output tokens cost more latency than input tokens).

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) — the original paper; skim §3
- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) — read this first if the paper is opaque
- [Karpathy: Let's build GPT from scratch](https://www.youtube.com/watch?v=kCc8FmEb1nY) — 2 hours, the single highest-leverage resource here

### 1.2 Tokenization and the context window

Models see tokens, not characters or words. Byte-pair encoding merges frequent character sequences, so English prose runs ~4 chars/token while code, JSON, non-Latin scripts, and long numbers are far less efficient. Three separate limits matter and get conflated constantly: **context window** (input + output combined), **max output tokens** (usually much smaller), and **effective** context — the point past which recall degrades even though the model technically accepts more.

- [Neural Machine Translation of Rare Words with Subword Units](https://arxiv.org/abs/1508.07909) — BPE, the source
- [Tiktokenizer](https://tiktokenizer.vercel.app/) — paste text, see the tokens; do this before estimating any cost
- [Lost in the Middle](https://arxiv.org/abs/2307.03172) — retrieval accuracy degrades for content buried mid-context

### 1.3 The training lifecycle: pretraining → SFT → preference tuning

Pretraining on a large corpus produces a next-token predictor with broad knowledge and no manners. Supervised fine-tuning on instruction/response pairs teaches it to follow instructions. Preference tuning (RLHF, DPO, or variants) aligns it with human ratings for helpfulness and safety. This explains the model's personality: it's optimized for *rated-as-good* responses, which is why it's agreeable, why it hedges, and why it will confidently produce a plausible answer rather than say "I don't know."

- [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155) — InstructGPT; the RLHF blueprint
- [Direct Preference Optimization](https://arxiv.org/abs/2305.18290) — the simpler alternative that much of the field moved to
- [Constitutional AI](https://arxiv.org/abs/2212.08073) — AI feedback instead of human labels for harmlessness

### 1.4 Inference and decoding: sampling params, KV cache, streaming

At each step the model outputs a probability distribution over the vocabulary; decoding parameters shape how you sample from it. `temperature` flattens or sharpens the distribution; `top_p` truncates it to the smallest set summing to *p*. Tune one, not both. The **KV cache** stores attention keys/values for tokens already processed so each new token is O(1) rather than O(n) — it's also why prompt caching works and why long conversations eat GPU memory. Latency decomposes into **TTFT** (time to first token, dominated by input length) and **TPOT/ITL** (per-output-token time). Streaming doesn't make generation faster; it makes TTFT the number the user feels.

- [How to generate text](https://huggingface.co/blog/how-to-generate) — greedy, beam, top-k, top-p, side by side
- [Efficient Memory Management for LLM Serving with PagedAttention](https://arxiv.org/abs/2309.06180) — the vLLM paper; KV cache as a memory-management problem
- [Anthropic: Prompt caching](https://docs.claude.com/en/docs/build-with-claude/prompt-caching) — the practical payoff of a reusable prefix

### 1.5 Model selection, scaling laws, and reasoning models

Scaling laws describe how loss falls with compute, parameters, and data — Chinchilla corrected the field toward more data per parameter, and post-2024 the frontier moved again toward spending compute at *inference* time. **Reasoning models** generate hidden intermediate tokens before answering, trading latency and cost for accuracy on multi-step problems. They are not a default upgrade: for extraction, classification, routing, and formatting, a fast non-reasoning model is usually better and 10× cheaper. Choose per task, on your own eval set, measuring quality *and* cost *and* p95 latency.

- [Scaling Laws for Neural Language Models](https://arxiv.org/abs/2001.08361) and [Training Compute-Optimal LLMs](https://arxiv.org/abs/2203.15556) (Chinchilla)
- [Scaling LLM Test-Time Compute Optimally](https://arxiv.org/abs/2408.03314) — why thinking longer beats being bigger, sometimes
- [DeepSeek-R1](https://arxiv.org/abs/2501.12948) — open recipe for RL-trained reasoning
- [LMArena](https://lmarena.ai/) — useful for a rough prior, useless as a substitute for your own evals

---

## Tools & Frameworks Used in Industry

| Purpose | Tools |
|---|---|
| Frontier model APIs | Anthropic Claude, OpenAI, Google Gemini |
| Cloud-hosted access | Amazon Bedrock, Google Vertex AI, Azure AI Foundry |
| Multi-provider routing | [LiteLLM](https://docs.litellm.ai/), [OpenRouter](https://openrouter.ai/) |
| Local / open-weights | [Ollama](https://ollama.com/), [llama.cpp](https://github.com/ggml-org/llama.cpp), [LM Studio](https://lmstudio.ai/) |
| Model hub & libraries | [Hugging Face Hub](https://huggingface.co/models), `transformers`, `tokenizers` |
| Tokenizers | `tiktoken` (OpenAI), HF `tokenizers`, provider token-count endpoints |
| Learning by building | [nanoGPT](https://github.com/karpathy/nanoGPT), [minGPT](https://github.com/karpathy/minGPT) |

---

## Hands-On Project: Model Bake-Off Harness

**Build a CLI that answers "which model should we use for this task?" with a table instead of an opinion.**

Requirements:
1. Define one narrow task with 25–50 inputs and known-good outputs (e.g. extract `{company, amount, due_date}` from invoice text).
2. Run the task across ≥4 models spanning tiers — one frontier, one mid, one small/fast, one open-weights via Ollama.
3. For each run record: exact-match or field-level accuracy, input tokens, output tokens, **computed USD cost**, TTFT, and total latency.
4. Print a markdown table sorted by cost-per-correct-answer, and report p50/p95 latency, not the mean.
5. Sweep `temperature` ∈ {0, 0.7} and show the variance across 3 repeated runs at each setting.

**Done when:** you can point at the table and defend a model choice on cost-per-correct-answer, and you can explain why the cheapest model isn't always the winner.

**Stretch:** add a reasoning model with a thinking budget and show where the extra tokens do and don't buy accuracy.

---

## Common Pitfalls

- **Estimating cost from characters.** Always tokenize. Code and JSON run ~2–3 chars/token; some non-English text is far worse.
- **Pricing only input tokens.** Output is typically 3–5× the input rate, and reasoning tokens are billed as output even when hidden.
- **Confusing context window with max output.** A 200K-context model may cap output at a small fraction of that. Truncated JSON is almost always this.
- **Expecting determinism at `temperature=0`.** Greedy decoding is not bit-reproducible across batching and hardware. Design for it; don't assert exact string equality in tests.
- **Tuning `temperature` and `top_p` together.** Pick one. Changing both makes results uninterpretable.
- **Assuming a bigger model is better for your task.** On narrow, well-specified tasks small models frequently tie frontier models at a fraction of the cost.
- **Trusting public leaderboards as production signal.** They're a prior for which models to *test*, not evidence about your data.
- **Ignoring that the API is stateless.** Every turn resends the full history — that's why conversation cost grows quadratically without compaction.

---

## Progress

- [ ] 1.1 Transformers and the attention mechanism
- [ ] 1.2 Tokenization and the context window
- [ ] 1.3 The training lifecycle: pretraining → SFT → preference tuning
- [ ] 1.4 Inference and decoding: sampling params, KV cache, streaming
- [ ] 1.5 Model selection, scaling laws, and reasoning models
- [ ] **Project:** model bake-off harness

---

[← Back to root](../README.md) · [Next: 02 · Prompting & Context Engineering →](../02-prompting-and-context-engineering/README.md)
