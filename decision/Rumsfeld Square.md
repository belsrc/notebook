---
tags:
  - decision
gardening: 🌱
date: 2026-05-19
reference:
  - https://avalon.law.yale.edu/sept11/dod_brief146.asp
  - https://usinfo.org/wf-archive/2002/020212/epf202.htm
  - https://inthesetimes.com/article/what-rumsfeld-doesn-know-that-he-knows-about-abu-ghraib
  - https://www.communicationtheory.org/the-johari-window-model/
  - https://en.wikipedia.org/wiki/Johari_window
---
## Foundation: what is "knowledge state"?

Before the framework makes sense, we need to define what it means to know something, and more importantly, what it means to know *that* you know it.

There are two distinct cognitive acts involved whenever knowledge comes up:

1. The fact itself: is it present in your mental model?
2. Meta-awareness: are you aware of your own knowledge state about it?

These two acts are independent. You can hold information without consciously recognizing that you hold it, as with tacit expertise. You can know that a gap exists without being able to fill it. You can also be unaware that a gap exists at all.

The Rumsfeld Square makes this independence explicit.

## Origin and context

Donald Rumsfeld, then U.S. Secretary of Defense, stated the framework publicly at a Department of Defense news briefing on February 12, 2002, in response to a question about the lack of evidence linking Iraq to weapons of mass destruction. The best-known line is:

> "There are known knowns... there are known unknowns... But there are also unknown unknowns."

Related models predate Rumsfeld. Variants appear in epistemology, risk management, and organizational psychology, including the Johari Window model from 1955. Rumsfeld's contribution was making the structure widely known in strategic and policy contexts.

## The two axes

The square comes from crossing one binary dimension with itself:

**Axis 1 (columns): is the thing present in your model?**

$$K_{\text{fact}} \in \{\text{known}, \text{unknown}\}$$

**Axis 2 (rows): are you aware of your own knowledge state?**

$$K_{\text{meta}} \in \{\text{known}, \text{unknown}\}$$

Crossing these two binary dimensions produces four cells.

```
                   ┌─────────────────────────────────┐
                   │     FACT STATE                  │
                   │  Known          │  Unknown      │
         ┌─────────┼─────────────────┼───────────────┤
         │  Known  │  Known Knowns   │  Known        │
META     │         │  (KK)           │  Unknowns     │
STATE    │         │                 │  (KU)         │
         ├─────────┼─────────────────┼───────────────┤
         │ Unknown │  Unknown        │  Unknown      │
         │         │  Knowns         │  Unknowns     │
         │         │  (UK)           │  (UU)         │
         └─────────┴─────────────────┴───────────────┘
```

## The four quadrants

### 1. Known knowns (KK)

You have the information. You know you have it.

This is explicit, accessible knowledge: engineering specs, documented procedures, historical records, or laws of physics you can state clearly. The contents of this quadrant can be directly acted on and communicated.

**Example in software:** You know your API has a rate limit of 1000 requests per second, and you designed around it.

Cognitive load here is low. This is your working surface.

### 2. Known unknowns (KU)

You do not have the information. You know you do not have it.

This is identified risk and active inquiry: a question on the whiteboard, a dependency you have not benchmarked under load, or a legal opinion you know you still need.

**Example in software:** You know you have not load-tested the database at 10x traffic, so that gap is visible and tracked.

Cognitive load is manageable. You can allocate resources, ask questions, and build experiments. This quadrant feeds risk registers and spike research.

Known unknowns have a formal treatment in decision theory. If you can enumerate outcomes and assign probabilities, the problem reduces to expected-value reasoning:

$$E[X] = \sum_{i} p_i \cdot x_i$$

The problem is tractable precisely because the uncertainty is legible.

### 3. Unknown unknowns (UU)

You do not have the information. You are not aware you are missing it.

This is the dangerous quadrant. You cannot ask the question because you do not know the question exists. Your mental model has a gap that is invisible to you. This is where many catastrophic surprises originate: complex-system accidents, intelligence failures, and avoidable architectural rewrites.

**Example in software:** A third-party library has a race condition under a specific garbage-collection pressure pattern, and nobody on your team knows it exists. You discover it months after launch.

You cannot eliminate unknown unknowns by definition. The practical goal is to convert them into known unknowns, where they become tractable.

Strategies that help include:

- Diverse perspectives: other people's known knowns are your unknown unknowns.
- Red teams and external audits: challenge assumptions from outside the system.
- Black box testing: probe behavior without assuming internals.
- Pre-mortems: ask what you may have missed before committing to a plan.
- Incident archaeology: study postmortems to expand your known unknowns.

