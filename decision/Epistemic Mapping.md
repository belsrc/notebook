---
tags:
  - decision
gardening: 🌱
date: 2026-05-17
reference:
  - https://plato.stanford.edu/entries/episteme-techne/
  - https://www.dni.gov/files/documents/ICD/ICD-203.pdf
  - https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions
  - https://en.wikipedia.org/wiki/Johari_window
  - https://pmc.ncbi.nlm.nih.gov/articles/PMC8918592/
  - https://bayes.wustl.edu/etj/prob/book.pdf
---
## What is *episteme*?

The word **epistêmê** (ἐπιστήμη) is the Greek term most often translated as knowledge, while **technê** is translated as craft or art. In ancient Greek philosophy, these terms are related but not identical, and their meanings vary across Plato, Xenophon, and Aristotle. In Aristotle's later account, *epistêmê* is associated with demonstrable knowledge, while *technê* concerns craft-like productive skill.

The distinction matters:

- *epistêmê* - demonstrable or scientifically grounded knowledge. Example: "I know that this conclusion follows from the premises."
- *doxa* - opinion or belief, often contrasted with demonstrable knowledge. Example: "I think it will rain tomorrow."
- *technê* - practical skill or craft knowledge. Example: "I know how to weld."

Modern **epistemology** is the branch of philosophy concerned with the nature, sources, scope, and limits of knowledge. It asks:

- What does it mean to *know* something?
- How do we distinguish knowledge from belief?
- What are the *limits* of what can be known?

## What is a *map*?

A map is a **representation** of a domain that preserves relationships. It answers:

1. **What exists** in this space?
2. **Where** does each thing sit relative to others?
3. **What are the boundaries**, where does the space end?
4. **What is the terrain**, what regions are dense vs. sparse, navigable vs. impassable?

A map is not the territory. As Alfred Korzybski observed, a map is an abstraction of reality rather than reality itself. A map compresses and simplifies the domain so that it can be used for navigation and reasoning.

## The core concept

### Definition

**Epistemic mapping** is the practice of constructing a structured representation of a *knowledge domain* that makes explicit:

1. **What is known** - claims with strong justification
2. **What is unknown** - recognized gaps
3. **What is uncertain** - claims with weak, contested, or probabilistic justification
4. **What is unknowable** - questions outside the domain's reach
5. **The relationships** between all of the above

Think of it as charting the topology of a knowledge space.

```
┌───────────────────────────────────────────────────────┐
│              Epistemic Map (Conceptual View)          │
│                                                       │
│   ╔════════════╗     ╔══════════════╗                 │
│   ║   KNOWN    ║────▶║  Inferences  ║                 │
│   ║ (justified)║     ║ (derived)    ║                 │
│   ╚════════════╝     ╚══════════════╝                 │
│          │                  │                         │
│          ▼                  ▼                         │
│   ┌────────────┐     ┌──────────────┐                 │
│   │  UNKNOWN   │     │  UNCERTAIN   │                 │
│   │  (gaps)    │     │  (contested) │                 │
│   └────────────┘     └──────────────┘                 │
│          │                                            │
│          ▼                                            │
│   ░░░░░░░░░░░░░░░░░░                                  │
│   ░  UNKNOWABLE    ░  <- outside the domain boundary  │
│   ░░░░░░░░░░░░░░░░░░                                  │
└───────────────────────────────────────────────────────┘
```

### The problem it solves

Why does this concept exist? What breaks without it?

Consider a research team working on a complex system, say, a distributed database. Each engineer carries a private epistemic state: what they personally know, believe, and are uncertain about. These private states are:

- **Inconsistent** - Engineer A believes the write path is correct; Engineer B believes it has a race condition
- **Non-overlapping** - Engineer C has domain knowledge the others lack
- **Uncharted** - No one has noticed the entire cache invalidation layer hasn't been analyzed

Without epistemic mapping, the team operates with shared uncertainty but without shared structure:

- $A$ knows $X$ --- doesn't know $B$ knows $\neg X$
- $B$ knows $\neg X$ --- doesn't know gap in $Z$ exists
- $C$ knows $Z$ --- doesn't know $A$ and $B$ conflict

Result: Conflicting assumptions, duplicated work, unnoticed gaps, false confidence.

