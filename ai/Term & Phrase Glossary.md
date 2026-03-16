## "Never tell the agent what it can infer"

**Definition:** Don't spend context window tokens stating facts the agent can derive from artifacts it already has access to. "Infer" here means any conclusion the agent can reach by reading code, observing structure, running tools, or applying general knowledge — without you explicitly spelling it out.

The principle exists because every token you spend on redundant information is a token that (a) costs money, (b) displaces genuinely novel instruction, and (c) can _conflict_ with the source of truth if the code changes but your description doesn't.

### The task's critical path

"Reading code" doesn't mean _all code in the repo_. It means code that falls on the **critical path of the current task**. The agent will read certain files _anyway_ to do its job. Inference from those files is essentially free — it's a byproduct of work already happening.

> **"Don't tell the agent what it will already encounter while doing the work."**

That's the precise version. If the agent is going to open `compose.ts` to fix a bug in it, you don't need to tell it that `compose` is variadic and right-to-left. It's about to read the implementation. If a fact lives in some file the agent has _no reason to open_ for the current task, then stating it in the prompt isn't redundant — it's saving the agent a tool call (or several). In that case the calculus flips:

```
Cost of stating it in prompt:  ~20 tokens once
Cost of agent discovering it:  tool call + file read + reasoning
                               (hundreds to thousands of tokens)
```

### The inference spectrum

"Infer" isn't one thing. It's a spectrum of derivation cost, from nearly free to expensive:

```
┌──────────────────────────────────────────────────────────┐
│              INFERENCE COST SPECTRUM                     │
│                                                          │
│  Cheap ◄────────────────────────────────────► Expensive  │
│                                                          │
│  Syntactic   Structural   Semantic   Domain   Strategic  │
│  inference   inference    inference  inference inference │
│                                                          │
│  "What       "How is      "What      "Why     "What      │
│   lang is     this         does it    this     should    │
│   this?"      organized?"  do?"       way?"    change?"  │
└──────────────────────────────────────────────────────────┘
```

**1. Syntactic inference** — Nearly free. The agent can see it directly.

Don't say: _"This project uses TypeScript with ES modules."_ The agent reads `tsconfig.json` and the `import` statements. It already knows.

**2. Structural inference** — Cheap. The agent reads a directory tree or a few files.

Don't say: _"The project follows a monorepo structure with packages in `/packages`."_ A single `ls` or a glance at `package.json` workspaces reveals this.

**3. Semantic inference** — Moderate. The agent reads code and understands intent.

Don't say: _"The `transform` function converts a config object into a Deck.gl layer."_ The agent reads the function signature, the return type, and the body. It can figure out what `transform` does.

**4. Domain inference** — Moderate to expensive. The agent applies general knowledge to the codebase.

Don't say: _"We use CQL because OGC Filter XML is verbose and harder to compose."_ The agent knows what CQL and OGC Filter XML are. It can infer the tradeoff from general knowledge.

**5. Strategic inference** — Expensive or impossible. The agent _cannot_ derive this.

**Do say:** _"We never throw from pure functions — return `Result<T, E>` instead."_ **Do say:** _"Cursor performance matters more than bundle size in this layer system."_ **Do say:** _"Don't use classes. We use functions + objects everywhere, intentionally."_

These are _policy decisions_. No amount of reading code tells the agent _why_ you chose this, or that you want it enforced going forward. The code shows what _is_, not what _must be_.

### The heuristic

A practical test for whether to include something:

```
Could the agent recover this information by:
  1. Reading a file it already has?        → Don't state it.
  2. Running a tool (ls, grep, test)?      → Don't state it.
  3. Applying common technical knowledge?   → Don't state it.
  4. Reading your code's patterns?          → Don't state it.
  5. None of the above?                     → State it. This is novel.
```

```
Will the agent read this file/artifact to complete the task?
  ├─ YES ──► Don't state what it contains. Redundant.
  └─ NO ───► Is the information a constraint, intent, or priority?
               ├─ YES ──► State it. Novel + saves exploration cost.
               └─ NO ───► Probably don't state it. If the agent
                          doesn't need the file and the fact isn't
                          a policy decision, it's likely irrelevant
                          to the task entirely.
```

Novel information falls into roughly three buckets:

- **Constraints** — rules the agent can't see in the code ("never use `any`")
- **Intent** — the _why_ behind decisions ("we curry data-last for composition")
- **Priorities** — what to optimize for when tradeoffs arise ("safety > performance > simplicity")

### The CLAUDE.md implication