The goal is not to eliminate unknown unknowns. That is impossible. The goal is to convert them into known unknowns, at which point they become tractable.

### 4. Unknown knowns (UK)

You have the information. You are not consciously aware that you have it.

Rumsfeld did not name this quadrant. Žižek later highlighted it in a 2004 critique, arguing that it captures disavowed or tacit knowledge that can be strategically important.

This quadrant covers:

- Tacit knowledge: a senior engineer who senses an architectural problem but cannot fully articulate it.
- Institutional memory: organizational practices nobody can explain but everyone follows.
- Cognitive bias: assumptions so deeply embedded that they no longer register as assumptions.
- Repressed knowledge (Žižek's angle): beliefs or incentives that people act on while denying them explicitly.

**Example in software:** A team writes code that quietly assumes single-threaded execution across the codebase. No document records that decision, and the developers would still say the system supports concurrency. The assumption is real and consequential, but unacknowledged.

Unknown knowns are a major source of undocumented tribal knowledge, implicit architectural constraints, and organizational blind spots. Surfacing them usually requires deliberate elicitation: pair programming, documentation, onboarding, interviews, and watching where new team members stumble.

## Complete map

```
                   ┌────────────────────────────────────────────────────┐
                   │                 FACT STATE                         │
                   │    Known                      Unknown              │
         ┌─────────┼────────────────────────────┼───────────────────────┤
         │ Known   │  KNOWN KNOWNS              │  KNOWN UNKNOWNS       │
META     │         │  Explicit knowledge        │  Identified risks     │
STATE    │         │  Directly actionable       │  Active inquiries     │
         │         │  Communicated and audited  │  Research and test    │
         ├─────────┼────────────────────────────┼───────────────────────┤
         │Unknown  │  UNKNOWN KNOWNS            │  UNKNOWN UNKNOWNS     │
         │         │  Tacit knowledge           │  Blind spots          │
         │         │  Tribal memory             │  Structural surprises │
         │         │  Surfaced by elicitation   │  Reduced by review    │
         └─────────┴────────────────────────────┴───────────────────────┘
```

## Dynamics: how knowledge moves between quadrants

The quadrants are not static. Knowledge transitions between them, and these transitions are the leverage points for good epistemic practice.

```
UU  --[diverse review, red teams]-->   KU
                                        |
                                        | [research, experiment, testing]
                                        v
UK  --[elicitation, documentation]-->  KK
```

The productive loop is:

1. Convert UU to KU by deliberately seeking blind spots through external review and adversarial probing.
2. Convert KU to KK by running targeted research on known gaps.
3. Convert UK to KK by eliciting and documenting tacit knowledge.

What to avoid:

```
KK  --[time, turnover, entropy]-->  UK  (knowledge erosion)
KU  --[ignored]------------------>  UU  (gap amnesia -- worse than never knowing)
```

## Application to software engineering

| Quadrant | Practical form | Mitigation |
|---|---|---|
| KK | Specs, tests, documented constraints | Maintain, audit, keep current |
| KU | Open tickets, spike stories, open questions in PRs | Track explicitly; time-box investigation |
| UU | Novel failure modes, upstream bugs, unknown interactions | Red teams, chaos engineering, diverse review, blameless postmortems |
| UK | Undocumented conventions, tribal architecture, implicit assumptions | ADRs, onboarding documentation, pair programming |

The QRSPI methodology maps directly onto this. The Research phase specifically converts UU and KU into KK before Design begins. Committing to design while still in the KU/UU region is the structural cause of rewrites.

## Limitations of the framework

1. The quadrants are not cleanly discrete. In practice, knowledge is often partial, so you may sit between KU and KK.
2. It says nothing about the quality of known knowns. A KK quadrant can still contain false beliefs.
3. Unknown is always relative to an agent. Your UU may be someone else's KK.
4. Žižek's critique goes beyond adding a fourth quadrant. He argued that the framework, as Rumsfeld used it, could function as a way of acknowledging uncertainty while downplaying institutional knowledge about predictable outcomes.

## Summary

The Rumsfeld Square is a 2x2 epistemological framework built from one binary axis crossed with itself: whether a fact is present in your model, and whether you are aware of your own knowledge state about that fact.

The four cells — known knowns, known unknowns, unknown knowns, and unknown unknowns — cover the space of epistemic positions an agent can occupy relative to a piece of information. The framework's value is in forcing explicit accounting for what you know, what you know you do not know, and what you do not know you know.

The most dangerous quadrant is unknown unknowns. The most neglected quadrant is unknown knowns. Good practice is to move both toward known knowns through targeted investigation, diverse review, and documentation.
