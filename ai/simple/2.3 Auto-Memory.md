---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-03-04
reference:
  - https://code.claude.com/docs/en/memory
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool
  - https://joseparreogarcia.substack.com/p/claude-code-memory-explained
---
### The Statelessness Problem

Each Claude Code session begins with a completely fresh context window.

There is no magic database, no long-term learning, and no mysterious internal state. Claude is, at its core, **a stateless function**. Its weights are frozen at inference time.

This creates a fundamental tension:

```
Session N                     Session N+1
┌───────────────────┐         ┌─────────────────┐
│ • Found that bug  │         │ • Empty context │
│   in thing.ts     │  ────►  │ • No memory     │
│ • pnpm build cmd  │  LOST   │ • Starts fresh  │
└───────────────────┘         └─────────────────┘
```

### `CLAUDE.md`/`AGENTS.md` vs `MEMORY.md`

The manual workaround was writing a `CLAUDE.md` file, a Markdown file _you_ author and maintain, containing persistent context that gets injected at session start.

`CLAUDE.md` is a markdown file in your project directory containing permanent context about your product, team, and preferences. Claude automatically reads it at session start and applies everything in it.

But Claude Code now has a **second, complementary mechanism**, `MEMORY.md`, which is the **auto-memory system**. Here's the critical distinction:

| CLAUDE.md     | MEMORY.md                           |
| ------------- | ----------------------------------- |
| You write it  | Claude writes it                    |
| Manual effort | Automatic                           |
| Rules/context | Learned insights                    |
| Permanent     | Evolves per session                 |
| Project root  | `~/.claude/projects/<repo>/memory/` |

### The Auto-Memory Directory Structure

Each project gets its own memory directory at `~/.claude/projects/<project>/memory/`. The `<project>` path is derived from the git repository, so all worktrees and subdirectories within the same repo share one auto memory directory.

The directory is structured like this:

```
~/.claude/projects/<repo>/memory/
├── MEMORY.md           ← Concise index, loaded every session
├── debugging.md        ← Detailed debugging patterns
├── api-conventions.md  ← API design decisions
└── ...                 ← Any topic files Claude creates
```

`MEMORY.md` is the **index file**, a table of contents pointing to detailed topic files. This is a deliberate architectural choice with a specific constraint.

### The 200-Line Constraint

The first 200 lines of `MEMORY.md` are loaded at the start of every conversation. Content beyond line 200 is not loaded at session start. Claude keeps `MEMORY.md` concise by moving detailed notes into separate topic files. This 200-line limit applies only to `MEMORY.md`. Topic files like `debugging.md` or `build.md` are not loaded at startup. Claude reads them on demand using its standard file tools when it needs the information.

This provides a two-tier access model:

```
Session Start
     │
     ▼
┌──────────────────────────────────────────────────────┐
│  MEMORY.md (first 200 lines, always loaded)          │
│  ┌───────────────────────────────────────────────┐   │
│  │ # Memory Index                                │   │
│  │ - debugging.md: TypedArray edge cases         │   │
│  │ - build.md: turbo pipeline flags              │   │
│  │ - arrow.md: schema encoding patterns          │   │
│  └───────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
     │
     │  On demand (when Claude needs details)
     ▼
┌─────────────────┐  ┌─────────────────┐  ┌──────────┐
│  debugging.md   │  │  build.md       │  │ other.md │
│  (full content) │  │  (full content) │  │   ...    │
└─────────────────┘  └─────────────────┘  └──────────┘
```

### What Claude Decides to Write

This is subtle and important. Claude doesn't write to memory every session indiscriminately. It decides what's worth remembering based on whether the information would be useful in a future conversation.

The types of things Claude autonomously captures include:

- Build commands and flags discovered during a session
- Debugging insights ("this error means X in our monorepo")
- Architecture notes and decisions made
- Code style preferences observed or discussed
- Workflow habits that were corrected

## Explicit Writes

To ask Claude to save something specific, tell it directly: "remember that we use pnpm, not npm" or "save to memory that the API tests require a local Redis instance".

You can also instruct it to forget: "Forget the rule about pnpm, we switched back to npm."

**`MEMORY.md` is plain Markdown you own and can edit freely.** Claude writes to it, but you are not locked out.

### The `/memory` Command

You can inspect and manage auto-memory directly in a Claude Code session:

```
/memory
```

This opens the memory management interface where you can:

- Toggle auto-memory on/off
- View current memory files
- Edit or delete entries

Auto memory is on by default. To toggle it, open `/memory` in a session and use the auto memory toggle, or set `autoMemoryEnabled` in your project settings.