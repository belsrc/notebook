---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-03-29
reference:
  - https://lilianweng.github.io/posts/2023-06-23-agent/
  - https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf
  - https://arxiv.org/abs/2012.14913
  - https://arxiv.org/abs/2106.09685
  - https://proceedings.neurips.cc/paper/2020/hash/6b493230205f780e1bc26945df7481e5-Abstract.html
  - https://arxiv.org/abs/2304.03442
  - https://platform.claude.com/docs/en/build-with-claude/prompt-caching
---
## Why Does Memory Matter?

A large language model in isolation is stateless. Feed it a prompt, it produces tokens, the computation ends. That works fine for a one-shot Q&A tool, but an *agent* (a system that takes sequential actions toward a goal over time) has fundamentally different requirements.

Think about a human doing research over several days. They:

- Remember what they read yesterday (long-term retention)
- Hold the current paragraph in mind while reading (working memory)
- Have intuitions built up over years of study (implicit knowledge)
- Write notes to externalize what their brain cannot hold (external storage)

An AI agent needs analogues to all of these. Lilian Weng's 2023 survey identified four canonical memory types that most agent frameworks now build around.

## The Four Memory Types

```
┌─────────────────────────────────────────────────────────────────┐
│                      AI AGENT MEMORY                            │
│                                                                 │
│  ┌──────────────────┐    ┌──────────────────────────────────┐   │
│  │  IN-WEIGHTS      │    │  IN-CONTEXT (Working Memory)     │   │
│  │  (Parametric)    │    │                                  │   │
│  │                  │    │  [ System ] [ History ] [ Docs ] │   │
│  │  Baked into      │    │  <------- Context Window ------> │   │
│  │  model params    │    │                                  │   │
│  └──────────────────┘    └──────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────┐    ┌──────────────────────────────────┐   │
│  │  IN-CACHE        │    │  EXTERNAL                        │   │
│  │  (KV Cache)      │    │                                  │   │
│  │                  │    │  [ Vector DB ]  [ Key-Value ]    │   │
│  │  Saved attention │    │  [ Relational ] [ Document ]     │   │
│  │  computation     │    │                                  │   │
│  └──────────────────┘    └──────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## In-Weights Memory (Parametric)

### What is it?

Knowledge encoded directly into a model's parameters: the billions of floating-point numbers that define the neural network's behavior. It is the implicit long-term memory of the model, roughly analogous to the procedural and semantic memory a human builds up over years of experience.

### Why does it exist?

Training a transformer on a large corpus pushes parameter values, via gradient descent, toward configurations that minimize prediction loss across the training distribution. Facts, reasoning patterns, language structure, and domain knowledge end up distributed across the weight matrix entries. There is no single "Paris is the capital of France" neuron. The knowledge emerges from the interaction of millions of parameters.

### How does it work?

The feedforward sublayers of each transformer block act as key-value memories:

$$\text{FFN}(x) = W_2 \cdot \sigma(W_1 x)$$

The rows of $W_1$ act as *key* patterns; the corresponding rows of $W_2$ act as *value* projections. When an input activates a key, the value is retrieved and added to the residual stream. This can be viewed as retrieval-like computation implemented through matrix operations.

Fine-tuning adjusts these weights to update or extend this implicit store. LoRA allows targeted modification without a full retraining run.

### Characteristics

| Property | Value |
|---|---|
| Capacity | Vast (entire training corpus, compressed) |
| Update cost | Extremely high (full or fine-tune training run) |
| Read latency | Zero (always present in forward pass) |
| Precision | Low (facts can be wrong, hallucinated, or stale) |
| Persistence | Permanent until retrained |

### When does it fail?

In-weights memory cannot be updated at runtime. An agent relying only on this will not know about events after its training cut-off, will confabulate with false confidence, and has no access to user-specific or proprietary data. That is what motivates the other three types.

## In-Context Memory (Working Memory)

### What is it?

Everything currently present in the model's context window: the system prompt, conversation history, retrieved documents, tool results, and scratchpad output. This is the agent's working memory. Immediately accessible, but finite and ephemeral.

```
┌────────────────────────────────────────────────────────────┐
│                    CONTEXT WINDOW                          │
│                                                            │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │ SYSTEM      │  │ CONVERSATION │  │ RETRIEVED CONTEXT │  │
│  │ PROMPT      │  │ HISTORY      │  │ (docs, tool data) │  │
│  │             │  │              │  │                   │  │
│  │ ~500 tokens │  │ ~2k-20k tok  │  │ ~1k-50k tokens    │  │
│  └─────────────┘  └──────────────┘  └───────────────────┘  │
│                                                            │
│  Total budget: 8k - 2M tokens depending on model           │
└────────────────────────────────────────────────────────────┘
```

### Why does it exist?

The context window is the only information directly visible to the model during inference. All reasoning, tool use, and response generation happens inside this space. Agents have to structure it deliberately, filling it with exactly the information needed for the current step.

### How is it managed?

Context management is a central engineering concern for agent systems. Naive accumulation fills the window and forces truncation of early, potentially critical content. Common strategies:

**Summarisation:** Replace raw history with compressed summaries as the window fills.

**Sliding window:** Keep only the last $N$ turns verbatim.

**RAG injection:** Rather than keeping all documents in-context permanently, retrieve only the relevant chunks at query time (covered under External Memory below).

### Characteristics

| Property     | Value                                                           |
| ------------ | --------------------------------------------------------------- |
| Capacity     | 8k-2M tokens (model-dependent, finite)                          |
| Update cost  | Zero: write tokens into the window                              |
| Read latency | Zero (full attention over all tokens)                           |
| Precision    | High: content is explicit, though interpretation can still fail |
| Persistence  | Ephemeral: lost when the session ends                           |

### The key insight

In-context memory trades **capacity** for **precision**. What is in the window, the model sees exactly. But the window is finite and computationally expensive. Every token in context costs compute in the attention layers ($O(n^2)$ in naive attention).

## External Memory

### What is it?

A persistent store outside the model, a database the agent reads from and writes to via tools. This removes the capacity ceiling and allows memory to persist across sessions and be shared across multiple agent instances.

```
                       ┌─────────────────┐
                       │   AGENT (LLM)   │
                       └────────┬────────┘
                read / write    │    read / write
          ┌─────────────────────┴──────────────────────┐
          │                                            │
          v                                            v
  ┌───────────────────┐                    ┌──────────────────────┐
  │   VECTOR STORE    │                    │  STRUCTURED STORES   │
  │                   │                    │                      │
  │  Embedding-based  │                    │  Key-Value (Redis)   │
  │  semantic search  │                    │  Relational (SQL)    │
  │                   │                    │  Document (Mongo)    │
  │ pgvector, Chroma, │                    │  Graph (Neo4j)       │
  │ Pinecone, Weaviate│                    │                      │
  └───────────────────┘                    └──────────────────────┘
