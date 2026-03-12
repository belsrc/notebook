## AGENTS.md vs. `openspec/config.yaml`: Separation of Concerns

The guiding principle is straightforward: **AGENTS.md governs _how_ the agent behaves; `config.yaml` governs _what_ the agent knows about the project.** They sit at different layers of the instruction stack and serve fundamentally different purposes.

### The Mental Model

```
┌──────────────────────────────────────────────────────────────┐
│                    Agent Instruction Stack                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  AGENTS.md / CLAUDE.md          (Layer 1: Agent Identity)    │
│  ─────────────────────                                       │
│  WHO the agent is, HOW it works, WHEN it communicates        │
│                                                              │
│  • Conversation style & tone                                 │
│  • Workflow procedures (PR flow, commit conventions)         │
│  • Decision-making heuristics ("ask before deleting files")  │
│  • Tool usage preferences ("prefer vitest over jest")        │
│  • Communication protocols ("summarize changes in bullets")  │
│  • Role definition ("you are a senior TS engineer")          │
│  • Behavioral guardrails ("never force-push to main")        │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  openspec/config.yaml           (Layer 2: Project DNA)       │
│  ────────────────────                                        │
│  WHAT the project is, WHAT stack it uses, WHAT rules apply   │
│                                                              │
│  • Domain description ("real-time geospatial viz platform")  │
│  • Tech stack & versions (TypeScript, Deck.gl, WebGPU)       │
│  • Architectural patterns (FP, data-oriented design)         │
│  • Coding standards (type > interface, no classes)           │
│  • Project structure & conventions                           │
│  • Per-artifact rules for OpenSpec workflows                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Why This Separation Matters

The old `project.md` approach was passive; agents might read it, might not, might forget what they read. The new `config.yaml` context is actively injected into every OpenSpec planning request. This means your project conventions are reliably present when the AI generates artifacts like proposals, specs, designs, and task lists.

AGENTS.md, by contrast, is loaded at session start by the agent runtime (Claude Code reads `CLAUDE.md`, Cursor reads `.cursorrules`, etc.). It shapes the agent's general behavior across all interactions, not just OpenSpec artifact creation.

The key distinction:

- **AGENTS.md is read once per session** and shapes all agent behavior globally.
- **`config.yaml` context is injected per-artifact**, the `generateInstructions()` function assembles instructions by wrapping context in `<context>...</context>` tags and artifact-specific rules in `<rules>...</rules>` tags, then appending the template content from the schema.

### What Goes Where : Concrete Examples

**AGENTS.md (behavioral / procedural):**

```markdown
# Agent Behavior
- Always run `pnpm check` before committing
- Use Conventional Commits (feat:, fix:, chore:)
- When uncertain about scope, ask before proceeding
- Prefer small, focused PRs over large changesets
- Respond concisely; avoid over-explaining

# Workflow
- For new features, start with /opsx:propose
- Run vitest after any code modification
- Reference existing specs before creating new ones
```

**`openspec/config.yaml` (structural / factual):**

```yaml
schema: spec-driven

context: |
  Domain: Geospatial visualization platform for real-time map layers.
  Monorepo: @certes/* packages managed with Turborepo + PNPM + Changesets.
  Stack: TypeScript 5.x, Deck.gl, WebGPU, Apache Arrow, Vitest.
  Patterns: Functional programming, data-oriented design, data-last curried APIs.
  Standards: `type` over `interface`, functions over classes, no `let` without perf reason.
  Build: GitHub Actions CI/CD with Turborepo caching.
  Key constraints: Main-thread sovereignty, zero unnecessary allocations.

rules:
  proposal:
    - Include performance impact assessment
    - Identify affected @certes/* packages
  specs:
    - Use Given/When/Then format for scenarios
    - Reference existing specs in openspec/specs/ before creating new patterns
  design:
    - Include memory/allocation analysis for hot paths
    - Document WebGPU/WebGL fallback strategy where applicable
  tasks:
    - Break into max 2-hour chunks
    - Each task must be independently testable
```

### The `config.yaml` Schema in Detail

The configuration is validated using a Zod-based schema with three fields: `schema` (required string, which workflow schema to use), `context` (optional string, max 50KB project context injected into all artifacts), and `rules` (optional record mapping artifact IDs to string arrays, per-artifact constraints).

```
┌─────────────────────────────────────────────────────────┐
│  openspec/config.yaml                                   │
├──────────┬──────────┬───────────────────────────────────┤
│  Field   │ Required │ Injection Target                  │
├──────────┼──────────┼───────────────────────────────────┤
│ schema   │ Yes      │ Selects workflow artifact graph   │
│ context  │ No       │ ALL artifacts (global preamble)   │
│ rules    │ No       │ MATCHING artifact only            │
└──────────┴──────────┴───────────────────────────────────┘
```

Because context is injected into every request, you'll want to be concise. Focus on what really matters. The 50KB max is generous, but token-efficient descriptions produce better AI output.

Valid artifact IDs for the built-in `spec-driven` schema are: `proposal`, `specs`, `design`, and `tasks`. Context appears in ALL artifacts; rules ONLY appear for the matching artifact.

## Generation Prompt

```
/openspec-explore

## Task
The `openspec/config.yaml` has been initialized but is in its fresh, blank
state. Explore the current codebase and populate it with appropriate values.

## Pre-requisites
1. Read `AGENTS.md` (or `CLAUDE.md`) first to understand what behavioral
   instructions already exist. Do NOT duplicate any of that content.
2. Read the existing codebase structure, `package.json` files, tsconfig,
   CI config, and any existing `openspec/specs/` to extract factual
   project details.

## config.yaml Structure
The file has exactly three fields:

\```yaml
# Required: default workflow schema
schema: spec-driven

# Optional (max 50KB): injected into ALL artifact instructions
context: |
  <concise project facts here>

# Optional: injected ONLY into the matching artifact
# Keys must match schema artifact IDs: proposal, specs, design, tasks
rules:
  proposal:
    - <rule>
  specs:
    - <rule>
  design:
    - <rule>
  tasks:
    - <rule>
\```

## What belongs in `context:`

Factual, structural, rarely-changing project DNA:
- Domain description (what the system does)
- Tech stack with versions
- Monorepo/package structure
- Architectural patterns and constraints
- Coding standards and conventions
- Build/CI/CD toolchain
- Key invariants or non-negotiable constraints

## What does NOT belong (goes in AGENTS.md instead)

- Conversation style or tone
- Workflow procedures (commit flow, PR process)
- Agent decision-making heuristics
- Tool usage preferences
- Communication protocols

## Separation Rule

AGENTS.md → HOW the agent behaves (process, style, workflow)
config.yaml → WHAT the project is (stack, patterns, standards, domain)
They complement; they never duplicate.

## Output
Produce the complete `openspec/config.yaml` file ready to save.
Keep `context:` under 2KB for token efficiency.
Include at least 2 rules per artifact type where meaningful.
```
