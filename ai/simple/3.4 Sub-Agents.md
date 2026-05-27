---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-02-23
reference:
  - https://docs.claude.com/
  - https://docs.claude.com/en/docs/claude-code/sub-agents
  - https://platform.claude.com/docs/en/agent-sdk/subagents
  - https://shipyard.build/blog/claude-code-subagents-guide/
  - https://dev.to/bhaidar/the-task-tool-claude-codes-agent-orchestration-system-4bf2
---
## What Are Subagents?

Subagents are specialized Claude instances that your main agent spawns to handle focused subtasks in separate, isolated contexts. Each one runs in its own context window with a custom system prompt, dedicated tool access, and independent permissions. When Claude encounters a task matching a subagent's description, it delegates automatically. Results return to the main conversation once the subagent finishes.

### Core concept: "Delegate and isolate"

Rather than cram every operation into one ever-growing context window, you delegate focused work into subagents so each one carries only the context it actually needs. Exploration noise, verbose test output, and domain-specific instructions live in the subagent's context, not yours.

### How Claude Code invokes subagents

In Claude Code, subagents run on top of the Task-based orchestration layer, so `Task` must appear in the orchestrator’s `allowedTools` for delegation to work.

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
             ▼                      ▼
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

## Built-in Subagents

Claude Code includes three built-in subagents that Claude invokes automatically. They inherit the parent session's permissions plus any additional tool restrictions.

### Explore

Explore is a fast, read-only codebase search agent used for file discovery and code understanding without making changes.

```
Purpose:  File discovery, code search, codebase understanding
Tools:    Read, Grep, Glob  (no write or execute)
Invoked:  When Claude needs to search without making changes
Modes:    quick | medium | very_thorough  (Claude chooses)
```

Delegating to Explore keeps codebase-scan output out of the main context. The orchestrator receives only the relevant excerpt.

### Plan

Plan is a research-only agent used inside plan mode to gather context before Claude proposes a plan to the user. Cannot spawn further subagents (no Task tool).

```
Purpose:  Pre-plan codebase research
Invoked:  Automatically when user is in plan mode
Nesting:  Prevented, plan subagent cannot spawn sub-subagents
```

### General-purpose

The general-purpose agent handles multi-step tasks that require both exploration and modifications, and is available whenever the Task layer is enabled.

```
Purpose:  Complex research, multi-step operations, code changes
Invoked:  When the task needs exploration + action + reasoning
```

The built-in general-purpose agent is always available whenever `Task` is in `allowedTools`, even without defining any custom agents.

### Helper agents (invoked automatically)

Beyond the three named agents, Claude Code includes additional helper agents for formatting, linting, and other specific tasks. You do not call them directly.

## Creating Custom Subagents

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