```

### Sub-types

#### Semantic / Vector Memory

Documents are embedded into a high-dimensional vector space. Retrieval finds the stored chunks closest in embedding space to the query embedding, using approximate nearest-neighbor search.

$$\text{similarity}(q, d) = \frac{q \cdot d}{\|q\| \cdot \|d\|}$$

This enables fuzzy, semantic retrieval. The query "how do I undo a commit?" will retrieve a chunk about `git reset` even if neither word "undo" nor "commit" appears together, because their embeddings land nearby geometrically.

This is the backbone of Retrieval-Augmented Generation.

#### Episodic Memory (Structured / Key-Value)

Explicit records of past events, interactions, or agent states, retrieved by exact lookup, time-range query, or structured filter. Some examples:

- A key-value store mapping `session_id` to a conversation summary
- A relational table logging every tool call with timestamps
- An entity store mapping `user_id` to user preferences

This is the closest analogue to human episodic memory: the memory of specific events.

Park et al.'s Generative Agents paper stored each observation with a timestamp and retrieved using a weighted combination of recency, importance, and relevance:

$$\text{score}(m) = \alpha_{\text{recency}} \cdot \text{recency}(m) + \alpha_{\text{importance}} \cdot \text{importance}(m) + \alpha_{\text{relevance}} \cdot \text{relevance}(m, q)$$

#### Graph Memory

Entities and their relationships stored as nodes and edges. This supports structured reasoning over relational knowledge that flat vector search handles poorly.

```
  [Bryan] --works_at--> [Hypergiant]
     |                       |
   knows                  produces
     |                       |
     v                       v
  [Deck.gl] <--uses---- [Ordo.gl]
