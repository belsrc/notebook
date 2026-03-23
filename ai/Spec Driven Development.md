---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-03-23
reference:
---
## The Problem Before the Solution

Before we can appreciate Spec Driven Development (SDD), we need to understand the failure mode it was designed to fix.

Imagine you ask a contractor to "build you a house." No blueprints, no dimensions, no material list. The contractor starts building. Three weeks later, they hand you a two-bedroom ranch when you needed a four-bedroom colonial. Everyone worked hard. Nothing was malicious. The outcome was still a disaster.

This is the **vague instruction problem**, and it has existed in software development since the beginning. Traditional software responses to this problem gave us requirements documents, user stories, and acceptance criteria: written contracts describing what the software must do before a single line of code is written.

Hold that mental model and apply it to AI.

## Why AI Makes This Problem Worse

When you give a task to a human engineer, they have a lifetime of context to fill in the gaps. They know what a "login form" looks like. They know that "export data" probably means a CSV file. They will ask clarifying questions when something is genuinely ambiguous.

AI models, particularly Large Language Models (LLMs), behave differently. They are **completion engines**. Their job is to produce the most statistically plausible continuation of whatever text you gave them. When your instruction is vague, they do not pause and ask for clarification; they guess. And they guess confidently.

Consider this prompt:

```
"Build me an agent that handles customer support tickets."
```

An LLM will produce something. It might be impressive-looking. But it has made dozens of silent assumptions:

- What channel? Email, chat, a ticketing API?
- What escalation rules apply?
- What tone should it use?
- What is it allowed to do vs. defer to a human?
- What data can it access?
- What counts as "handled"?

Every one of those silent assumptions is a potential production failure.

## What is Spec Driven Development?

Spec Driven Development is a methodology where you write a formal, structured specification for a system **before you build it**, and that specification serves as the authoritative source of truth throughout the entire development lifecycle.

This is not a new idea. OpenAPI specifications have described REST APIs in machine-readable contracts for years. What is new is applying this discipline to the instruction of AI agents.

In SDD for AI, the specification is not just documentation for humans. It is **input to the AI itself**. The spec becomes part of the agent's context: its system prompt, its tool definitions, and its evaluation criteria.

Here is a simple mental model:

```
┌─────────────────────────────────────────────────────────┐
│                  Traditional Development                │
│                                                         │
│  Human Idea  →  Vague Request  →  Code  →  Maybe Works  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  Spec Driven Development                │
│                                                         │
│  Human Idea  →  Formal Spec  →  Code/Agent/Prompt       │
│                     ↑                   ↓               │
│                  Validation  ←  Verifiable Outcomes     │
└─────────────────────────────────────────────────────────┘
```

The key shift: the spec is a **living artifact** that connects intent (what you want) to implementation (what gets built) to validation (how you know it worked).

## The Anatomy of a Good Spec for AI

A specification for an AI agent is not a paragraph of instructions. It is a structured document with distinct, purpose-built sections.

### Role and Scope

This section answers: *What is this agent, and what is it explicitly NOT?*

Defining what an agent does is easy. Defining what it refuses to do is the part most teams skip. In AI systems, **negative space is as important as positive definition**. An agent without clearly defined refusals will attempt to handle anything that looks vaguely related to its purpose, often badly.

**Weak:**

```
The agent assists users with billing questions.
```

**Strong:**

```
Role: Billing Support Agent
Scope IN:
  - Invoice lookups for the current and previous 12 billing periods
  - Payment method updates (add, remove, set default)
  - Subscription plan queries (read-only)

Scope OUT:
  - Refund processing (escalate to human billing team)
  - Account deletion
  - Any task not explicitly listed above
```

The `Scope OUT` section is not boilerplate. It is functional. When this spec is included in an agent's system prompt, the agent has explicit permission to say "I can't help with that" rather than fabricating a path forward.

### Inputs and Outputs

This section describes the data contract: what does the agent receive, and what is it expected to produce?

Be as concrete as possible. Vague input descriptions produce vague outputs.

**Example:**

```
Input:
  - user_id: string (UUID v4)
  - query: string (natural language, max 500 characters)
  - conversation_history: array of { role: "user"|"assistant", content: string }

Output:
  - response: string (natural language reply to the user)
  - action_taken: "none" | "lookup" | "update" | "escalate"
  - escalation_reason: string | null (required if action_taken = "escalate")
```

