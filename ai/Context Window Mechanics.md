---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-03-17
reference:
  - https://arxiv.org/abs/1508.07909
  - https://arxiv.org/abs/1706.03762
  - https://platform.openai.com/docs/guides/error-codes
  - https://arxiv.org/abs/2307.03172
---
## Tokens

A **token** is the atomic unit of text a language model processes. Not a character, word, or sentence, but a subword unit produced by a tokenization algorithm.

The dominant algorithm is **Byte Pair Encoding (BPE)** and adopted by GPT and Claude-family models. BPE starts from a character-level vocabulary and iteratively merges the most frequent adjacent symbol pairs until a target vocabulary size is reached.

A rough heuristic (OpenAI's stated estimate, broadly applicable):

$$1 \text{ token} \approx 4 \text{ characters (English text)}$$
$$1 \text{ token} \approx 0.75 \text{ words (English prose)}$$
$$100 \text{ tokens} \approx 75 \text{ words}$$

This is a statistical average, not a law. Code, JSON, and non-Latin scripts tokenize very differently:

| Content Type | Tokens per Word (approx.) |
|---|---|
| English prose | 1.3 |
| Python/TypeScript code | 1.5-2.5 |
| JSON (with whitespace) | 2-4 |
| Chinese/Japanese | 2-4 (one character ≈ 2 tokens) |
| Whitespace/indentation | Often 1 token per indent level |

## The Context Window

The **context window** (also: context length, token budget) is the maximum number of tokens a transformer model can process simultaneously in a single pass. It is a hard limit baked into the model at training time and cannot be changed at inference.

| Model | Context Limit (tokens) |
|---|---|
| GPT-3 (2020) | 2,048 |
| GPT-4 (2023) | 8,192 - 128,000 |
| Claude 3 Haiku | 200,000 |
| Claude 3.5 Sonnet | 200,000 |
| Gemini 1.5 Pro | 1,000,000 |

### Why There's a Hard Limit

The constraint comes from the **self-attention mechanism** in transformers. For each token, attention asks three questions: "what am I looking for?", "what do I contain?", and "what do I return if selected?" Then scores every token against every other token to decide what to pay attention to.

To avoid recomputing those scores from scratch on every generation step, the model caches the intermediate results for every token it has already seen. This cache, called the **KV cache**, is the primary memory consumer during inference, and it grows with every token added to the context.

The problem is that attention is not just expensive per token, it is expensive per *pair* of tokens. A context of 1,000 tokens requires roughly 1,000,000 comparisons. Double the context to 2,000 tokens and you need 4,000,000 comparisons, four times as many, not two. This quadratic relationship between context length and memory means there is a physical ceiling: the point at which the KV cache no longer fits in GPU memory. That ceiling is the context limit.

Techniques like sliding window attention (Mistral) and state-space models (Mamba) exist to break this quadratic relationship, but standard GPT and Claude-family models are subject to it.

## Anatomy of the Context Window

The context window isn't a homogeneous blob. It's partitioned into distinct logical regions, consumed in this order:

```
┌────────────────────────────────────────────────────────────────┐
│                    CONTEXT WINDOW  (hard limit)                │
├────────────────┬───────────────┬───────────────────┬───────────┤
│  System Prompt │  Chat History │   Tool Outputs    │  Output   │
│   (static)     │  (grows each  │ (injected inline) │ (reserve) │
│                │    turn)      │                   │           │
│  ~500-5,000 T  │  ~grows...    │  0-50,000+ T      │ up to     │
│                │               │                   │  max_tok  │
└────────────────┴───────────────┴───────────────────┴───────────┘
```

**System Prompt (Static).** Instructions set by the operator. In Claude.ai this is large, Anthropic injects personality, safety instructions, tool manifests, and memory contents. For API users, this is whatever you pass as `role: "system"`. Typically 500-8,000 tokens in production.

**Conversation History (Dynamic, grows monotonically).** Every user turn and every assistant turn appended verbatim. This is the primary consumption driver. Each exchange costs tokens for the user message plus tokens for the assistant response, and that sum accumulates across every turn in the conversation without any automatic compression.

**Tool / Function Call Outputs (Bursty).** When a model uses tools (code execution, web search, file reads), the raw output is injected back into context. A single web search result can return 5,000-15,000 tokens. A "summarize this PDF" task might consume 80,000 tokens in one turn.

**Output Reserve.** The model needs room to generate. The `max_tokens` parameter reserves this space upfront. A 200,000-token window with `max_tokens` set to 8,192 leaves roughly 191,800 tokens available for input.

## How It Fills

```
Turn 1:
┌────────────┬──────┬────────┐
│ sys_prompt │ u_1  │ a_1    │
└────────────┴──────┴────────┘
             ▲ consumed: system prompt + first exchange

Turn 2:
┌────────────┬──────┬────────┬──────┬────────┐
│ sys_prompt │ u_1  │ a_1    │ u_2  │ a_2    │
└────────────┴──────┴────────┴──────┴────────┘
             ▲ previous turns now immovable; window grows right

Turn N:
┌────────────┬──────┬────────┬─ ... ─┬──────┬────────┬──────────┐
│ sys_prompt │ u_1  │ a_1    │  ...  │ u_N  │ a_N    │ [reserve]│
└────────────┴──────┴────────┴─ ... ─┴──────┴────────┴──────────┘
                                              ↑ approaching limit
```

Context is append-only. Earlier turns cannot be removed selectively without breaking conversation coherence. The decision of what gets trimmed belongs to the **serving infrastructure**, not the model itself.

A typical conversational turn:
- User message: ~50 tokens
- Assistant response: ~300 tokens
- Net growth per turn: ~350 tokens

At 350 tokens per turn with a 200,000-token window (minus 5,000 for the system prompt), you have roughly 557 turns before exhaustion. That sounds like a lot until you factor in tool use. A single document analysis can consume 80,000 tokens in one shot.

## What Happens at the Limit

Behavior at the limit depends entirely on the serving infrastructure. There's no single universal behavior.

### Hard Rejection

The API returns an error before processing:

```
Error: prompt is too long: 203,847 tokens > 200,000 max
```

The model never runs. This is the safest outcome, you get an explicit signal rather than silent degradation. OpenAI's API does this.

### Truncation

When the serving layer truncates automatically, it must decide which tokens to drop.

**Left truncation (FIFO):** drop the oldest tokens first. This preserves the most recent context, which is usually most relevant for the next response.

```
Before:
┌──────────┬─────┬─────┬─────┬─────┬─────┬───────┐
│ sys_prom │ u_1 │ a_1 │ u_2 │ a_2 │ u_3 │ [new] │
└──────────┴─────┴─────┴─────┴─────┴─────┴───────┘

After left truncation (limit exceeded):
┌──────────┬─────┬─────┬─────┬───────┐
│ sys_prom │ u_2 │ a_2 │ u_3 │ [new] │  ← u_1, a_1 silently gone
└──────────┴─────┴─────┴─────┴───────┘
```

The system prompt is typically pinned, losing it causes immediate behavioral collapse (the model no longer has its persona or instructions).

**Middle truncation:** drop tokens from the middle, preserving both the system prompt and the most recent turns:

```
┌──────────┬─────┬─────┬─ [DROPPED] ─┬─────┬─────┬───────┐
│ sys_prom │ u_1 │ a_1 │   ...       │ u_N │ a_N │ [new] │
└──────────┴─────┴─────┴─────────────┴─────┴─────┴───────┘
```

Some chat clients (including Claude.ai) use this because the opening context often contains critical framing worth preserving.

### Sliding Window

The serving layer maintains a rolling buffer over the most recent tokens at the limit, continuously evicting the oldest as new ones arrive. Computationally equivalent to continuous left truncation. Common in long-running agent deployments.

## Degradation Modes

When truncation has occurred, silently or otherwise, the model's output degrades in predictable ways.

**Referential amnesia.** The model references earlier conversation state as if it no longer exists, because it doesn't.

```
Turn 1:  User: "My name is Bryan."
Turn 2:  Assistant: "Got it, Bryan."
...
[40 turns of heavy tool use later — turns 1-2 truncated]
...
Turn 45: User: "What's my name again?"
          Assistant: "I don't have that information in our conversation."
```

The model isn't malfunctioning. It's correctly reporting on the context it actually has. The failure is silent data loss upstream.

**Instruction drift.** If the system prompt or early framing turns are truncated, behavioral constraints disappear. A model told to "respond only in JSON" reverts to prose. A coding assistant told to "use TypeScript with explicit types" produces `any`-typed JavaScript. This is why production systems pin the system prompt against truncation.

**Hallucinated reconstruction.** The model sees a reference to something that was defined in the dropped tokens:

```
Surviving context:
  "...as we established with the DataFilterExtension approach..."

Dropped context:
  [The entire discussion of DataFilterExtension]
```

The model will synthesize a plausible-but-wrong reconstruction from its training data, not from your actual architectural decision. It won't flag the uncertainty.

**Attention dilution.** When the context is very full but not yet truncated, the model's attention is spread across an enormous token sequence. Recency bias in positional encodings means the most recent tokens receive the strongest attention, but distant tokens get diluted weights. The result: repeated information the model already gave, and lower-quality reasoning. The model isn't wrong, just less sharp. This is the hardest degradation to spot because there's no obvious failure, just a subtle drop in output quality.

## Lost in the Middle

Liu et al. demonstrated empirically that transformer models attend more strongly to tokens at the beginning and end of the context window, with a measurable performance trough for tokens in the middle.

```
Attention
strength
   │
 ▲ │
   │ █                                   █
   │ █ █                             █ █
   │ █ █ █                       █ █ █
   │ █ █ █ █ █ █ █ █ █ █ █ █ █ █ █
   └─────────────────────────────────────▶
     start            middle            end
     of context                      of context
```

Information placed in the middle of a large context is statistically less reliable than information at the start or end. Not a bug, an artifact of how positional encodings and attention patterns interact across long sequences. Critical information (key constraints, core instructions, primary examples) should go at the start or immediately before the query.

## Full Picture

```
Text Input
    │
    ▼ tokenization (BPE)
Token Sequence  [t₁, t₂, ..., tₙ]
    │
    ▼ context assembly
┌───────────────────────────────────────────────────────────────┐
│  [sys_prompt] [turn_1] [turn_2] ... [turn_k] [current_input]  │
│  ◄──────────────────── hard limit ───────────────────────────►│
└───────────────────────────────────────────────────────────────┘
    │                              │
    │ fits within limit            │ exceeds limit
    ▼                              ▼
  Forward pass              Truncation strategy applied
  (attention over                │
   all tokens)                   ├─ Left truncation (oldest dropped)
    │                            ├─ Middle truncation
    ▼                            ├─ Hard rejection (API error)
  Full output quality            └─ Sliding window
                                      │
                                      ▼
                                 Degraded output:
                                 - Referential amnesia
                                 - Instruction drift
                                 - Hallucinated reconstruction
                                 - Attention dilution / repetition
                                 - Lost-in-the-middle failures
```

## Summary

| Concept | Core Fact |
|---|---|
| Token | Subword unit from BPE; ~4 characters per token in English |
| Context window | Hard maximum tokens per forward pass; quadratic memory cost |
| Consumption | Append-only: system prompt + history + tool outputs + output reserve |
| Truncation | Infrastructure decision, not model decision; system prompt typically pinned |
| Degradation modes | Amnesia, instruction drift, hallucinated reconstruction, attention dilution |
| Lost in the middle | Models attend less reliably to context in the middle of long windows (Liu et al., 2023) |

The context window is finite working memory. Everything that matters for the current task has to fit inside it. When it overflows, the loss is silent, the degradation is structural, and recovery requires either a fresh context or external memory strategies to compress and rehydrate the relevant state.