| Field             | Required | Description                                                                                                                                                        |
| ----------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `name`            | Yes      | Identifier used in logs and explicit invocation                                                                                                                    |
| `description`     | Yes      | Natural-language trigger; Claude reads this to decide when to delegate                                                                                             |
| `tools`           | No       | Allowlist. Omit to inherit all tools from parent                                                                                                                   |
| `disallowedTools` | No       | Denylist. Takes precedence over inherited tools                                                                                                                    |
| `model`           | No       | `sonnet` \| `opus` \| `haiku` \| `inherit`. Defaults to `inherit`                                                                                                  |
| `permissionMode`  | No       | `default` \| `acceptEdits` \| `bypassPermissions` \| `plan`. Overrides the session-level permission mode for this subagent only                                    |
| `maxTurns`        | No       | Cap the agent's turn limit                                                                                                                                         |
| `skills`          | No       | Preload specific skills into the subagent at startup                                                                                                               |
| `hooks`           | No       | Inline hooks configuration scoped only to this subagent. Uses the same structure as `.claude/settings.json` hooks. See [Hooks for Subagents](#hooks-for-subagents) |
| `memory`          | No       | Controls whether the subagent reads/writes MEMORY.md files. See [Memory settings](#memory-settings) below                                                          |
| `background`      | No       | `true` to always run as a background task                                                                                                                          |

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

Prefer a tight allowlist plus targeted denylist entries for high-risk tools; this keeps accidental destructive commands out of reach even if prompts go off-script.

#### Memory settings

The `memory` field controls whether the subagent reads and writes MEMORY.md files at startup and shutdown.

```yaml
memory:
  read: true    # load MEMORY.md into context at startup (default: true)
  write: true   # allow the subagent to update MEMORY.md on completion (default: true)
```

Setting `write: false` is useful for read-only or exploratory agents that should not modify persistent memory. Setting both to `false` gives the subagent a fully amnesiac context with no memory I/O.

#### Inline hooks

The `hooks` frontmatter field embeds hook configuration directly in the agent file instead of in `.claude/settings.json`. The structure is identical to the session-level hooks format:

```yaml
---
name: verified-implementer
description: Implements features and verifies tests pass before finishing
tools: Read, Edit, Write, Bash
hooks:
  Stop:
    - hooks:
        - type: command
          command: "npm test 2>&1 | tail -20"
---
```

Inline hooks take effect only when this subagent runs. They do not affect the main session or other subagents.

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

## Invoking Subagents

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

Each subagent gets its own context window and tool budget, and returns results independently.

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

## Subagent Result Format

When a subagent finishes, the Task tool delivers its output to the orchestrator as a plain string: the text of the subagent's final assistant turn.

In the SDK stream, this appears as a `tool_result` block paired with the original `tool_use` block for the `Task` call:

```
orchestrator message stream
  └── tool_use  { id: "tu_abc", name: "Task", input: { subagent_type: "code-reviewer", ... } }
  └── tool_result { tool_use_id: "tu_abc", content: "<subagent's final text output>" }
```

The result is always a string. There is no structured schema enforced by the runtime. If you need structured output (JSON, a pass/fail signal, a file path), instruct the subagent explicitly in its system prompt:

```yaml
---
name: test-runner
description: Runs the test suite and returns structured results
tools: Bash, Read
---

You are a test runner. After running the tests, you MUST respond with only
a JSON object in this exact shape and nothing else:

{
  "passed": <number>,
  "failed": <number>,
  "errors": [{ "test": "<name>", "message": "<error>" }]
}
```

The orchestrator can then parse `message.result` as JSON. Without this contract, the subagent may return a human-readable summary that is hard to process programmatically.

## Token and Cost Implications

Each subagent invocation is a separate API call with its own context window, which has direct cost consequences.

```
Main session:       [system prompt + conversation history]        → billed
Subagent A:         [subagent system prompt + task + tool calls]  → billed separately
Subagent B:         [subagent system prompt + task + tool calls]  → billed separately
```

A parallel workload with three subagents and one orchestrator produces four concurrent billing streams, one for each active context. Each subagent's input token count includes its full system prompt, the task description passed by the orchestrator, and every tool result accumulated during its run. Long tool outputs (a grep over a large codebase, a full test log) can drive up token counts fast.

Practical controls:

- **Narrow the tools list.** A subagent with only `Read` and `Grep` cannot accumulate Bash output. If the task does not need execution, remove Bash.
- **Set `maxTurns`.** Uncapped subagents can run many tool-use rounds. A code reviewer probably needs 10-20 turns, not 100.
- **Use `haiku` for high-volume, low-complexity tasks.** A subagent that classifies files or extracts metadata does not need Sonnet.
- **Keep system prompts lean.** See [System prompt length vs. skills preloading](#system-prompt-length-vs-skills-preloading) below.
- **Return compact summaries.** Instruct subagents to return findings as structured summaries rather than quoting full file contents back to the orchestrator.

## Nesting Limits

Subagents cannot spawn other subagents. This keeps subagent topologies flat and makes it easier to reason about cost and failure modes.

```
Allowed:
  Orchestrator  →  Task(subagent-A)
  Orchestrator  →  Task(subagent-B)

Not allowed:
  Orchestrator  →  Task(subagent-A)  →  Task(sub-sub-agent)  ✗
```

The exception is the built-in `Plan` subagent. It is explicitly blocked from calling `Task` even if `Task` appears in the parent's `allowedTools`.

If you need sustained parallelism or work that exceeds a single context window, use **Agent Teams** instead of nested subagents.

## Agent Teams

Agent Teams coordinate multiple Claude Code sessions that need to share state or hand off work across session boundaries. They differ from subagents in lifetime and communication model:

| | Subagents | Agent Teams |
|---|---|---|
| Lifetime | Single session | Persist across sessions |
| Communication | Return value only (string result) | Shared state / message passing |
| Nesting | Flat (one level) | Peer-to-peer or hierarchical |
| Use case | Isolated subtask within a session | Long-running workflows, pipelines |

Agent Teams are defined and coordinated at the application layer, not inside `.claude/agents/`. Each agent is typically a full Claude Code session, and shared state lives in external storage (files, a database, a message queue) that each session reads and writes.

A minimal pattern: a dispatcher session writes task definitions to a shared JSON file, and worker sessions poll for and claim tasks by updating that file. A `SubagentStop` hook or a separate orchestration loop routes results back.

Use Agent Teams when a single session's context window is too small for the work, when work must survive session restarts, or when agents need to coordinate as peers rather than through a single blocking orchestrator.

## Subagents + Skills

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

In a regular session, skill descriptions load at startup but bodies load on demand. With a preloaded subagent, **full skill content is injected at turn 1**. This avoids a round-trip to read the skill file when you need the subagent ready immediately.

### Decision: which direction?

Do you have a specific task to execute?
- Yes → Skill with context: fork
- No  → continue
Do you need domain knowledge available from turn 1?
- Yes → Subagent with skills: [list]
- No  → Subagent (skills load on demand via Skill tool when subagent triggers them)

### System prompt length vs. skills preloading

Every token in a subagent's system prompt is billed on every invocation. A 4,000-token system prompt on a subagent called 50 times costs 200,000 input tokens before the subagent does any actual work.

The `skills` frontmatter field handles this. Instead of embedding large blocks of conventions, style guides, or domain knowledge directly in the system prompt, extract them into skill files and let the subagent load them on demand:

```
Without skills preloading:
  system prompt = role instructions + 3,000 tokens of Python style guide
  → billed 3,000 tokens on every invocation, even if the subagent
    only reformats a one-line comment

With skills preloading (skills: [python-style-guide]):
  system prompt = role instructions  (short)
  skill content injected at turn 1 via Skill tool call
  → same effective context, but the skill body is loaded once
    per session, not once per invocation
```

Use direct embedding in the system prompt for:
- Short role definitions (under ~300 tokens)
- Instructions that must be available before the first tool call
- Rules that cannot be expressed as a reusable skill

Use `skills` preloading for:
- Style guides, coding conventions, or domain references over ~500 tokens
- Content shared across multiple subagents (define once, load anywhere)
- Content that may need to be updated independently of the agent definition

On Windows, the combined system prompt passed through the command line is capped at 8,191 characters. Skill preloading sidesteps this limit because skill content is injected inside the conversation, not as part of the initial command.


## Worktree Isolation

Subagents support **worktree isolation**: each subagent runs in its own temporary git worktree, which prevents concurrent file conflicts.

```yaml
---
name: feature-implementer
description: Implements features in an isolated worktree
isolation: worktree
---
```

From the CLI you can also pass `--worktree` (`-w`) when starting Claude to scope the entire session to a worktree.

Worktree isolation is useful for:
- Parallel implementation of independent features
- Running speculative experiments without touching the main working tree
- Safe batch modifications that need review before merging

## SDK Programmatic Definition

When building agents with the Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`), subagents can be defined programmatically via the `agents` parameter. No filesystem files required.

In SDK contexts you’ll see the Agent tool name used for subagent invocation; it sits on top of the same Task-style orchestration engine used by Claude Code.

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

**Do not include `Task` in a subagent's `tools` array.** Subagents cannot spawn sub-subagents.

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

## Error Handling

When a subagent fails, the Task tool returns an error result to the orchestrator as a `tool_result` with `is_error: true` and a description of what went wrong. The main conversation continues. The orchestrator decides whether to retry, fall back, or surface the failure.

Common failure modes:

| Failure | What the orchestrator sees | Notes |
|---|---|---|
| `maxTurns` reached | Result string with partial output | Subagent stopped after hitting the turn cap. Partial work may still be useful. |
| Tool call denied | Error result describing the denied tool | Usually a permission not pre-approved for a background subagent. Resume in foreground to retry. |
| Model error / timeout | Error result with provider error message | Transient. Retry with the same `sessionId` if resumable, or re-invoke with a fresh call. |
| Subagent explicitly gives up | Normal result string stating it could not complete | Not an error at the protocol level. Detect by inspecting the result text or using a `SubagentStop` prompt hook. |

In the SDK, check `is_error` on the `tool_result` block to distinguish a failed subagent from one that returned an empty result:

```typescript
for await (const message of query({ prompt, options })) {
  if ("message" in message && message.message.content) {
    for (const block of message.message.content) {
      if (block.type === "tool_result" && block.is_error) {
        console.error("Subagent failed:", block.content);
        // decide whether to retry or abort
      }
    }
  }
  if ("result" in message) console.log(message.result);
}
```

Use the `SubagentStop` prompt hook to add LLM-evaluated verification on top of the protocol-level error check. The hook can inspect the subagent's output and return `{"decision": "block", "reason": "..."}` if the work is incomplete even when no protocol error was raised.

## Resuming Subagents

Subagents can be resumed from where they stopped: full conversation history and all prior tool calls are preserved.

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

## Hooks for Subagents

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

Hooks can spawn a subagent for verification tasks that need file access or command execution:

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

Agent hooks use the same `ok` / `reason` response format as prompt hooks, but default timeout is 60s with up to 50 tool-use turns.

## Detecting Subagent Invocations (SDK)

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

## Troubleshooting

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

On Windows, subagents with very long system prompts may fail due to the 8,191-character command-line limit. Keep prompts concise or use filesystem-based agents for complex instructions.

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