This matters for AI agents specifically because LLMs can produce text in any format. Without a structured output contract, a downstream system that expects JSON may receive a sentence that *describes* JSON.

### Tools and Capabilities

If the agent has access to tools (functions it can call, APIs it can invoke), the spec must list them with their purpose and constraints.

```
Tools Available:

  get_invoice(user_id, invoice_id)
    Purpose: Retrieves a single invoice record
    Rate limit: 10 calls per agent turn
    Constraint: Can only retrieve invoices belonging to the provided user_id

  update_payment_method(user_id, method_token)
    Purpose: Sets a new default payment method
    Rate limit: 1 call per agent turn
    Constraint: Requires explicit user confirmation before calling
```

The constraint column is critical. Without it, an agent might call `update_payment_method` without confirmation because nothing told it not to.

### Behavioral Rules

This section codifies policies as explicit, unambiguous statements. Think of it as if-then logic written in plain language.

```
Behavioral Rules:

  1. If the user asks for data that belongs to a different user_id than
     the one provided, respond with a denial and do not call any tools.

  2. If the query is ambiguous, ask one clarifying question before
     taking any action.

  3. If a tool call fails with an error, inform the user and offer
     to escalate. Do not retry automatically.

  4. Always acknowledge the user's emotion before delivering
     technical information if the query contains language
     indicating frustration.
```

Rules like these are extremely difficult to reverse-engineer from observed agent behavior after the fact. Writing them down first is the only way to ensure consistency.

### Success and Failure Criteria

This section defines how you evaluate whether the agent is working correctly. Without it, "does the agent work?" is a matter of opinion.

```
Success Criteria:
  - The response accurately reflects the data returned by tools
  - The action_taken field matches what the agent actually did
  - Escalation is triggered for all refund requests

Failure Criteria:
  - The agent fabricates invoice data not returned by tools
  - The agent modifies data without explicit user confirmation
  - The agent handles a task listed in Scope OUT
```

These criteria feed directly into your **evaluation harness**: the automated tests you run against the agent to verify correctness. Without them, testing is ad hoc. With them, testing is systematic.

## How the Spec Flows Into the Agent

The spec is not a document you write and then set aside. It is the source of your agent's behavior.

```
┌─────────────────────────────────────────────────────────────────┐
│                        Agent Architecture                       │
│                                                                 │
│   ┌────────────┐     ┌──────────────────────────────────────┐   │
│   │  Spec      │────▶│  System Prompt                       │   │
│   │  Document  │     │  (Role, Scope, Behavioral Rules)     │   │
│   └────────────┘     └──────────────────────────────────────┘   │
│         │                                                       │
│         │            ┌──────────────────────────────────────┐   │
│         ├───────────▶│  Tool Definitions                    │   │
│         │            │  (Capabilities, Constraints)         │   │
│         │            └──────────────────────────────────────┘   │
│         │                                                       │
│         │            ┌──────────────────────────────────────┐   │
│         └───────────▶│  Evaluation Suite                    │   │
│                      │  (Success/Failure Criteria as Tests) │   │
│                      └──────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**The system prompt** is built from the Role, Scope, and Behavioral Rules sections. It is the persistent instruction the agent carries into every conversation.

**Tool definitions** are implemented using the structure from the Tools section. If your LLM platform uses JSON Schema for tool definitions (as most do today), the spec's tool descriptions become the `description` fields. The constraints become guard logic in your tool wrapper functions.

**The evaluation suite** runs the spec's success and failure criteria against the agent using synthetic test cases. If the agent produces a fabricated invoice, a test fails.

This is the core discipline of SDD: **the spec is not documentation about the agent; it is the agent**.

## A Complete Worked Example

Let's walk through SDD end-to-end with a concrete scenario.

**Goal:** An AI agent that triages incoming GitHub issues for an open source project.

### Step 1: Write the Spec First

```
# GitHub Triage Agent - Specification v1.0

## Role and Scope

Role:
  Automated first-responder for new GitHub issues on the
  certes/platform repository.

Scope IN:
  - Labeling issues (bug, feature-request, question, duplicate)
  - Requesting missing reproduction steps from the reporter
  - Closing issues that are duplicates of known open issues
  - Assigning priority (P0/P1/P2/P3) based on severity rules below