Epistemic mapping externalizes and structures these private states into a shared artifact that makes uncertainty easier to inspect and manage. This is especially useful in complex planning and analysis, where missing assumptions and hidden disagreements can strongly affect outcomes

### Anatomy of an epistemic map

A well-formed epistemic map has five structural elements:

**1. Nodes** - individual knowledge claims or questions.

Each node has a *justification type*:

| Justification | Description                              |
| ------------- | ---------------------------------------- |
| Empirical     | Derived from observation/experiment      |
| Inferential   | Derived by logical deduction from others |
| Axiomatic     | Taken as foundational without proof      |
| Testimonial   | Accepted from a trusted source           |
| Probabilistic | Held with a degree of confidence `[0,1]` |

**2. Edges** - relationships between nodes.

| Relationship   | Meaning                                |
| -------------- | -------------------------------------- |
| supports       | $A$ increases confidence in $B$        |
| contradicts    | $A$ decreases confidence in $B$        |
| presupposes    | $B$ cannot be evaluated without $A$    |
| implies        | $A \rightarrow B$ (logical entailment) |
| is-independent | $A$ has no bearing on $B$              |

**3. Confidence values** - numerical or ordinal assignment.

A Bayesian framing gives this rigor. Each claim $C$ carries a *prior* probability $P(C)$, updated by evidence $E$ via Bayes' theorem:

$$P(C \mid E) = \frac{P(E \mid C) \cdot P(C)}{P(E)}$$

Epistemic mapping makes these confidence values explicit rather than leaving them implicit in an agent's head.

**4. Boundaries** - the edges of the domain

Marking *what the map does not cover* is as important as marking what it does. Three boundary types:

