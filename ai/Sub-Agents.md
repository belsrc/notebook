---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-02-23
reference:
  - https://docs.claude.com/
  - https://docs.claude.com/en/docs/claude-code/sub-agents
---
## Module 1: What Are Subagents?

Subagents are specialized Claude instances that your main agent can spawn to handle focused subtasks.
Each one runs in its own context window with a custom system prompt, dedicated tool access, and independent permissions.
When Claude encounters a task matching a subagent's description it delegates automatically; results return to the main conversation when the subagent finishes.

### Core concept: "Delegate and isolate"

Rather than cram every operation into one ever-growing context window, you push focused work into subagents. Exploration noise, verbose test output, and domain-specific instructions live in the subagent's context, not yours.

### How Claude Code invokes subagents

Subagents are called via the internal **Task tool**. The Task tool must therefore appear in `allowedTools` / `tools` for subagent delegation to be possible.

```
┌─────────────────────────────────────────────────────────┐
│ Main conversation (orchestrator)                        │
│                                                         │
│  User: "Review the auth module and run all tests"       │
│                                                         │
│  Claude: evaluates subagent descriptions …              │
│    → Task(subagent_type="code-reviewer", …)             │
│    → Task(subagent_type="test-runner", …)               │
└────────────┬──────────────────────┬─────────────────────┘
             │ own context window   │ own context window
             ▼                     ▼
┌────────────────────┐   ┌────────────────────────┐
│  code-reviewer     │   │  test-runner           │
│  tools: Read,Grep  │   │  tools: Bash,Read,Grep │
│  [reads 40 files]  │   │  [runs 200 tests]      │
│  returns: summary  │   │  returns: pass/fail    │
└────────────────────┘   └────────────────────────┘
             │                     │
             └──────────┬──────────┘
                        ▼
             Main context receives only
             the two compact summaries
```

### Subagents vs. other Claude Code customization

| Mechanism | Loaded | Context | Isolation | Invocation |
|---|---|---|---|---|
| `CLAUDE.md` | Every session | Shared | None | Always-on |
| **Skills** | On demand | Shared | None | Auto (description) or `/skill-name` |
| **Subagents** | On demand | Isolated | Full | Auto (description) or explicit name |
| Hooks | On tool events | N/A | N/A | Reactive, event-driven |

Skills inject instructions *into* the current conversation. Subagents spawn a *separate* instance and return only results.

---

## Module 2: Built-in Subagents

Claude Code ships three first-class built-in subagents that Claude invokes automatically. They inherit the parent session's permissions plus additional tool restrictions.

### Explore

A fast, read-only codebase search agent.

```
Purpose:  File discovery, code search, codebase understanding
Tools:    Read, Grep, Glob  (no write or execute)
Invoked:  When Claude needs to search without making changes
Modes:    quick | medium | very_thorough  (Claude chooses)
```

Delegating to Explore keeps codebase-scan output out of the main context. The orchestrator receives only the relevant excerpt.

### Plan

A research agent used inside **plan mode** to gather context before a plan is presented to the user. Cannot spawn further subagents (no Task tool).

```
Purpose:  Pre-plan codebase research
Invoked:  Automatically when user is in plan mode
Nesting:  Prevented, plan subagent cannot spawn sub-subagents
```

### General-purpose

A capable multi-step agent for tasks requiring both exploration and modification.

```
Purpose:  Complex research, multi-step operations, code changes
Invoked:  When the task needs exploration + action + reasoning
```

The built-in general-purpose agent is always available whenever `Task` is in `allowedTools`, even without defining any custom agents.

### Helper agents (invoked automatically)

Beyond the three named agents, Claude Code includes additional helper agents for formatting, linting, and other specific tasks. You do not call them directly.

---

## Module 3: Creating Custom Subagents

### Filesystem-based definition (Claude Code CLI)

Subagents are Markdown files with YAML frontmatter stored in `.claude/agents/`.

```
project-root/
└── .claude/
    └── agents/
        ├── code-reviewer.md
        ├── test-runner.md
        └── db-migration.md
```