```

### Characteristics

| Property | Value                                                                        |
| ------------ | ---------------------------------------------------------------------------- |
| Capacity | Unbounded (limited by storage infrastructure)                                |
| Update cost | Low: single write per record                                                 |
| Read latency | Medium: network round-trip plus search time                                  |
| Precision | Retrieval-dependent; exact lookup is precise, semantic search is approximate |
| Persistence | Durable across sessions, shareable across agents                             |

## In-Cache Memory (KV Cache)

### What is it?

This is the most architecturally specific of the four types. During transformer inference, computing attention requires materializing the **key** ($K$) and **value** ($V$) matrices for every token in the sequence. For long static prefixes (a large system prompt repeated on every request, for instance), recomputing these projections on every call is wasteful.

The KV cache saves the $K$ and $V$ tensors for a prefix so they can be reused across multiple inference calls without recomputation.

### The mechanism

In a transformer attention layer:

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$$

If the first $n$ tokens are a fixed prefix, their $K$ and $V$ projections are constant:

$$K_{\text{prefix}} = W_K \cdot X_{\text{prefix}}, \quad V_{\text{prefix}} = W_V \cdot X_{\text{prefix}}$$

On each new call, only the suffix tokens need to be projected. This is prompt caching, offered by Anthropic, OpenAI, and others as a latency and cost reduction feature.

```
WITHOUT CACHE:
  Request 1: [System Prompt 2048 tok][User turn] -> compute ALL tokens
  Request 2: [System Prompt 2048 tok][User turn] -> compute ALL tokens again
                                                          ^ wasteful

WITH KV CACHE:
  Request 1: [System Prompt 2048 tok][User turn] -> compute all, cache prefix
  Request 2:  xxxxxxxx (cached) xxxxxxxx [User turn] -> compute only new tokens
                                                          ^ ~80% cost reduction
```

### Why does it matter for agents?

Long-horizon agent loops often repeat the same large context (tool schemas, long system prompts, accumulated documents) across many inference calls. Without caching this is prohibitively expensive. Anthropic's prompt caching requires the cached prefix to be at least 1024 tokens, placed at the start of the prompt in a stable position.

### Characteristics

| Property | Value |
|---|---|
| Capacity | Bounded by GPU VRAM; large prefixes only |
| Update cost | Computed once, reused until evicted |
| Read latency | Near-zero (in-memory on inference hardware) |
| Precision | Perfect: exact tensor reuse |
| Persistence | Ephemeral: typically minutes to hours, server-side |

## How the Four Types Compose

Real agent systems combine all four. Here is how they interact in a RAG-enabled agent with caching:

```
┌──────────────────────────────────────────────────────────────────────┐
│  AGENT LOOP                                                          │
│                                                                      │
│  1. USER QUERY ARRIVES                                               │
│                                                                      │
│  2. EXTERNAL MEMORY: semantic search over vector store               │
│     -> top-k chunks retrieved                                        │
│                                                                      │
│  3. IN-CONTEXT MEMORY: assemble context window                       │
│     [ System Prompt ][ Retrieved Chunks ][ History ][ Query ]        │
│          ^                                                           │
│  4. IN-CACHE: KV cache hits on stable system prompt prefix           │
│     -> prefix tokens not recomputed                                  │
│                                                                      │
│  5. IN-WEIGHTS: LLM runs inference                                   │
│     -> implicit world knowledge plus reasoning applied               │
│                                                                      │
│  6. AGENT TAKES ACTION (tool call, response, etc.)                   │
│                                                                      │
│  7. EXTERNAL MEMORY: write new observations back to store            │
│     -> persist for future sessions                                   │
│                                                                      │
│  8. LOOP                                                             │
└──────────────────────────────────────────────────────────────────────┘
```

### Comparative summary

| Dimension        | In-Weights                  | In-Context                                  | External            | In-Cache                       |
| ---------------- | --------------------------- | ------------------------------------------- | ------------------- | ------------------------------ |
| Capacity         | Vast (implicit)             | Finite (tokens)                             | Unbounded           | Medium (VRAM)                  |
| Precision        | Low (hallucination risk)    | High, but interpretation can still fail     | Retrieval-dependent | Perfect                        |
| Update speed     | Slow, requiring training    | Instant                                     | Milliseconds        | Fast after initial computation |
| Persistence      | Persistent across inference | Session only                                | Durable             | Ephemeral                      |
| Update frequency | Rarely                      | Every token                                 | Every event         | Per unique prefix              |
| Cost to read     | Zero                        | $O(n^2)$ attention in naive implementations | Network plus search | Near-zero                      |

## The Trade-off

Every memory architecture decision comes down to three competing properties:

$$\text{Memory} \sim \text{Trade-off}(\underbrace{\text{Capacity}}_{\text{how much?}},\; \underbrace{\text{Latency}}_{\text{how fast?}},\; \underbrace{\text{Precision}}_{\text{how accurate?}})$$

In-weights memory maximizes capacity and read latency at the cost of precision and updateability. In-context maximizes precision at the cost of capacity. External memory maximizes capacity and persistence, but introduces retrieval and system latency trade-offs. In-cache is an optimization layer that reduces the latency cost of large in-context prefixes without changing what information is available.

Knowing which type to reach for (and when to combine them) is the practical skill in agent architecture design.
