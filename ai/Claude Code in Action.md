---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-02-20
reference:
  - https://anthropic.skilljar.com/claude-code-in-action
  - https://docs.anthropic.com/en/docs/claude-code
---
# Course Summary

> Free course hosted on Anthropic's Skilljar academy. Targets software developers and teams integrating AI assistance into existing development workflows.

## Module Breakdown

### 1. What is Claude Code?

Foundational concepts covering the architecture of AI coding assistants — how they interact with codebases via tool integration, and the technical foundations enabling code analysis and modification.

- Introduction
- What is a coding assistant?
- Claude Code in action

### 2. Getting Hands On

Practical setup and initial use of Claude Code inside a real project.

- Claude Code setup
- Project setup
- Adding context
- Making changes

### 3. Controlling Context

Strategies for maintaining and scoping relevant context across long conversations to keep AI assistance accurate and focused.

### 4. Custom Commands

Building reusable automations and slash-commands to eliminate repetitive prompting for common development tasks.

### 5. MCP Servers with Claude Code

Extending Claude Code's capabilities by integrating external tools via the **Model Context Protocol (MCP)** — including browser automation and specialized dev workflows.

### 6. GitHub Integration

Setting up AI-assisted code review and automating GitHub workflows (PR reviews, issue triage, etc.).

### 7. Hooks and the SDK

The deepest technical section of the course:

|Lesson|Description|
|---|---|
|Introducing hooks|What hooks are and when to use them|
|Defining hooks|Hook configuration and lifecycle|
|Implementing a hook|Concrete implementation walkthrough|
|Gotchas around hooks|Edge cases and failure modes|
|Useful hooks|Practical, real-world examples|
|Another useful hook|Additional patterns|
|The Claude Code SDK|Programmatic control of Claude Code for custom tooling|

### 8. Wrapping Up

- Quiz on Claude Code
- Summary and next steps

---

## Module 1: What is Claude Code?

### What is a Coding Assistant?

A coding assistant is an AI agent that interacts with a codebase through a **tool-use system** rather than through passive text generation. The model is given access to a set of tools it can invoke in a loop — the **agent loop** — until it determines the task is complete.

```
┌─────────────────────────────────────────────────────────────┐
│                        Agent Loop                           │
│                                                             │
│   User Prompt ──► LLM ──► Tool Call ──► Tool Result         │
│                    ▲                         │              │
│                    └─────────────────────────┘              │
│                   (repeats until Stop condition)            │
└─────────────────────────────────────────────────────────────┘
```

The model never directly reads your filesystem or runs commands — it requests a tool invocation, and the host process executes it and returns results.

### Built-in Tools

Claude Code ships with a set of built-in tools covering the core operations needed for software development:

|Tool|Purpose|
|---|---|
|`Bash`|Execute shell commands|
|`Read` / `FileRead`|Read file contents|
|`Write` / `FileWrite`|Write file contents|
|`Edit` / `FileEdit`|Patch/replace sections of a file|
|`Glob`|Pattern-match files across the project|
|`Grep`|Search file contents via regex|
|`WebFetch`|Fetch a URL|
|`WebSearch`|Search the web|
|`TodoWrite`|Manage a structured task list|
|`Task`|Spawn a subagent for a sub-task|
|`AskUserQuestion`|Block and ask the user for input|

### What Claude Code Is (and Is Not)

Claude Code is an **agentic** tool — it will autonomously plan, reason, and execute multi-step tasks across many files. It is not simply autocomplete or a chat panel sitting beside an editor. It operates:

- In the **terminal** (`claude` CLI)
- As a **VS Code / Cursor extension**
- Via the **Claude Agent SDK** (programmatic, Python + TypeScript)
- In **GitHub Actions** (CI automation)

---

## Module 2: Getting Hands On

### Installation