Scope OUT:
  - Writing code or suggesting code changes
  - Making architectural decisions
  - Communicating with contributors outside GitHub issue comments

## Inputs

  - issue_title: string
  - issue_body: string
  - issue_author: string
  - existing_open_issues: array of { id, title, labels }

## Outputs

  - labels_to_apply: array of string
  - priority: "P0" | "P1" | "P2" | "P3" | null
  - comment_to_post: string | null
  - close_issue: boolean
  - close_reason: string | null

## Tools

  search_issues(query: string) -> array of { id, title, body, labels }
    Purpose: Full-text search of open issues
    Constraint: Must be called before labeling any issue as "duplicate"

  get_known_bugs() -> array of { id, title, affected_versions }
    Purpose: Retrieves the current known-bugs registry
    Constraint: Read-only

## Behavioral Rules

  1. An issue is P0 if it reports data loss or complete service outage.
  2. An issue is P1 if it reports a regression in a previously
     working feature.
  3. An issue is P2 if it is a confirmed bug with a workaround.
  4. An issue is P3 for everything else, including feature requests.
  5. Before labeling an issue "duplicate", search_issues must have
     been called and the matching issue id must be cited in the
     close_reason.
  6. If the issue body contains no steps to reproduce and the label
     is "bug", post a comment requesting reproduction steps.
     Do not close the issue.
  7. Never address the author by name in comments.

## Success Criteria

  - Priority assignment matches the severity rules above
  - Duplicate labels are only applied after search_issues is called
  - Bug issues without reproduction steps receive a comment,
    not a closure

## Failure Criteria

  - Issue labeled "duplicate" without a cited matching issue id
  - P0 assigned to a feature request
  - Comment posted that addresses the author by name
```

### Step 2: Derive the System Prompt

The system prompt is not a creative rewrite of the spec. It is a faithful translation.

```
You are an automated GitHub issue triage agent for the certes/platform
repository. Your job is to classify, label, and respond to new issues.

You are authorized to:
  - Apply labels: bug, feature-request, question, duplicate
  - Assign priority: P0, P1, P2, P3
  - Post a single comment per issue
  - Close issues that are confirmed duplicates

You are NOT authorized to:
  - Suggest code changes
  - Make architectural decisions
  - Communicate outside of GitHub issue comments

Priority Rules:
  P0 = data loss or complete service outage
  P1 = regression in previously working feature
  P2 = confirmed bug with a workaround
  P3 = everything else, including feature requests

Before labeling any issue as "duplicate", you must call search_issues
and cite the matching issue id in your close_reason. Never address
the author by name in comments.
```

### Step 3: Implement Tool Wrappers with Constraint Enforcement

The spec said `search_issues` must be called before a duplicate label is applied. We enforce this in code, not just in the prompt, because prompts can be disobeyed.

```typescript
type TriageState = {
  readonly searchWasCalled: boolean;
  readonly searchResults: ReadonlyArray<IssueSearchResult>;
};