Personal (cross-project) subagents live at `~/.claude/agents/`.

#### Minimum viable subagent

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices. Use for quality,
  security, and maintainability reviews.
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide specific,
actionable feedback on quality, security, and best practices.
```

#### All frontmatter fields

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Identifier used in logs and explicit invocation |
| `description` | Yes | Natural-language trigger; Claude reads this to decide when to delegate |
| `tools` | No | Allowlist. Omit to inherit all tools from parent |
| `disallowedTools` | No | Denylist. Takes precedence over inherited tools |
| `model` | No | `sonnet` \| `opus` \| `haiku` \| `inherit`. Defaults to `inherit` |
| `permissionMode` | No | Override permission mode for this subagent |
| `maxTurns` | No | Cap the agent's turn limit |
| `skills` | No | Preload specific skills into the subagent at startup |
| `hooks` | No | Hooks configuration scoped to this subagent |
| `memory` | No | Memory settings for this subagent |
| `background` | No | `true` to always run as a background task |

#### Tool allowlist vs denylist

```yaml
# Allowlist, only these tools
tools: Read, Grep, Glob, Bash

# Denylist on top of inherited, block specific tools
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit

# Both together, useful for narrowing MCP-extended tool sets
tools: Read, Bash
disallowedTools: Bash(rm *), Bash(sudo *)
```

### Creating subagents via the `/agents` command

The `/agents` interactive command is the recommended way to create and manage subagents:

```
/agents
  → View all available subagents (built-in, user, project, plugin)
  → Create new subagent with guided setup or Claude generation
  → Edit existing subagent configuration
  → Identify which subagents are active when duplicates exist
```

To list all subagents non-interactively:

```bash
claude agents  # prints agents grouped by scope
```

### Scope and priority hierarchy

```
┌─────────────────────────────────────────────────────────┐
│  Plugin subagents   (namespaced, no conflicts)          │  lowest
├─────────────────────────────────────────────────────────┤
│  Project subagents  .claude/agents/                     │
├─────────────────────────────────────────────────────────┤
│  User subagents     ~/.claude/agents/                   │
├─────────────────────────────────────────────────────────┤
│  Managed settings   (enterprise / policy)               │  highest
└─────────────────────────────────────────────────────────┘
```

When multiple subagents share the same name, the higher-priority scope wins. Use `claude agents` to see which definition is active.

---

## Module 4: Invoking Subagents

### Automatic invocation

Claude reads every subagent's `description` field and delegates automatically when a task matches.

**Write descriptions that are specific and action-oriented:**

```yaml
# Too vague, Claude may not delegate reliably
description: Helper for code tasks.

# Better, clear trigger conditions
description: Expert code review specialist. Use proactively for quality,
  security, and maintainability reviews on any code that was recently changed
  or is being prepared for a pull request.
```

Include "use proactively" to nudge Claude toward early delegation.

### Explicit invocation

Force a specific subagent by mentioning it by name in your prompt:

```
"Use the code-reviewer subagent to check the authentication module."
"Have the db-migration agent review this schema change."
```

### Foreground vs background execution

**Foreground (default):**
- Blocks main conversation until complete
- Permission prompts passed through to user
- AskUserQuestion calls work normally

**Background:**
- Runs concurrently while you continue working
- ALL permissions pre-approved before launch
- MCP tools unavailable
- AskUserQuestion tool calls silently fail
- Can be resumed in foreground if it needs input


Force background with frontmatter:

```yaml
background: true
```

Or press **Ctrl+B** to background an already-running task. Disable all background tasks globally:

```bash
CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1 claude
```

### Parallelism

Multiple subagents can run concurrently. Claude decides whether to run tasks in parallel based on dependency analysis. You can describe parallel intent in your prompt:

```
"Run the style-checker, security-scanner, and test-coverage agents simultaneously."
```

Each independent subagent gets its own context window, its own tool budget, and returns its result independently.

```
Main conversation
    │
    ├─── Task(style-checker)   ──→ runs concurrently ──→ returns style report
    ├─── Task(security-scanner) ─→ runs concurrently ──→ returns vuln report
    └─── Task(test-coverage)   ──→ runs concurrently ──→ returns coverage %
    │
    ▼