```bash
# Via npm (auto-updates)
npm install -g @anthropic-ai/claude-code

# Via Homebrew (manual updates required)
brew install claude-code
# To update: brew upgrade claude-code

# Via WinGet (manual updates required)
winget install Anthropic.ClaudeCode
# To update: winget upgrade Anthropic.ClaudeCode
```

Start in any project directory:

```bash
cd /path/to/your/project
claude
```

First launch prompts for login (OAuth or API key). API keys on macOS are stored in the system Keychain.

### Project Setup and Context Loading

When Claude Code starts, it reads the following files to build its initial understanding of the project:

```
project-root/
├── CLAUDE.md                 ← Project-level instructions (auto-loaded)
├── .claude/
│   ├── settings.json         ← Project settings (committed, shared)
│   ├── settings.local.json   ← Local overrides (gitignored)
│   ├── commands/             ← Custom slash commands (legacy path, still works)
│   └── skills/               ← Skills / custom commands (preferred path)
└── ...
~/.claude/
    ├── settings.json         ← User-level settings (all projects)
    ├── CLAUDE.md             ← User-level instructions (all projects)
    └── skills/               ← User-level skills
```

`CLAUDE.md` is the primary mechanism for injecting persistent project context — coding standards, architectural decisions, team conventions, banned patterns, etc.

### @ Mentions and Context Injection

Within the REPL you can reference project resources directly:

```
> Refactor @src/utils/color.ts to use the new color space API
> What does @ARCHITECTURE.md say about the data flow?
> Run the tests for !`git diff --name-only HEAD~1`
```

|Syntax|Effect|
|---|---|
|`@<filepath>`|Inject file contents into context|
|`!<shell command>`|Execute command and inject output|
|`/add-dir <path>`|Add an additional directory to the search scope|

---

## Module 3: Controlling Context

### Why Context Management Matters

The model has a finite context window. Long sessions accumulate tool call results, file reads, and conversation history that eventually push out important early context. Claude Code handles this in two ways:

**Manual compaction:**

```
/compact
```

Summarizes the conversation history, replacing raw messages with a condensed summary while preserving important facts and the current task state.

**Automatic compaction:** When the context window is nearly full, Claude Code automatically compacts before continuing.

### Context Visibility

```
/context
```

Shows a breakdown of what is currently in context — skills, agents, slash commands, and a sorted token count — so you can see what's consuming space.

### Settings Precedence

When multiple settings sources are active, they merge with this priority (highest wins):

```
programmatic options
      ↓
local settings  (.claude/settings.local.json)
      ↓
project settings (.claude/settings.json  /  .mcp.json)
      ↓
user settings   (~/.claude/settings.json)
```

---

## Module 4: Custom Commands (Skills)

### What a Skill Is

A **skill** is a Markdown file that defines a reusable prompt template and optional metadata. Storing a skill is equivalent to storing a slash command — the two are unified:

```
.claude/skills/<name>/SKILL.md   →  /<name>
.claude/commands/<name>.md       →  /<name>   (legacy, still works)
~/.claude/skills/<name>/SKILL.md →  /<name>   (user-level, all projects)
```

Subdirectory nesting creates namespaced commands:

```
.claude/commands/frontend/component.md  →  /frontend:component
```

### Skill Frontmatter

```markdown
---
description: Run the full test suite and fix any failures
allowed-tools: Bash(npm test:*), Read, FileEdit
disable-model-invocation: true   # Only user can invoke; Claude won't auto-trigger
# user-invocable: false          # Only Claude can invoke (background knowledge)
---

Run `npm test` and fix any failing tests without changing the test assertions themselves.
```

|Frontmatter field|Effect|
|---|---|
|`description`|Text loaded into context so Claude knows the skill exists|
|`allowed-tools`|Restrict which tools the skill may use|
|`disable-model-invocation`|Prevents Claude from auto-invoking (use for destructive ops like `/deploy`)|
|`user-invocable: false`|Hides from slash-command menu; Claude uses it as background knowledge|