const createTools = (state: { current: TriageState }) => ({
  search_issues: async (query: string): Promise<IssueSearchResult[]> => {
    const results = await githubSearchIssues(query);
    state.current = { ...state.current, searchWasCalled: true, searchResults: results };
    return results;
  },

  apply_label: (label: string, issueId: string): void => {
    if (label === "duplicate" && !state.current.searchWasCalled) {
      throw new Error(
        "Constraint violation: search_issues must be called before applying the duplicate label."
      );
    }
    applyGithubLabel(issueId, label);
  },
});
```

The spec's constraint is a runtime guard in the tool wrapper. If the agent attempts to shortcut the required search call, the system throws rather than silently complying.

### Step 4: Implement the Evaluation Suite

Each failure criterion becomes a test case.

```typescript
describe("GitHub Triage Agent", () => {
  it("does not label duplicate without calling search_issues", async () => {
    const result = await runAgent({
      issue_title: "App crashes on startup",
      issue_body: "It just crashes.",
      mockSearchIssues: vi.fn().mockImplementation(() => {
        throw new Error("search_issues was not called");
      }),
    });

    expect(result.labels_to_apply).not.toContain("duplicate");
  });

  it("does not assign P0 to a feature request", async () => {
    const result = await runAgent({
      issue_title: "Add dark mode support",
      issue_body: "I would love a dark mode option.",
    });

    expect(result.priority).not.toBe("P0");
  });
});
```

## SDD vs. Prompt Engineering

These two are frequently confused, so it is worth drawing a clear line between them.

**Prompt engineering** is the craft of writing effective instructions for a single interaction. It is tactical. You iterate on wording until the model behaves the way you want in the moment.

**Spec Driven Development** is a methodology that covers the full lifecycle. It is strategic. The prompt is one artifact that SDD produces, not the practice itself.

```
┌────────────────────────────────────────────────────────────────┐
│                    What SDD Encompasses                        │
│                                                                │
│   Requirement    System      Tool          Evaluation          │
│   Capture    →   Prompt  +   Definitions + Suite               │
│   (Spec)         (from spec) (from spec)   (from spec)         │
│                                                                │
│   Prompt engineering covers only the "System Prompt" slice.    │
└────────────────────────────────────────────────────────────────┘
```

The difference matters most when something goes wrong. If an agent misbehaves and your only artifact is a prompt, your debugging process is: edit the prompt, run the agent, observe. Repeat until it works. This is trial and error with no source of truth.

With a spec, your debugging process is: find which rule the agent violated, verify the rule is correctly expressed in the system prompt, verify the constraint is enforced in the tool wrapper, add a test case for the failure. That is engineering.

## SDD and Multi-Agent Systems

The value of SDD compounds when you move from a single agent to a network of agents, because now agents are calling other agents. Every agent-to-agent interface is a potential failure point.

In a multi-agent system, each agent's spec becomes the contract that other agents (and the orchestrator) rely on.

```
┌──────────────────────────────────────────────────────────────────┐
│                     Multi-Agent Triage System                    │
│                                                                  │
│  ┌─────────────┐      Spec A         ┌───────────────────────┐   │
│  │ Orchestrator│ ─────────────────▶  │ Triage Agent          │   │
│  │ Agent       │                     │ (Spec: inputs/outputs │   │
│  └─────────────┘                     │  clearly defined)     │   │
│         │                            └───────────────────────┘   │
│         │            Spec B                                      │
│         └─────────────────────────▶  ┌───────────────────────┐   │
│                                      │ Notification Agent    │   │
│                                      │ (Spec: inputs/outputs │   │
│                                      │  clearly defined)     │   │
│                                      └───────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

When the Triage Agent's spec says its output includes `escalation_reason: string | null`, the Orchestrator can be built to rely on that contract. If the spec later changes to make `escalation_reason` required, the impact is visible in the spec diff, not discovered as a runtime crash in production.

This is the same discipline that makes API versioning and contract testing valuable in distributed systems. SDD brings that discipline to agent networks.

## Common Pitfalls

### Pitfall 1: Specs That Are Too Vague

A spec that says "respond helpfully" is not a spec. Every behavioral claim in a spec should be testable. If you cannot write a test that distinguishes passing from failing for a given rule, the rule is too vague.

### Pitfall 2: Specs That Live Only in the Prompt

If the spec's constraints are only expressed as natural language instructions to the model, they can be overridden. A sufficiently persuasive user message can sometimes convince a model to ignore its system prompt instructions. Critical constraints (financial operations, data mutations, access control) must be enforced in code, not just in language.

### Pitfall 3: Specs That Are Never Updated

Agents evolve. Business rules change. A spec that was accurate at launch and has never been updated is actively dangerous, because the development team will make changes based on the written spec while the running agent behaves according to its actual (diverged) state. Treat the spec like source code: version-controlled, reviewed, and updated whenever the agent changes.

### Pitfall 4: Skipping the Evaluation Suite

Writing a spec without building tests for its success and failure criteria gives you documentation but not confidence. The evaluation suite is what converts SDD from a writing exercise into an engineering practice. Without it, you have no systematic way to catch regressions when the model provider updates the underlying model.

## OpenSpec: SDD as a Framework

Everything discussed so far describes SDD as a discipline: a set of practices you apply manually. [OpenSpec]([github.com/Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec)) is an open-source framework that automates those practices. It turns the workflow into a structured, repeatable system that lives inside your project repository.

