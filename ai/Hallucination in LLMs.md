---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-03-18
reference:
  - https://arxiv.org/abs/2202.03629
  - https://arxiv.org/abs/2311.05232
  - https://arxiv.org/abs/2212.10511
  - https://arxiv.org/abs/2212.09251
  - https://arxiv.org/abs/2203.11171
  - https://arxiv.org/abs/2005.11401
  - https://arxiv.org/abs/2204.04991
---
## A Precise Definition

The word "hallucination" gets used loosely, so it is worth being specific about what it means in the context of language models.

In NLP, a hallucination is model-generated content that is factually incorrect, unverifiable, or inconsistent with its input or source, but expressed with high fluency and apparent confidence.

Two subtypes exist at the foundational level:

| Type                        | What It Means                                                               | Example                                                                                  |
| --------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Intrinsic hallucination** | Output directly contradicts a known source or ground truth                  | A model says "Einstein was born in 1889" (he was born in 1879)                           |
| **Extrinsic hallucination** | Output cannot be verified or refuted from any source; it is simply invented | A model cites a paper with a real-looking title, journal, and author that does not exist |

In practice, intrinsic hallucinations are easier to catch because you can compare directly against a source, while extrinsic hallucinations require external verification.


A more specific taxonomy for LLMs distinguishes two further categories:

| Category | Definition |
|---|---|
| **Factuality hallucination** | The output conflicts with established world knowledge |
| **Faithfulness hallucination** | The output conflicts with the user's instructions, the conversation context, or documents that were explicitly provided |

Faithfulness hallucinations are particularly hard to catch because the model may have been given the correct answer in its input and still contradict it.

What makes this a hallucination rather than just an error is that the model does not signal uncertainty. It delivers a false claim with the same tone, structure, and confidence as a true one.

## How Language Models Actually Work

To understand why hallucination happens, you need a clear picture of what a large language model (LLM) is actually doing when it produces text.

### The Core Mechanism: Next-Token Prediction

A base language model does not query a database or explicitly look things up; it performs one operation: predict the next most likely token given the previous tokens.

A "token" is roughly a word fragment (the word "unhappiness" might be three tokens: "un", "happi", "ness"). At each step, the model outputs a probability score over its entire vocabulary (often 50,000 to 100,000+ tokens) and selects the next one.

```
┌──────────────────────────────────────────────────────┐
│              Next-Token Prediction Loop              │
│                                                      │
│  Input tokens                                        │
│  ┌──────────────────────────────┐                    │
│  │ "The capital of France is"   │                    │
│  └──────────────┬───────────────┘                    │
│                 │                                    │
│                 ▼                                    │
│  ┌──────────────────────────────┐                    │
│  │        Transformer Model     │                    │
│  │  (billions of parameters)    │                    │
│  └──────────────┬───────────────┘                    │
│                 │                                    │
│                 ▼                                    │
│  ┌──────────────────────────────┐                    │
│  │  Probability over vocabulary │                    │
│  │  "Paris"   → 0.82            │                    │
│  │  "Lyon"    → 0.07            │                    │
│  │  "London"  → 0.04            │                    │
│  │  "the"     → 0.02            │                    │
│  │  ...                         │                    │
│  └──────────────┬───────────────┘                    │
│                 │                                    │
│                 ▼                                    │
│         Selected token: "Paris"                      │
│                                                      │
│  New input: "The capital of France is Paris"         │
│  (loop repeats)                                      │
└──────────────────────────────────────────────────────┘
```

The model does this one token at a time until it produces an end-of-sequence token or hits a length limit.

### Where the "Knowledge" Comes From

During training, the model processed enormous quantities of text (web pages, books, code, articles) and adjusted billions of numerical parameters to get better at predicting the next token in that training data. The "knowledge" a model appears to have is not stored in a lookup table. It is **distributed across all those parameters** as statistical patterns.

The model learned that the pattern "The capital of France is ___" was almost always followed by "Paris" in its training data, so the parameter weights encode that strong association. There is no stored fact labeled `{ country: "France", capital: "Paris" }`. There is pressure in a high-dimensional numerical space that makes "Paris" the high-probability continuation.