### Dynamic Context in Skills

Skills can inject live data via shell execution and file references:

```markdown
---
description: Create a well-formed git commit
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
---

## Context
- Current status: !`git status`
- Staged diff: !`git diff --cached`

## Task
Write a conventional commit message and commit the staged changes.
```

### Character Budget

Skill descriptions are loaded into the context window on startup. The budget is `max(16000, 2% of context window)` characters. Run `/context` to check for truncation warnings.

---

## Module 5: MCP Servers with Claude Code

### What MCP Is

The **Model Context Protocol (MCP)** is an open standard that defines how an AI agent communicates with external tool servers. An MCP server exposes a set of named tools over either `stdio` or HTTP. Claude Code treats MCP tools identically to built-in tools once a server is connected.

MCP tool names follow the pattern: `mcp__<server-name>__<tool-name>`

### Adding MCP Servers

```bash
# stdio transport (local process)
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://user:pass@host:5432/db"

# HTTP transport (remote server, with OAuth)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Interactive wizard
claude mcp add

# JSON config (for complex args)
claude mcp add-json myserver '{"command":"node","args":["./server.js"]}'
```

### MCP Server Scopes

```
┌─────────────────────────────────────────────────────────────────┐
│ Scope     │ Storage                   │ Shared?                 │
├─────────────────────────────────────────────────────────────────┤
│ local     │ ~/.claude.json (per-proj) │ No  (default)           │
│ project   │ .mcp.json                 │ Yes (commit to VCS)     │
│ user      │ ~/.claude/settings.json   │ No  (all your projects) │
└─────────────────────────────────────────────────────────────────┘
```

### Managing Servers

```bash
claude mcp list          # List all configured servers
claude mcp get github    # Inspect a specific server
claude mcp remove github # Remove a server

# Within the REPL:
/mcp                     # Check server connection status + authenticate OAuth
```

### OAuth Authentication

Many hosted MCP servers (GitHub, Slack, etc.) use OAuth 2.0. Claude Code handles the flow automatically when you run `/mcp` and select "Authenticate". For servers without dynamic client registration, pass `--client-id` and `--client-secret` at add time.

---

## Module 6: GitHub Integration

### Claude Code GitHub Action

The `anthropics/claude-code-action@v1` GitHub Action embeds Claude Code into your CI pipeline. It listens for `@claude` mentions in PR comments and issues, then performs the requested action and pushes results as commits or comments.

```yaml
# .github/workflows/claude.yml
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          claude_args: |
            --append-system-prompt "Follow the conventions in CLAUDE.md"
            --max-turns 20
            --model claude-sonnet-4-6
```

### Interaction Modes

|Mode|Trigger|Claude's action|
|---|---|---|
|`@claude` on PR|PR comment|Implements requested change, pushes commits|
|`@claude` on issue|Issue comment|Analyzes issue, may open a PR|
|Automatic|`direct_prompt` in workflow|Runs without a human @-mention|

### Automated Code Review

For a PR review workflow that runs automatically on every pull request:

```yaml
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Review this PR for correctness, security issues, and adherence to CLAUDE.md conventions. Post your review as a PR comment."
```

### Cloud Provider Auth (OIDC)

For workflows that need AWS/GCP access (e.g., deploying, querying prod databases), use OIDC rather than static credentials:

```yaml
# AWS
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::ACCOUNT:role/github-actions-role
    aws-region: us-east-1

# GCP
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

---

## Module 7: Hooks and the SDK

### What Hooks Are

Hooks are **user-defined shell commands** that execute at specific points in the Claude Code agent lifecycle. They provide **deterministic** side effects that bypass the LLM's decision-making — if you add a hook, it will always run at that lifecycle point, unconditionally.

> Hooks convert suggestions into application-layer enforcement.

### Hook Lifecycle Events

|Event|When it fires|Can block?|
|---|---|---|
|`SessionStart`|When the session begins|No|
|`UserPromptSubmit`|When the user submits a message|Yes (via `continue: false`)|
|`PreToolUse`|Before any tool call executes|Yes|
|`PostToolUse`|After a tool call succeeds|No|
|`PostToolUseFailure`|After a tool call fails|No|
|`Notification`|When Claude needs user attention|No|
|`PreCompact`|Before context compaction|No|
|`Stop`|When the agent loop exits|No|
|`SubagentStop`|When a subagent exits|No|

All matching hooks for a given event run **in parallel**. Identical hook commands are deduplicated automatically.

### Defining a Hook

Hooks are stored in `settings.json` (user or project level):

```jsonc
// ~/.claude/settings.json  (user-level, applies to all projects)
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "FileEdit|FileWrite",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs input\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

Configure via the interactive menu (`/hooks`) or by editing the JSON directly. Note: direct edits to `settings.json` do **not** take effect in the running session — restart Claude Code.

### Hook Communication Protocol

Claude Code passes JSON via **stdin** to the hook script. The script communicates back via **exit code** and optionally a JSON object on **stdout**:

```
┌──────────────┐  JSON (stdin)   ┌──────────────┐
│  Claude Code │ ──────────────► │  Hook Script │
│              │ ◄────────────── │              │
└──────────────┘  exit code +    └──────────────┘
                  JSON (stdout)
```

**Exit codes:**

|Code|Meaning|
|---|---|
|`0`|Success — proceed normally|
|`2`|Block the operation (PreToolUse only)|
|other|Error — logged, operation may still proceed|

**stdout JSON fields:**

```jsonc
{
  "continue": false,           // Stop the agent loop
  "stopReason": "Message",     // Shown to user when continue=false
  "decision": "block",         // Block the tool call (PreToolUse)
  "reason": "Feedback for Claude",
  "systemMessage": "Warning to user",
  "suppressOutput": true       // Hide stdout from transcript
}
```

### Practical Hook Examples

**Auto-format TypeScript after every edit:**

```bash
#!/usr/bin/env bash
# .claude/hooks/format.sh
INPUT=$(cat)
FILES=$(echo "$INPUT" | jq -r '.tool_input.path // empty')

if [[ "$FILES" == *.ts || "$FILES" == *.tsx ]]; then
  npx prettier --write "$FILES"
fi
```

**Log every Bash command executed:**

```bash
#!/usr/bin/env bash
INPUT=$(cat)
echo "$INPUT" | jq -r '"\(.tool_input.command) — \(.tool_input.description // "no desc")"' \
  >> ~/.claude/bash-command-log.txt
```

**Block writes to production config files:**

```bash
#!/usr/bin/env bash
INPUT=$(cat)
PATH_ARG=$(echo "$INPUT" | jq -r '.tool_input.path // empty')

if [[ "$PATH_ARG" == *"production"* || "$PATH_ARG" == *".env.prod"* ]]; then
  echo '{"decision":"block","reason":"Writes to production config are not permitted."}' 
  exit 2
fi
```

### Hook Types: command, prompt, agent

Beyond `"type": "command"` (shell script), hooks can use a Claude model to make decisions:

```jsonc
// Prompt-based hook: single-turn LLM evaluation
{
  "type": "prompt",
  "prompt": "Does this bash command look dangerous? Reply with JSON: {\"decision\": \"allow\" | \"block\", \"reason\": \"...\"}",
  "model": "claude-haiku-4-5-20251001"
}
```

```jsonc
// Agent-based hook: multi-turn LLM with tool access
{
  "type": "agent",
  "prompt": "Verify the SQL query does not perform a DROP or TRUNCATE.",
  "allowedTools": ["Bash"]
}
```

### Hook Security Considerations

Hooks run with the same credentials as the current user. Key rules:

- Always quote shell variables: `"$VAR"`, not `$VAR`
- Check for path traversal: reject paths containing `..`
- Use absolute paths — `$CLAUDE_PROJECT_DIR` is available for the project root
- Skip sensitive files: `.env`, `.git/`, key files
- Default timeout is 60 seconds per hook command
- Hooks from `settings.json` edits during a session are not hot-reloaded

---
##  The Claude Agent SDK

> **Note:** The Claude Code SDK was renamed to the **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`). Existing `@anthropic-ai/claude-code-sdk` imports continue to work via the migration shim.

### Core Concept

The SDK exposes the same agent loop that powers the CLI — but as a programmatic async generator you can embed in your own application:

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "What files are in this directory?",
  options: {
    allowedTools: ["Bash", "Glob"],
    permissionMode: "acceptEdits",
    cwd: "/path/to/project",
  }
})) {
  if (message.type === "result") {
    console.log(message.result);
  }
}
```

### Key Options

```typescript
type Options = {
  // Tool control
  allowedTools?: string[];          // Whitelist. Supports glob: "mcp__github__*"
  disallowedTools?: string[];       // Blacklist
  permissionMode?: PermissionMode;  // "default" | "acceptEdits" | "bypassPermissions" | "plan"

  // Session
  continueConversation?: boolean;   // Continue last session
  resume?: string;                  // Resume by session ID
  maxTurns?: number;                // Cap agent loop iterations

  // Context
  systemPrompt?: string | { preset: "claude_code"; append?: string };
  settingSources?: SettingSource[]; // Load filesystem config: "user" | "project" | "local"
  cwd?: string;

  // Model
  model?: string;                   // e.g., "claude-sonnet-4-6"
  maxBudgetUsd?: number;            // Hard cost cap

  // MCP
  mcpServers?: Record<string, McpServerConfig>;
};
```

### Permission Modes

```
"default"           → Standard prompting for sensitive ops
"acceptEdits"       → Auto-approve file edits; still prompts for Bash
"bypassPermissions" → No prompts at all (use only in controlled CI envs)
"plan"              → Read-only analysis; Claude cannot execute changes
```

### Settings Sources

By default the SDK **ignores** filesystem config (no `CLAUDE.md`, no `settings.json`). To load project config:

```typescript
options: {
  settingSources: ["project"],      // Load .claude/settings.json + CLAUDE.md
  systemPrompt: { preset: "claude_code" }  // Also required to activate CLAUDE.md
}
```

### MCP in the SDK

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "List the 5 most recent open issues in my-org/my-repo",
  options: {
    mcpServers: {
      github: {
        command: "npx",
        args: ["-y", "@modelcontextprotocol/server-github"],
        env: { GITHUB_TOKEN: process.env.GITHUB_TOKEN }
      }
    },
    allowedTools: ["mcp__github__list_issues"]
  }
})) {
  if (message.type === "result") console.log(message.result);
}
```

### Custom Tools (In-Process MCP Server)

Define tools directly in your application without running a separate process:

```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const server = createSdkMcpServer({
  name: "project-tools",
  version: "1.0.0",
  tools: [
    tool(
      "get_deploy_status",
      "Check the current deployment status of a service",
      { service: z.string().describe("Service name to query") },
      async (args) => {
        // call your internal API
        const status = await fetchDeployStatus(args.service);
        return { content: [{ type: "text", text: JSON.stringify(status) }] };
      }
    )
  ]
});

// Note: custom MCP tools require streaming input (AsyncIterable), not a plain string
async function* streamPrompt() {
  yield { type: "user", content: "What is the deploy status of the auth service?" };
}