Claude synthesizes all three results
```

---

## Module 5: Nesting Limits

Subagents cannot spawn other subagents. This constraint is intentional, it prevents runaway nesting and unbounded context consumption.

```
Allowed:
  Orchestrator  →  Task(subagent-A)
  Orchestrator  →  Task(subagent-B)

Not allowed:
  Orchestrator  →  Task(subagent-A)  →  Task(sub-sub-agent)  ✗
```

The exception is the built-in `Plan` subagent, it is explicitly blocked from calling `Task` even if `Task` is in the parent's `allowedTools`.

If you need sustained parallelism or work that exceeds a single context window, use **Agent Teams** instead of nested subagents.

---

## Module 6: Subagents + Skills

Skills and subagents integrate in two directions.

### Direction 1: Skill with `context: fork` (runs in a subagent)

A skill can declare `context: fork` to execute its content inside an isolated subagent instead of the main conversation:

```yaml
---
name: deep-research
description: Research a topic thoroughly in isolation
context: fork
agent: Explore          # which subagent type to use
---

Research $ARGUMENTS thoroughly:
1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

The `agent` field accepts any built-in agent (`Explore`, `Plan`, `general-purpose`) or any custom subagent from `.claude/agents/`. Omitting `agent` defaults to `general-purpose`.

`context: fork` only makes sense for skills with explicit task instructions. A skill containing only conventions and guidelines (no task) would give the subagent instructions but nothing to do.

### Direction 2: Subagent with preloaded skills

Preload a skill into a subagent so the full skill body is injected at startup, unlike the normal progressive-disclosure model where skill content loads on demand:

```yaml
---
name: python-specialist
description: Expert Python coding agent with style conventions preloaded
tools: Read, Edit, Write, Bash
skills:
  - python-style-guide
  - python-testing
---

You are a Python specialist. Apply the preloaded style and testing conventions
to all code you write or review.
```

In a regular session, skill descriptions load at startup but bodies load on demand. In a preloaded subagent, **full skill content is injected immediately**, useful when you need the subagent ready without an extra round-trip to read the skill file.

### Decision: which direction?

Do you have a specific task to execute?
- Yes → Skill with context: fork
- No  → continue
Do you need domain knowledge available from turn 1?
- Yes → Subagent with skills: [list]
- No  → Subagent (skills load on demand via Skill tool when subagent triggers them)

---

## Module 7: Worktree Isolation

Subagents support **worktree isolation**, each subagent runs in its own temporary git worktree, preventing concurrent file conflicts:

```yaml
---
name: feature-implementer
description: Implements features in an isolated worktree
isolation: worktree
---
```

From the CLI you can also pass `--worktree` (`-w`) when starting Claude to scope the entire session to a worktree.

Worktree isolation is ideal for:
- Parallel implementation of independent features
- Running speculative experiments without touching the main working tree
- Safe batch modifications that need review before merging

---

## Module 8: SDK Programmatic Definition

When building agents with the Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`), subagents can be defined programmatically via the `agents` parameter, no filesystem files required.

### TypeScript example

```typescript
import { query, type AgentDefinition } from "@anthropic-ai/claude-agent-sdk";

const codeReviewer: AgentDefinition = {
  description:
    "Expert code review specialist. Use for quality, security, and maintainability reviews.",
  prompt: `You are a code review specialist with expertise in security,
performance, and best practices.

When reviewing code:
- Identify security vulnerabilities
- Check for performance issues
- Verify adherence to coding standards
- Suggest specific improvements`,
  tools: ["Read", "Grep", "Glob"],
  model: "sonnet",
};

