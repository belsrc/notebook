---
tags:
  - ai
gardening: 🌳
date: 2026-03-16
---
Don't spend your briefing budget stating facts the agent can derive from materials it already has access to. "Infer" here means any conclusion the agent can reach by reading documents, observing structure, running tools, or applying general knowledge without you explicitly spelling it out.

The principle exists because every token (word or phrase in the agent's input) you spend on redundant information is a token that (a) costs money, (b) displaces genuinely novel instruction, and (c) can _conflict_ with the source of truth if the underlying material changes but your description doesn't.

Think of briefing an agent the way you'd brief a skilled consultant who has already read the project files on your shared drive. You wouldn't read the files back to them. You'd tell them what the files can't tell them: your priorities, your constraints, and the decisions that aren't written down anywhere.

### The task's critical path

The agent doesn't read everything, it reads whatever it needs to complete the task at hand. That set of materials is the **critical path**. Inference from those materials is essentially free, because the agent was going to read them anyway.

> **"Don't tell the agent what it will already encounter while doing the work."**

If a fact lives in some file or document the agent has _no reason to open_ for the current task, stating it in your briefing isn't redundant, it's saving the agent from having to go find it. In that case the calculus flips:

```
Cost of stating it in your briefing:  a few words, once
Cost of agent discovering it:         searching, reading, reasoning
                                      (many times more expensive)
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

**1. Syntactic inference:** Nearly free. The agent can see it directly in the material.

| Role     | Don't say                            | Because                                 |
| -------- | ------------------------------------ | --------------------------------------- |
| Engineer | _"This project uses TypeScript."_    | The agent reads the config files.       |
| PM       | _"This brief is for the Q3 launch."_ | The date and title are on the document. |

**2. Structural inference:** Cheap. The agent reads an index, outline, or directory.

| Role     | Don't say                                     | Because                                     |
| -------- | --------------------------------------------- | ------------------------------------------- |
| Engineer | _"Packages are organized under `/packages`."_ | A single directory listing reveals this.    |
| PM       | _"Marketing reports to the VP of Growth."_    | The agent reads the org chart you attached. |

**3. Semantic inference:** Moderate. The agent reads material and understands intent.

| Role     | Don't say                                                                         | Because                                                        |
| -------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| Engineer | _"The `transform` function converts a config object into a visualization layer."_ | The agent reads the function signature and body.               |
| PM       | _"The weekly status report summarizes schedule, budget, and risks."_              | The agent reads the template. The structure speaks for itself. |

**4. Domain inference:** Moderate to expensive. The agent applies general knowledge.

| Role     | Don't say                                                         | Because                                                  |
| -------- | ----------------------------------------------------------------- | -------------------------------------------------------- |
| Engineer | _"We use CQL because OGC Filter XML is verbose."_                 | The agent knows both formats and can infer the tradeoff. |
| PM       | _"We use Slack instead of email for faster async communication."_ | The agent understands what Slack and email are.          |

**5. Strategic inference:** Expensive or impossible. The agent _cannot_ derive this from any material.

| Role     | Do say                                                                              | Because                                                                                    |
| -------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Engineer | _"Pure functions return `Result<T, E>`, never throw."_                              | This is a policy choice. No amount of reading code reveals it as a rule.                   |
| PM       | _"Never share budget figures with vendors before Procurement signs off."_           | This is an organizational policy. No document makes it explicit.                           |
| Engineer | _"Don't use classes. Functions and objects, always."_                               | The code shows what _is_. Only you can tell the agent what _must be_.                      |
| PM       | _"Always cc the account manager on client escalations, they own the relationship."_ | The agent can observe the pattern in past emails, but not that it's a non-negotiable rule. |

These are _policy decisions_. The code or documents show what _is_. Only your briefing can communicate what _must be_.

### The heuristic

A practical test for whether to include something:

```
Could the agent recover this information by:
  1. Reading a document it already has?              → Don't state it.
  2. Searching or scanning available materials?      → Don't state it.
  3. Applying common knowledge about the domain?     → Don't state it.
  4. Observing consistent patterns in past work?     → Don't state it.
  5. None of the above?                              → State it. This is novel.
```

```
Will the agent read this material to complete the task?
  ├─ YES ──► Don't restate its contents. Redundant.
  └─ NO ───► Is the information a constraint, intent, or priority?
               ├─ YES ──► State it. Novel + saves search cost.
               └─ NO ───► Probably don't state it. If the agent
                          doesn't need the material and the fact
                          isn't a policy decision, it's likely
                          irrelevant to the task entirely.
```

Novel information falls into roughly three buckets:

- **Constraints:** rules invisible in any document ("never use `any` in TypeScript"; "never quote a price to a client without legal review")
- **Intent:** the _why_ behind decisions ("we chose this architecture for type-checking speed"; "we cc the account manager because they own the relationship, not just for visibility")
- **Priorities:** what to optimize for when tradeoffs arise ("safety > performance > simplicity"; "hit the launch date before scope; cut features, not the deadline")

### The Context File

Most agent systems let you define a persistent context document, loaded before every task, every time. For Claude, this is `CLAUDE.md` or `MEMORY.md`. Project managers can think of it as a **standing brief**: a one-page document a new contractor would read on day one that tells them how you operate, not what the project is.

Because this document is read on _every_ task, every word in it is paid every single time. The bar is highest here: only include information that is (a) genuinely novel, can't be inferred from materials the agent will encounter on a **typical task** and (b) broadly applicable, relevant to most tasks, not just one.

### Concrete example

**(Engineering) Bad - tells the agent what it can infer:**

```markdown
## Architecture
This is a TypeScript monorepo using pnpm workspaces.
Packages are in /packages. We have:
- @mypackage/composition: compose, pipe, curry utilities
- @mypackage/list: list manipulation functions
- @mypackage/lazy: generator-based lazy evaluation
The project uses vitest for testing. Tests are co-located
with source files using .test.ts extension.
CI runs via GitHub Actions with Turborepo.
```

The agent can derive _every single line_ of this by reading the filesystem.

**(Engineering) Good - tells the agent only what it can't infer:**

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
conditional types) because we measured 6-14x faster type-checking.
This is intentional, don't "simplify" it.
```

Every line here is something the agent genuinely cannot derive from the code alone. The token budget goes entirely toward steering behavior.