The core problem OpenSpec addresses is that AI coding assistants are powerful but unpredictable when requirements live only in chat history. That is exactly the failure mode from [Why AI Makes This Problem Worse](#Why%20AI%20Makes%20This%20Problem%20Worse). OpenSpec's response is to give every change its own folder of structured artifacts, agreed upon before any code is written.

### The Filesystem as Spec Store

OpenSpec's foundational design decision is that specs are files, not database records or platform-specific state. When you initialize OpenSpec in a project, it creates a directory structure that becomes the persistent record of the project's intent:

```
your-project/
├── openspec/
│   ├── project.md          ← persistent project worldview (see 10.2)
│   ├── AGENTS.md           ← persistent agent behavioral rules
│   ├── specs/              ← accumulated specifications (grows over time)
│   │   └── auth/
│   │       └── spec.md
│   └── changes/            ← active and archived change proposals
│       ├── add-dark-mode/  ← one folder per in-flight change
│       │   ├── proposal.md
│       │   ├── specs/
│       │   ├── design.md
│       │   └── tasks.md
│       └── archive/        ← completed changes moved here
│           └── 2025-01-23-add-dark-mode/
```

This structure is tool-agnostic. It is plain Markdown and directories. Any AI assistant that can read files can consume it. Any developer can read it without special tooling. The spec is not locked inside a platform.

### Persistent Project Context

Two files in the OpenSpec structure map directly to the SDD concepts from earlier sections.

**`openspec/project.md`** is the persistent "worldview" document. Developers should exhaustively describe the tech stack (specifying versions to prevent AI from using outdated syntax) and architecture patterns (for example: "all database access must go through the Repository layer; Controller direct queries are strictly forbidden").

This is the equivalent of the Role and Scope section from [Role and Scope](#Role%20and%20Scope), but project-wide rather than per-agent. It establishes the invariants that every change must respect.

**`openspec/AGENTS.md`** contains behavioral rules for the AI assistants working in the project. This maps to the Behavioral Rules section from [Behavioral Rules](#Behavioral%20Rules), but at the project level rather than the feature level.

Together, these two files solve a problem that individual per-agent specs do not: they provide context continuity across sessions. A fresh AI context window that reads `project.md` and `AGENTS.md` before starting work has the same foundational constraints as every previous session.

### The Change Artifact Lifecycle

The core workflow unit in OpenSpec is a **change**: a self-contained folder representing one feature, fix, or modification. Each change contains four artifact types that map to the SDD sections we defined:

```
┌───────────────────────────────────────────────────────────────┐
│                  OpenSpec Change Artifacts                    │
│                                                               │
│  proposal.md  →  WHY: motivation, what is changing, scope     │
│  specs/       →  WHAT: requirements, scenarios, constraints   │
│  design.md    →  HOW: technical approach, architecture        │
│  tasks.md     →  SEQUENCE: ordered implementation checklist   │
└───────────────────────────────────────────────────────────────┘
```

This maps directly to the anatomy of a spec from [The Anatomy of a Good Spec for AI](#The%20Anatomy%20of%20a%20Good%20Spec%20for%20AI):

| SDD Section              | OpenSpec Artifact  |
|--------------------------|--------------------|
| Role and Scope           | proposal.md        |
| Inputs, Outputs, Tools   | specs/             |
| Behavioral Rules         | specs/             |
| Technical approach       | design.md          |
| Success/Failure Criteria | specs/ (scenarios) |
| Implementation steps     | tasks.md           |

The separation matters. It prevents a common failure where implementation details contaminate the requirement definition. A developer reading `specs/` should understand what the feature must do without needing to understand how it will be done. `design.md` handles the how.

### The Command Workflow

OpenSpec installs slash commands into your AI coding assistant (Claude Code, Cursor, Windsurf, and others). The commands drive you through the SDD lifecycle:

```
┌────────────────────────────────────────────────────────────────────┐
│                    Core OPSX Workflow                              │
│                                                                    │
│  /opsx:propose "add dark mode"                                     │
│       │                                                            │
│       ▼                                                            │
│  Creates openspec/changes/add-dark-mode/ with all four artifacts   │
│  (proposal.md, specs/, design.md, tasks.md) from a single prompt   │
│       │                                                            │
│       ▼ (human reviews and edits artifacts before proceeding)      │
│                                                                    │
│  /opsx:apply                                                       │
│       │                                                            │
│       ▼                                                            │
│  AI works through tasks.md line by line, implementing the change   │
│  using specs/ and design.md as the authoritative constraint set    │
│       │                                                            │
│       ▼                                                            │
│  /opsx:archive                                                     │
│       │                                                            │
│       ▼                                                            │
│  Merges delta specs into openspec/specs/                           │
│  Moves change folder to openspec/changes/archive/                  │
└────────────────────────────────────────────────────────────────────┘
```

The human review step between `propose` and `apply` is where the SDD discipline actually lives. The AI drafts the spec; a human verifies it is correct before authorizing implementation. This is the "agree before you build" gate.

For teams that want finer-grained control, an expanded workflow breaks `propose` into incremental steps:

```
/opsx:new       → creates the change folder with an empty scaffold
/opsx:continue  → creates the next artifact in dependency order
/opsx:ff        → fast-forwards to create all artifacts at once
/opsx:verify    → validates that implementation matches the spec
/opsx:sync      → updates spec files without archiving
```

Traditional workflows force you through phases: planning, then implementation, then done. The OPSX approach treats these as fluid actions rather than rigid phase gates, so you can update any artifact at any time.

### Delta Specs: Tracking What Changed

One of the more useful concepts in OpenSpec that has no equivalent in hand-rolled SDD is the **delta spec**. When you create a change, the specs inside `openspec/changes/add-dark-mode/specs/` are not standalone documents; they are deltas against the accumulated specs in `openspec/specs/`.

Delta specs show what is changing relative to your current specs, using sections to indicate the type of change. A delta spec for an authentication change looks like this:

```markdown
# Delta for Auth

## ADDED Requirements

### Requirement: Two-Factor Authentication
The system MUST require a second factor during login.

#### Scenario: OTP required
- GIVEN a user with 2FA enabled
- WHEN the user submits valid credentials
- THEN an OTP challenge is presented

## MODIFIED Requirements

### Requirement: Session Timeout
The system SHALL expire sessions after 30 minutes of inactivity.
(Previously: 60 minutes)

## REMOVED Requirements

### Requirement: Remember Me
(Deprecated in favor of 2FA)
```

This matters most in a team context. When multiple changes are in flight simultaneously, the delta format makes it possible to detect conflicts at the spec level. When running bulk archive, OpenSpec checks for spec conflicts. If two changes both touch the same spec files, it can inspect the codebase to resolve them.

Without delta specs, two developers working on overlapping features can produce two separate spec documents that silently contradict each other. The delta model surfaces the conflict before it becomes a code conflict.

### Tool Integration and Portability

OpenSpec installs workflow artifacts based on selected workflows and profiles. When selected by profile and workflow configuration, OpenSpec generates skills and slash commands for the target AI tool. The installation generates tool-native files: `.claude/skills/` for Claude Code, `.cursor/` for Cursor, `.github/prompts/` for GitHub Copilot, and so on.

The same `openspec/` directory works with whichever assistant a developer uses. A team where one person uses Claude Code and another uses Cursor is working from the same spec artifacts; only the command surface differs.

### Where OpenSpec Fits in the SDD Picture

OpenSpec does not replace the thinking described in this document. It automates the scaffolding and lifecycle management around it. The judgment work still belongs to humans: deciding what a spec should say, reviewing the AI-generated proposal before applying it, and evaluating whether the implementation matches the intent.

```
┌──────────────────────────────────────────────────────────────────────────┐
│             SDD Practice vs. OpenSpec                                    │
│                                                                          │
│  SDD Practice (this document):                                           │
│    Define role, scope, inputs, outputs, tools, rules, criteria.          │
│    Derive prompt. Enforce constraints in code. Write eval tests.         │
│    Keep spec versioned and synchronized with the running agent.          │
│                                                                          │
│  OpenSpec (the framework):                                               │
│    Provides the folder structure, file conventions, and slash            │
│    commands that enforce the SDD lifecycle as a repeatable ritual.       │
│    Handles artifact scaffolding, delta tracking, archiving,              │
│    and multi-tool delivery.                                              │
│                                                                          │
│  What OpenSpec does NOT replace:                                         │
│    Human review of generated proposals.                                  │
│    Constraint enforcement in tool wrapper code (How the Spec Flows).     │
│    Evaluation suites that test agent behavior (Complete Worked Example). │
│    Judgment about whether a spec is complete and correct.                │
└──────────────────────────────────────────────────────────────────────────┘
```

For teams adopting SDD for the first time, OpenSpec is a useful forcing function: it is harder to skip the spec step when the tooling actively scaffolds and expects it. For teams already practicing SDD manually, it provides a consistent structure that makes specs machine-readable and easier to audit over time.