for await (const message of query({
  prompt: streamPrompt(),
  options: { mcpServers: { "project-tools": server } }
})) {
  if (message.type === "result") console.log(message.result);
}
```

### Hooks in the SDK

```typescript
import { query, type HookInput } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Refactor the auth module",
  options: {
    hooks: {
      PostToolUse: [
        {
          hooks: [
            async (input: HookInput) => {
              // Log every tool execution
              console.log(`Tool used: ${(input as any).tool_name}`);
              return { continue: true };
            }
          ]
        }
      ]
    }
  }
})) { /* ... */ }
```

### Subagents in the SDK

Subagents are specialized agent instances with isolated context windows, allowing parallel execution and focused system prompts:

```typescript
const result = query({
  prompt: "Perform a full code review of the authentication module",
  options: {
    agents: [
      {
        name: "security-scanner",
        description: "Security vulnerability scanner. Use for auth, crypto, and input validation.",
        systemPrompt: "You are a security expert. Look for OWASP Top 10 issues only.",
        allowedTools: ["Read", "Grep", "Glob"]   // read-only
      },
      {
        name: "style-checker",
        description: "Code style and convention checker.",
        systemPrompt: "Check against the project's ESLint config and naming conventions.",
        allowedTools: ["Read", "Bash(npx eslint:*)"]
      }
    ]
  }
});
```

Claude decides which subagent to delegate to based on the `description` field. Multiple subagents can run **concurrently**, drastically reducing wall-clock time on multi-part tasks.

---

## Key Architectural Summary

```
┌───────────────────────────────────────────────────────────────────┐
│                        Claude Code Ecosystem                      │
│                                                                   │
│  ┌────────────┐  ┌──────────────┐  ┌───────────────┐              │
│  │  Terminal  │  │  VS Code /   │  │ GitHub Actions│              │
│  │    CLI     │  │   Cursor     │  │  (CI/CD)      │              │
│  └─────┬──────┘  └──────┬───────┘  └───────┬───────┘              │
│        └────────────────┴──────────────────┘                      │
│                          │                                        │
│               ┌──────────▼──────────┐                             │
│               │   Claude Agent SDK  │  (TypeScript / Python)      │
│               └──────────┬──────────┘                             │
│                          │                                        │
│         ┌────────────────┼─────────────────┐                      │
│         ▼                ▼                 ▼                      │
│   ┌──────────┐   ┌─────────────┐   ┌────────────┐                 │
│   │ Built-in │   │    Hooks    │   │    MCP     │                 │
│   │  Tools   │   │ (lifecycle) │   │  Servers   │                 │
│   └──────────┘   └─────────────┘   └────────────┘                 │
│                                                                   │
│   Context: CLAUDE.md + settings.json + Skills + Subagents         │
└───────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference: Important File Paths

```
~/.claude/
├── settings.json          User-level settings & hooks
├── CLAUDE.md              User-level persistent instructions
├── skills/                User-level skills (slash commands)
├── agents/                User-level subagent definitions
├── commands/              User-level legacy commands
└── output-styles/         User-level output style definitions

<project-root>/
├── CLAUDE.md              Project-level instructions (commit this)
├── .mcp.json              Shared MCP server config (commit this)
└── .claude/
    ├── settings.json      Project-level settings (commit this)
    ├── settings.local.json  Local overrides (gitignore this)
    ├── skills/            Project skills
    ├── agents/            Project subagents
    └── commands/          Legacy project commands
```

---

## Quick Reference: Common Slash Commands

|Command|Effect|
|---|---|
|`/help`|Show all available commands|
|`/context`|Show context breakdown with token counts|
|`/compact`|Summarize history to reclaim context|
|`/clear`|Clear conversation history|
|`/hooks`|Interactive hook configuration|
|`/mcp`|Show MCP server status / authenticate|
|`/model`|Switch active model|
|`/config`|Open configuration menu|
|`/add-dir <path>`|Add an extra directory to scope|
|`/resume`|Resume a previous session|
|`/export`|Export current conversation|
|`/output-style`|Switch output style (Default / Explanatory / Learning)|
|`/agents`|Manage subagents|
|`/vim`|Enable Vim keybindings|
|`/debug`|Claude-assisted session debugging|