const testRunner: AgentDefinition = {
  description:
    "Runs and analyzes test suites. Use for test execution and coverage analysis.",
  prompt: `You are a test execution specialist. Run tests and provide clear
analysis of results.

Focus on:
- Running test commands
- Analyzing test output
- Identifying failing tests
- Suggesting fixes for failures`,
  tools: ["Bash", "Read", "Grep"],
};

for await (const message of query({
  prompt: "Review the authentication module and run its tests",
  options: {
    // Task tool is required, subagents are invoked via Task
    allowedTools: ["Read", "Grep", "Glob", "Bash", "Task"],
    agents: {
      "code-reviewer": codeReviewer,
      "test-runner": testRunner,
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

### AgentDefinition fields (SDK)

| Field | Type | Required | Notes |
|---|---|---|---|
| `description` | `string` | Yes | Controls auto-invocation; be specific |
| `prompt` | `string` | Yes | The subagent's system prompt |
| `tools` | `string[]` | No | Allowlist. Omit to inherit parent tools |
| `model` | `'sonnet' \| 'opus' \| 'haiku' \| 'inherit'` | No | Defaults to main model if omitted |

**Do not include `Task` in a subagent's `tools` array**, subagents cannot spawn sub-subagents.

Programmatically defined agents take precedence over filesystem agents with the same name.

### Dynamic agent factory pattern

```typescript
const createSecurityAgent = (level: "strict" | "balanced"): AgentDefinition => ({
  description: "Security code reviewer",
  prompt: `You are a ${level === "strict" ? "strict" : "balanced"} security reviewer.
Apply ${level === "strict" ? "zero-tolerance" : "risk-weighted"} analysis.`,
  tools: ["Read", "Grep", "Glob"],
  // Use more capable model for high-stakes reviews
  model: level === "strict" ? "opus" : "sonnet",
});

for await (const message of query({
  prompt: "Review this PR for security issues",
  options: {
    allowedTools: ["Read", "Grep", "Glob", "Task"],
    agents: {
      "security-reviewer": createSecurityAgent("strict"),
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

---

## Module 9: Resuming Subagents

Subagents can be resumed to continue exactly where they left off, including full conversation history and all previous tool calls.

```
┌──────────────────────────────────────────────────────┐
│ First query                                          │
│   → subagent runs, returns agentId + sessionId       │
└─────────────────┬────────────────────────────────────┘
                  │ capture session_id and agentId
                  ▼
┌──────────────────────────────────────────────────────┐
│ Second query  (resume: sessionId)                    │
│   → prompt: "Resume agent <agentId> and …"           │
│   → subagent picks up from its previous state        │
└──────────────────────────────────────────────────────┘
```

### Transcript persistence

Subagent transcripts are stored separately from the main conversation:

| Behaviour | Details |
|---|---|
| Main conversation compaction | Does not affect subagent transcripts |
| Session restart | Transcripts persist; resume using the same `sessionId` |
| Automatic cleanup | Controlled by `cleanupPeriodDays` setting (default: 30 days) |

---

## Module 10: Hooks for Subagents

Two hook events are specific to subagent lifecycle:

### `SubagentStop`

Fires when a subagent finishes. Supports both command-based and prompt-based (LLM-evaluated) hooks:

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if this subagent completed its assigned task. Input: $ARGUMENTS\n\nCheck if:\n- The subagent completed its assigned task\n- Any errors occurred that need fixing\nReturn: {\"decision\": \"approve\" or \"block\", \"reason\": \"explanation\"}"
          }
        ]
      }
    ]
  }
}
```

### `PreToolUse` with Task matcher

Intercept subagent invocations before they start:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Task",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Subagent starting: $CLAUDE_TOOL_INPUT\" >> ~/.claude/subagent.log"
          }
        ]
      }
    ]
  }
}
```

### Agent-based hooks

Hooks themselves can spawn a subagent for verification tasks requiring file access or command execution:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results.",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

Agent hooks use the same `ok` / `reason` response format as prompt hooks, but default timeout is 60 s with up to 50 tool-use turns.

---

## Module 11: Detecting Subagent Invocations (SDK)

Subagents appear as `tool_use` blocks with `name: "Task"` in the message stream. Messages originating from within a subagent carry a `parent_tool_use_id` field.

```typescript
for await (const message of query({ prompt, options })) {
  // Detect when orchestrator spawns a subagent
  if ("message" in message && message.message.content) {
    for (const block of message.message.content) {
      if (
        block.type === "tool_use" &&
        block.name === "Task"
      ) {
        const subagentType = (block.input as Record<string, string>)
          .subagent_type;
        console.log(`Subagent spawned: ${subagentType}`);
      }
    }
  }

  // Detect messages from inside a subagent's execution context
  if ("parent_tool_use_id" in message && message.parent_tool_use_id) {
    console.log("  (message from within subagent)");
  }

  if ("result" in message) console.log(message.result);
}
```

---

## Module 12: Troubleshooting

### Claude is not delegating to subagents

```
Diagnosis checklist:
  1. Is Task in allowedTools?
       CLI:    Task tool is enabled by default
       SDK:    add "Task" to allowedTools in query options
  2. Is the description specific and action-oriented?
       Vague:  "A helper for code."
       Better: "Expert code reviewer. Use proactively on any code changes."
  3. Is the agent file in the correct directory?
       Project: .claude/agents/*.md
       User:    ~/.claude/agents/*.md
  4. Did you restart the session after adding a new file?
       Agents load at session start. Use /agents or restart.
  5. Are you explicitly naming the subagent?
       Try:  "Use the code-reviewer agent to …"
```

### Background subagent failing due to permissions

Background subagents pre-approve all needed tool permissions at launch. If a subagent needs a permission not pre-approved:

1. The tool call is auto-denied.
2. Resume the subagent in the foreground to retry interactively.

### Filesystem-based agents not loading

Subagent files are scanned once at session start. If you create or modify a `.md` file while a session is active, either:

```
/agents        ← loads immediately inside an active session
```
or restart the Claude Code session.

### Windows: prompt length limit

On Windows, subagents with very long system prompts may fail due to the 8 191-character command-line limit. Keep prompts concise or use filesystem-based agents for complex instructions.

---

## Quick Reference: Decision Matrix

Do you need instructions always present?
- Yes → CLAUDE.md
- No  → continue

Do you want to react to tool events?
- Yes → Hooks (.claude/settings.json)
- No  → continue

Do you need context isolation or parallelism?
- Yes → Subagent (.claude/agents/*.md)
- No  → Skill (.claude/skills/*/SKILL.md)