This is why the principle matters most for persistent context files like `CLAUDE.md` or `MEMORY.md`. These are loaded on _every_ task regardless. Every token in them is paid every single invocation. So the bar is highest there: only include information that is (a) novel (can't be inferred from the critical path of _any typical task_) and (b) broadly applicable (relevant to most tasks, not just one).

A constraint like "never use classes" clears both bars — it applies to every task and no file reading reveals it as a _policy_. A description like "the monorepo has 6 packages" clears neither — the agent discovers it on any task that touches the repo, and it's not actionable guidance.

### Concrete example

**Bad `CLAUDE.md` (tells the agent what it can infer):**

```markdown
## Architecture
This is a TypeScript monorepo using pnpm workspaces.
Packages are in /packages. We have:
- @certes/composition: compose, pipe, curry utilities
- @certes/list: list manipulation functions
- @certes/lazy: generator-based lazy evaluation
The project uses vitest for testing. Tests are co-located
with source files using .test.ts extension.
CI runs via GitHub Actions with Turborepo.
```

The agent can derive _every single line_ of this by reading the filesystem.

**Good `CLAUDE.md` (tells the agent only what it can't infer):**

```markdown
## Constraints
- `type` over `interface`, always.
- Functions + objects over classes, always.
- Curried, data-last APIs — no exceptions.
- Never use `any`. Prefer `unknown` + narrowing.
- Pure functions return Result<T, E>, never throw.

## Priorities
Safety > Performance > Simplicity > Readability

## Intent
The curry implementation uses explicit overloads (not recursive
conditional types) because we measured 6-14× faster type-checking.
This is intentional — don't "simplify" it.
```

Every line here is something the agent genuinely cannot derive from the code alone. The token budget goes entirely toward steering behavior.

---
## Domain Knowledge

**Definition:** The specialized understanding of a particular problem space that sits *outside* general programming competence. It's the difference between knowing how to write TypeScript and knowing *why* a geospatial system reprojects WGS84 coordinates to Web Mercator before tiling.

From the agent's perspective, domain knowledge is the hardest category of inference. The agent has broad but shallow coverage of most domains from training data. It can write a correct `for` loop in any language, but it may not know that your organization's "layer" abstraction means something very specific and different from what Deck.gl's documentation calls a "layer."

### The three layers of domain knowledge

```
┌──────────────────────────────────────────────────────────┐
│                  DOMAIN KNOWLEDGE STACK                  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  PUBLIC DOMAIN KNOWLEDGE                           │  │
│  │  "What does WGS84 mean?"                           │  │
│  │  Agent: I know this from training data.            │  │
│  │  Action: Don't tell me.                            │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  APPLIED DOMAIN KNOWLEDGE                          │  │
│  │  "Why do we reproject before filtering?"           │  │
│  │  Agent: I can probably figure this out if I read   │  │
│  │         enough of your code and comments.          │  │
│  │  Action: State it if it saves significant cost.    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  TRIBAL DOMAIN KNOWLEDGE                           │  │
│  │  "A 'coffin corner' in our system means the        │  │
│  │   convergence zone of stall speed and max speed    │  │
│  │   at altitude — we render it as a wedge overlay."  │  │
│  │  Agent: I cannot derive this. It's your term,      │  │
│  │         your mapping, your rendering decision.     │  │
│  │  Action: You MUST tell me.                         │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

**Public domain knowledge** is what the agent absorbed during training. Standard protocols, well-documented APIs, textbook concepts. The agent knows what a Mercator projection is, what GeoJSON looks like, what an ECS architecture does. This is the agent's baseline competence in your field.

**Applied domain knowledge** is how public concepts get wired together in *your* system. The agent can often reconstruct this by reading your code, but it's more expensive than public knowledge. It's the "how" and "why" of your specific implementation choices. Why you use `DataFilterExtension` for GPU-side filtering instead of CPU-side predicate evaluation. The agent might figure this out, but it could also guess wrong and filter on the CPU.

**Tribal domain knowledge** is the dangerous one. It exists only inside your team's heads. It never appeared in training data. It's naming conventions that diverge from industry norms, business rules encoded as code patterns, overloaded terms that mean something different in your context than in the public domain. The agent has *zero* ability to infer this. If you don't state it, the agent will substitute its own understanding — confidently and incorrectly.

### Why this matters for agents specifically

A human junior developer joining your team absorbs tribal knowledge over weeks through conversation, code review, and osmosis. An agent gets a single context window. It has no hallway conversations, no Slack history, no memory of the PR where someone explained why the `ImplementationConfig` event handlers exist.

This creates an asymmetry:

```
Human developer:
  Public domain  ──► learns in school / training
  Applied domain ──► learns by reading codebase over days
  Tribal domain  ──► learns through team interaction over weeks

Agent:
  Public domain  ──► has from training data          ✓
  Applied domain ──► can read code on critical path  ✓ (with cost)
  Tribal domain  ──► has no source whatsoever        ✗
```

The agent's blind spot is tribal knowledge, and this is precisely where most bugs from agent-generated code originate. Not syntax errors. Not algorithmic mistakes. The agent writes perfectly valid code that violates an unstated assumption your team shares.

### Concrete examples

**Public — don't state:**

> "GeoJSON uses longitude, latitude ordering per RFC 7946."

The agent knows this. Telling it wastes tokens and risks contradicting itself if you get the RFC number wrong.

**Applied — state if off the critical path:**

> "The `createPipeline` function in `src/engine/pipeline.ts` memoises intermediate transforms by reference equality on the input array. If you mutate the array in place instead of returning a new one, the cache serves stale results."

The agent *could* read the implementation and figure this out, but if the current task is in a different package, this one sentence saves a multi-file exploration.

**Tribal — always state:**

> "We call them 'stages', not 'steps' or 'phases'. A stage is a single unit of work inside a pipeline — the term is used in the config schema, the type system, and every internal API. Using a synonym will compile but will confuse grep, code review, and future contributors."

No amount of reading code or training data tells the agent that your team chose "adornment" deliberately and that it's a term with precise scope. Without this, the agent will freely substitute "decorator" or "annotation" and introduce naming inconsistency across the codebase.

### The operational takeaway

When writing persistent context for an agent (`CLAUDE.md`, system prompts, skill files), audit every statement against the domain knowledge stack:

```
For each statement, ask:
  Is this public domain knowledge?
    → Delete it. The agent knows.
  Is this applied domain knowledge on the typical critical path?
    → Delete it. The agent will read the code.
  Is this applied domain knowledge OFF the critical path?
    → Keep it if it prevents expensive wrong turns.
  Is this tribal domain knowledge?
    → Keep it unconditionally. The agent is blind here.
```

The highest-value content you can put in front of an agent is tribal domain knowledge. Everything else is either redundant or discoverable.

---