```
┌───────────────────────────────────────────────────────────────┐
│              How Humans Imagine It Works                      │
│                                                               │
│  Question ──► [ Knowledge Base ] ──► Retrieve fact ──► Answer │
│                                                               │
│              How It Actually Works                            │
│                                                               │
│  Tokens ──► [ Statistical Pattern Matcher ] ──► Next token    │
└───────────────────────────────────────────────────────────────┘
```

The model has no explicit fact-checker and no built-in concept of "I know this" vs. "I do not know this." It only has: "given this sequence, what token is most likely to follow?"

## Why Hallucination Follows Directly from This

When you ask a model about something rare or absent from its training data, it has no way to detect that gap. Consider what happens:

```
┌─────────────────────────────────────────────────────────────┐
│  Scenario: Asking about an obscure research paper           │
│                                                             │
│  User: "What did Chen et al. 2021 find about                │
│         hippocampal plasticity in aging mice?"              │
│                                                             │
│  What a retrieval system would do:                          │
│  ┌──────────────────────────────────────────────┐           │
│  │ Search index ──► paper not found ──► "I don't│           │
│  │ have that paper"                             │           │
│  └──────────────────────────────────────────────┘           │
│                                                             │
│  What next-token prediction does:                           │
│  ┌──────────────────────────────────────────────┐           │
│  │ "Chen et al. 2021 found that..." is a        │           │
│  │ plausible continuation pattern.              │           │
│  │ Fill with statistically likely neuroscience  │           │
│  │ findings from the training distribution.     │           │
│  └──────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

The model has no mechanism that says "stop, I should report uncertainty here." It continues generating the most statistically plausible tokens, which produces text that **looks exactly like a correct answer** because that is the pattern it learned to follow.

### The Specific Causes

**1. Training data gaps and long-tail knowledge**

If a topic appeared rarely in training data, the model has weak, poorly-calibrated associations for it. Rather than producing uncertainty, it fills the gap with the nearest high-probability pattern, which is often plausible-sounding but wrong.

**2. The confidence-accuracy mismatch**

The model was trained to produce fluent, complete text. Incomplete or hedged sentences were less common in training data than confident assertions. The model has been, in effect, rewarded for sounding confident, independent of whether the content is correct.

**3. Sycophancy pressure from RLHF**

Most production LLMs are fine-tuned using Reinforcement Learning from Human Feedback (RLHF), where human raters score model outputs. Studies of RLHF show that human raters often reward confident, fluent answers, which pushes models toward confident text even when uncertainty or partial answers would be more appropriate.

**4. Context window limitations and attention dilution**

For faithfulness hallucinations specifically: when the model is given a long document to reason over, attention to distant parts of the context degrades. The model may genuinely "lose track" of what it was told and fall back on its parametric knowledge, contradicting the source material.

**5. Decoding strategy choices**

The token selection step is not always deterministic. When "temperature" (a sampling parameter) is increased, the model samples from lower-probability tokens more often, increasing creativity but also increasing hallucination rate. This is a direct, operator-controllable variable.

## A Taxonomy of Hallucination Types

Having names for different forms of hallucination helps when diagnosing problems in production:

```
Hallucination
├── Factuality Hallucinations
│   ├── Entity fabrication       (inventing a person, place, or thing)
│   ├── Relation errors          (wrong relationship between real things)
│   ├── Temporal errors          (wrong date, sequence, or time period)
│   └── Numerical errors         (wrong quantities, statistics, measurements)
│
└── Faithfulness Hallucinations
    ├── Instruction deviation    (ignores constraints given in the prompt)
    ├── Context contradiction    (contradicts a document given as input)
    └── Self-contradiction       (contradicts something the model said earlier
                                  in the same conversation)