Do you need agents communicating across sessions?
- Yes → Agent Teams
- No  → Subagents (single session)

## Quick Reference: Subagent Authoring Checklist

```
Core quality
  [ ] description is action-oriented and specific
  [ ] description says "Use proactively" if appropriate
  [ ] description covers both "what" and "when"
  [ ] system prompt clearly states the subagent's role
  [ ] system prompt includes step-by-step behavior instructions

Tool access
  [ ] tools field is as narrow as possible for the task
  [ ] disallowedTools used where inherited tools need blocking
  [ ] Task is NOT in the subagent's own tools list
  [ ] Task IS in the orchestrator's allowedTools

Model selection
  [ ] sonnet for most tasks
  [ ] opus only for complex reasoning or high-stakes decisions
  [ ] haiku for simple, fast, high-volume tasks
  [ ] inherit when you want the subagent to mirror the main session

Background vs foreground
  [ ] background: true only when the subagent never needs user input
  [ ] MCP tools excluded from background subagents
  [ ] CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1 set if background is unwanted

Scope placement
  [ ] project .claude/agents/ for team-shared agents
  [ ] user   ~/.claude/agents/ for personal, cross-project agents
  [ ] plugin agents/ for distributable, namespaced agents

Testing
  [ ] tested with /agents to confirm the definition loads
  [ ] description tested with explicit invocation first
  [ ] parallel execution tested where applicable
  [ ] session resume tested if long-running continuity is required
```
