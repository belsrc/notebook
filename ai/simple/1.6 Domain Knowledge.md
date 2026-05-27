---
tags:
  - ai
gardening: 🌳
date: 2026-03-16
---
**Definition:** The specialized understanding of a particular problem space that sits _outside_ general programming competence or general project management competence. It's the difference between knowing how to write a project brief and knowing _why_ your organization calls the launch gate a "readiness review" and routes it through Engineering before Product, not the other way around. It's the difference between knowing how to write TypeScript and knowing _why_ a geospatial system reprojects WGS84 coordinates to Web Mercator before tiling.

From the agent's perspective, domain knowledge is the hardest category of inference. The agent has broad but shallow coverage of most domains from training data. It can write a correct `for` loop in any language, but it may not know that your organization's "layer" abstraction means something very specific and different from what Deck.gl's documentation calls a "layer." It can draft a stakeholder update, but it may not know that in your organization, "stakeholder" means executive sponsors only, never the implementation team leads.

### The three layers of domain knowledge

```
┌──────────────────────────────────────────────────────────┐
│                  DOMAIN KNOWLEDGE STACK                  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  PUBLIC DOMAIN KNOWLEDGE                           │  │
│  │  "What does WGS84 mean?"                           │  │
│  │  "What is a RACI matrix?"                          │  │
│  │  Agent: I know this from training data.            │  │
│  │  Action: Don't tell me.                            │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  APPLIED DOMAIN KNOWLEDGE                          │  │
│  │  "Why do we reproject before filtering?"           │  │
│  │  "Why does QA sign off before legal, not after?"   │  │
│  │  Agent: I can probably figure this out if I read   │  │
│  │         enough of your code, docs, or briefs.      │  │
│  │  Action: State it if it saves significant cost.    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  TRIBAL DOMAIN KNOWLEDGE                           │  │
│  │  "A 'coffin corner' is a four-corner overlay       │  │
│  │   rendered on a track to indicate hover or         │  │
│  │   selected state — not a tooltip, not a highlight, │  │
│  │   not a bounding box."                             │  │
│  │  "'Done' on our board means shipped to prod AND    │  │
│  │   verified by the client — not merged to main."    │  │
│  │  Agent: I cannot derive this. It's your term,      │  │
│  │         your mapping, your decision.               │  │
│  │  Action: You MUST tell me.                         │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

**Public domain knowledge** is what the agent absorbed during training. Standard protocols, well-documented frameworks, textbook concepts. The agent knows what a Mercator projection is, what GeoJSON looks like, what an ECS architecture does. It also knows what a sprint retrospective is, what a critical path means in project scheduling, and what a RACI chart is for. This is the agent's baseline competence in your field.

**Applied domain knowledge** is how public concepts get wired together in _your_ system. The agent can often reconstruct this by reading your code or documents, but it's more expensive than public knowledge. It's the "how" and "why" of your specific implementation choices. Why you use `DataFilterExtension` for GPU-side filtering instead of CPU-side predicate evaluation. Why your release process runs a security review in Sprint N-1 rather than Sprint N. The agent might figure this out from context, but it could also guess wrong and a wrong guess in a project plan lands in an executive slide deck.

**Tribal domain knowledge** is the dangerous one. It exists only inside your team's heads. It never appeared in training data. It's naming conventions that diverge from industry norms, business rules encoded as process steps, and overloaded terms that mean something different in your context than in the public domain. The agent has _zero_ ability to infer this. If you don't state it, the agent will substitute its own understanding, confidently and incorrectly.

### Why this matters for agents specifically

A human junior developer or new PM joining your team absorbs tribal knowledge over weeks through conversation, code review, and osmosis. An agent gets a single context window. It has no hallway conversations, no Slack history, no memory of the PR where someone explained why the `ImplementationConfig` event handlers exist or the meeting where leadership decided that "MVP" in your organization explicitly excludes mobile.

This creates a sharp asymmetry between how humans and agents acquire domain knowledge:

```
Human (developer or PM):
  Public domain  ──► learns in school / training
  Applied domain ──► learns by reading codebase/docs over days
  Tribal domain  ──► learns through team interaction over weeks