```

The most dangerous in practice is **entity fabrication**: the model invents references, citations, legal cases, product names, or individuals that sound entirely real. These pass casual inspection and require ground-truth verification to catch.

## How to Detect Hallucinations

Detection falls into three categories: human methods, automated methods, and architectural methods.

### Human Detection

The only fully reliable method for high-stakes content is **ground-truth verification**: taking each factual claim and checking it against a primary source. This is slow but gives you near-certain results.

A faster heuristic is **claim decomposition**: break the model's output into individual atomic claims, then ask "could I find a source for this specific claim?" Claims that feel familiar but cannot be sourced are candidates for hallucination.

### Automated Detection

**Self-consistency sampling**:

Run the same query multiple times (with some variation in phrasing or temperature) and compare outputs. Consistent answers across runs are more likely to reflect strong training signal. Inconsistent answers signal low confidence and potential hallucination.

```
┌────────────────────────────────────────────────────────────┐
│  Self-Consistency Detection                                │
│                                                            │
│  Query (run 3 times):                                      │
│  "What year was the Eiffel Tower completed?"               │
│                                                            │
│  Run 1: "The Eiffel Tower was completed in 1889."          │
│  Run 2: "Construction finished in 1889."                   │
│  Run 3: "The tower opened in 1889."                        │
│                                                            │
│  Result: High agreement → higher confidence                │
│                                                            │
│  ─────────────────────────────────────────                 │
│                                                            │
│  Query (run 3 times):                                      │
│  "What did Dr. X argue in their 2019 paper on Y?"          │
│                                                            │
│  Run 1: "Dr. X argued that mechanism A dominates..."       │
│  Run 2: "The paper focused on mechanism B..."              │
│  Run 3: "Dr. X's central claim was about mechanism C..."   │
│                                                            │
│  Result: Low agreement → hallucination likely              │
└────────────────────────────────────────────────────────────┘
```

**Retrieval-Augmented Grounding** :

Pair the LLM with a document retrieval system (**retrieval-augmented generation, or RAG**). The model generates claims, and an automated checker asks: "is this claim supported by the retrieved source?" This converts an open-ended generation problem into a bounded entailment check, which is significantly easier.

This is the architectural basis of modern "grounded" AI systems (search-augmented chat, document Q&A tools).

**Natural Language Inference (NLI) Scoring**:

NLI models are classifiers trained to determine whether a premise *entails*, *contradicts*, or is *neutral toward* a hypothesis. You can use an NLI model to score whether a model's output is entailed by a known-good source document. High contradiction scores are a strong hallucination signal.

**LLM-as-a-judge with citations**:

Use a second LLM call to fact-check the first. Provide the output and ask: "For each factual claim, state whether it is verifiable, and what source would confirm it." The judge can also hallucinate, so this is not airtight, but it catches obvious factual errors at low cost.

### Architectural and Deployment Mitigations

Detection and mitigation are closely related. The following deployment patterns reduce hallucination rates and make remaining hallucinations easier to catch:

| Pattern | How It Helps |
|---|---|
| **Grounded generation (RAG)** | Model output is anchored to retrieved documents; contradictions are checkable |
| **Citation enforcement** | Prompt instructs the model to cite a source for every claim; unsourced claims are flagged |
| **Low temperature for factual tasks** | Reduces sampling randomness; produces more deterministic, high-probability outputs |
| **Structured output + validation** | Constrain model output to a schema; validate field values against known-good data |
| **Confidence elicitation** | Ask the model to rate its own confidence; low-confidence claims receive human review |

No single method eliminates hallucination. The practical approach for high-stakes deployments is grounded retrieval to reduce the opportunity for hallucination, followed by automated NLI or self-consistency checks, followed by human review of flagged claims.

## Summary

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Definition:  Output that is factually wrong or             │
│               unverifiable, stated with confidence          │
│                                                             │
│  Root cause:  Next-token prediction is a pattern            │
│               matcher, not a knowledge retrieval            │
│               system. It has no mechanism to                │
│               distinguish "I know this" from                │
│               "this sounds plausible."                      │
│                                                             │
│  Compounding  - Rare training data for the topic            │
│  factors:     - RLHF pressure toward confidence             │
│               - Attention dilution over long context        │
│               - Sampling temperature                        │
│                                                             │
│  Detection:   - Human: ground-truth verification            │
│               - Automated: self-consistency,                │
│                 NLI scoring, LLM-as-judge                   │
│               - Architectural: RAG, citation                │
│                 enforcement, structured outputs             │
└─────────────────────────────────────────────────────────────┘
```

**Do not treat LLM output as a primary source. Treat it as a first draft, and verify claims in proportion to the cost of being wrong.**