| Boundary Type   | Example                                  |
| --------------- | ---------------------------------------- |
| Known unknown   | "We haven't measured latency under load" |
| Unknown unknown | (By definition, can't be listed)         |
| Out-of-scope    | "Network hardware is outside our model"  |

You can also think of this as the boundary between current knowledge and the limits of the current model.

> *"There are known knowns... known unknowns... and unknown unknowns."*
> -- Donald Rumsfeld, 2002 (adapted from Luft & Ingham's Johari Window, 1955)

**5. Provenance** - where each claim comes from

Every node should be traceable to its source: a measurement, a document, an inference, an assumption. This is what separates a rigorous epistemic map from an ordinary brainstorm. ODNI's analytic standards explicitly require analysts to describe source quality, explain uncertainty, distinguish assumptions from judgments, and identify information gaps.

## The Johari Window

Before moving to more formal models, it helps to look at a simpler conceptual predecessor.

The **Johari Window** was designed for interpersonal awareness, but it usefully illustrates the idea of explicit and hidden knowledge.

|                       | Known to Self                    | Unknown to Self                 |
| --------------------- | -------------------------------- | ------------------------------- |
| **Known to Others**   | Open (shared known)              | Blind (unknown unknown to self) |
| **Unknown to Others** | Hidden (known unknown to others) | Unknown (not yet accessible)    |

Epistemic mapping expands this from a 2x2 grid about *self and others* to a full graph about a *knowledge domain*. The same quadrant logic applies:

- Expand the Open quadrant by making more knowledge explicit and shared.
- Shrink the Blind quadrant by surfacing blind spots through review.
- Audit the Hidden quadrant by making assumptions and private knowledge visible.
- Treat the Unknown quadrant as a prompt for inquiry rather than a category to ignore.

## Formal representation

### As a directed graph

Formally, an epistemic map $\mathcal{M}$ can be modeled as a directed labeled graph:

$$\mathcal{M} = (N, E, \sigma, \rho)$$

Where:

- $N$ = a finite set of nodes, representing propositions or questions.
- $E \subseteq N \times N$ = a set of directed edges representing epistemic relationships.
- $\sigma : N \to [0, 1]$ = a confidence function assigning each node a probability value.
- $\rho : N \to \mathcal{S}$ = a provenance function mapping each node to a source set $\mathcal{S}$.

A **known** node $n_i$ satisfies $\sigma(n_i) \geq \theta_k$ for some threshold $\theta_k$.

An **unknown** node $n_j$ is a represented question whose confidence is undefined, unavailable, or highly unstable.

A gap is an unarticulated question: a node that should exist in the graph but has not yet been identified. That distinction matters because missing questions are often more important than missing answers.

### Confidence propagation

The edges in the map can influence confidence. If node $A$ supports node $B$ with weight $w_{AB}$, then a simplified update rule might be:

$$
\sigma'(B) = \sigma(B) + w_{AB} \cdot \sigma(A) \cdot (1 - \sigma(B))
$$

This is a heuristic update rule, not a general theorem of epistemic mapping. In a full Bayesian model, you would use conditional probabilities or a graphical model. The point is that confidence should be revisable as evidence changes.

This has a practical consequence for software systems:

```typescript
type EpistemicNode = {
  id: string;
  claim: string;
  confidence: number;         // [0.0, 1.0]
  justification: JustificationType;
  provenance: string[];       // source references
};

type EpistemicEdge = {
  from: string;               // node id
  to: string;                 // node id
  relation: RelationType;
  weight: number;             // [-1.0, 1.0] -- negative = contradicts
};

type EpistemicMap = {
  nodes: ReadonlyArray<EpistemicNode>;
  edges: ReadonlyArray<EpistemicEdge>;
  domain: string;
  boundaryNotes: ReadonlyArray<string>;  // known unknowns, out-of-scope
};
```

## Where it appears

This is not a purely philosophical concept. It shows up in several domains familiar to engineers.

### Software architecture

An **Architecture Decision Record** is a lightweight epistemic map of a design decision. It makes explicit:

- What is *known* about the system's constraints
- What *assumptions* are being made (axiomatic nodes)
- What is *uncertain* about the options
- What is *out of scope* for the decision

Without this framing, ADRs often record what was decided but omit what was not known at the time.

### Intelligence analysis

ODNI's analytic standards require analysts to describe source quality, express uncertainty, distinguish assumptions from judgments, identify knowledge gaps, and support claims with clear reasoning. The directive also specifies confidence and likelihood language for analytic products.

The Intelligence Community confidence framework uses structured likelihood language such as "remote", "unlikely", "roughly even", "likely", "almost certain", and similar standardized terms.

### AI and LLM systems

Large language models have a structural epistemic limitation: they do not natively maintain an explicit map of what they know, what they infer, what they are uncertain about, and where a claim came from. Their behavior is therefore better understood as implicit pattern-based generation than as explicit knowledge tracking.

Retrieval-augmented generation is a partial remedy because it externalizes some knowledge into a retrievable corpus with provenance, but it does not by itself produce a complete epistemic map. It improves access to source material, but it does not fully solve uncertainty tracking, boundary management, or gap detection.

Systems like OpenSpec that track artifact states across phases are implicitly constructing epistemic maps: each artifact carries provenance, the phase transitions encode confidence changes, and the Research/Design boundary enforces separation of the known and unknown states before implementation begins.

## Common failure modes

Knowing when epistemic mapping breaks down is as important as knowing what it is.

| Failure             | Description                                                            |
| ------------------- | ---------------------------------------------------------------------- |
| Overconfidence bias | Confidence is assigned higher than the evidence supports.              |
| Gap blindness       | Missing questions are not recognized.Unknown unknowns not acknowledged |
| Provenance collapse | Claims exist without traceable sources.                                |
| Edge omission       | Contradictions are not surfaced as relationships.                      |
| Boundary creep      | Out-of-scope claims are treated as in-scope.                           |
| Static maps         | Confidence is not updated as evidence changes.                         |

The most dangerous failure is overconfidence bias (the Dunning-Kruger effect at the map level). A team may believe its map is complete and accurate, when in fact it contains unrecognized gaps and unstated assumptions. The best antidote is adversarial review: deliberately asking someone to look for what the map misses, not merely what it contains.

## Synthesis

A knowledge domain has structure:
- Claims exist at varying confidence levels `[0,1]`.
- Claims relate to each other through support, contradiction, or implication.
- The domain has a boundary (in-scope vs. out-of-scope).
- Gaps exist where questions have not yet been articulated or answered.

Epistemic mapping externalizes this structure into an artifact that makes it:
- Inspectable - anyone can read the map
- Debatable - claims can be challenged with evidence
- Updatable - confidence changes as evidence arrives
- Gap-visible - unknown unknowns can be sought explicitly
- Provable - every claim has a source

The map is not the territory. It is a navigation tool, and its value depends on how honestly it represents both what is known and what is not.