Agent:
  Public domain  ──► has from training data               ✓
  Applied domain ──► can read code/docs on critical path  ✓ (with cost)
  Tribal domain  ──► has no source whatsoever             ✗
```

The agent's blind spot is tribal knowledge, and this is precisely where most mistakes from agent-generated work originate. For engineers, it's not syntax errors or algorithmic mistakes, it's perfectly valid code that violates an unstated assumption your team shares. For project managers, it's not structural errors in a plan, it's a schedule that looks correct but assigns the wrong lead to a workstream, or a status report that uses "blocked" when your team's convention reserves that word for executive escalation only.

### Concrete examples

**Public: don't state:**

> _(Engineering)_ "GeoJSON uses longitude, latitude ordering per RFC 7946."

> _(Project Management)_ "A project's critical path is the longest sequence of dependent tasks."

The agent knows these. Telling it wastes tokens and risks contradicting itself if you get the RFC number wrong.

**Applied: state if off the critical path:**

> _(Engineering)_ "The `createPipeline` function in `src/engine/pipeline.ts` memoizes intermediate transforms by reference equality on the input array. If you mutate the array in place instead of returning a new one, the cache serves stale results."

> _(Project Management)_ "The dependency matrix for this program lives in `planning/deps.xlsx`, not in Jira. Jira links are aspirational and often stale; the spreadsheet is the source of truth for scheduling decisions."

The agent _could_ discover either of these by reading the codebase or project files, but if the current task is focused elsewhere, a single sentence saves significant exploratory cost.

**Tribal: always state:**

> _(Engineering)_ "We call the four-corner selection overlay a 'coffin corner'. It is distinct from a highlight, a bounding box, and a tooltip, it has its own render path and its own component. Using any other term will break grep, confuse reviewers, and risk collision with the generic bounding-box utilities."

> _(Project management)_ "In our status reports, 'at risk' means the PM has already identified a mitigation plan. 'Escalation needed' means no mitigation exists yet. These are not interchangeable, the steering committee treats them differently and expects different follow-up actions."

No amount of reading code, project files, or training data tells the agent either of these things. They are your team's conventions. Without them, the agent will freely substitute synonyms and introduce inconsistencies that compound over time.

### The operational takeaway

When writing persistent context for an agent (`CLAUDE.md`, system prompts, skill files), audit every statement against the domain knowledge stack:

```
For each statement, ask:
  Is this public domain knowledge?
    → Delete it. The agent knows.
  Is this applied domain knowledge on the typical critical path?
    → Delete it. The agent will read the code or documents.
  Is this applied domain knowledge OFF the critical path?
    → Keep it if it prevents expensive wrong turns.
  Is this tribal domain knowledge?
    → Keep it unconditionally. The agent is blind here.
```

The highest-value content you can put in front of an agent is tribal domain knowledge. Everything else is either redundant or discoverable. This holds whether you are an engineer writing a skill file or a project manager writing a context document for a planning assistant, the shape of the problem is identical.

### Practical Guidance by Role

#### For Engineers Writing Agent Context

Focus on three categories of tribal knowledge:

- **Naming conventions:** If your team uses a term differently from how the broader industry uses it, state the term, its scope, and why it was chosen. One sentence prevents weeks of naming drift.
- **Architectural invariants:** Constraints the agent cannot derive from reading a single file. "All data transformations must be pure functions" or "we never mutate state in the event handlers" are examples. These are load-bearing decisions that span the codebase.
- **Contextual overrides:** Places where your system deliberately violates a common pattern for a specific reason. The agent's training data will push it toward the common pattern unless told otherwise.

#### For Project Managers Preparing Agent Briefs

You may not write code, but you control something equally important: the scope and vocabulary of the work. When briefing an agent (or briefing an engineer who will work with an agent), include:

- **Team vocabulary:** A glossary of terms your team uses that differ from industry defaults. This is the single highest-leverage artifact you can produce for agent productivity.
- **Process constraints:** If work must happen in a specific order, or certain outputs require specific reviewers, state it. The agent will default to the most common workflow in its training data, which is probably not yours.
- **Success criteria:** What "done" looks like in your context. The agent is very good at producing technically correct output. Whether that output meets _your_ definition of correct depends on you stating what that definition is.
