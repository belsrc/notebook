---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-02-20
reference:
  - https://anthropic.skilljar.com/introduction-to-agent-skills
  - https://docs.claude.com/
  - https://docs.claude.com/en/docs/claude-code/skills
---
## What Are Skills?

Agent Skills are modular capabilities packaged as filesystem artifacts that Claude discovers and invokes autonomously. Each Skill is a directory containing a `SKILL.md` entry point plus optional supporting files.

### Core concept: "Teach once, reuse everywhere"

Rather than re-explaining a workflow every session, you encode it once in a Skill. Claude loads it on demand.

### Architecture

At startup, only the YAML frontmatter `name` and `description` fields from all Skills are pre-loaded into the system prompt. The full `SKILL.md` body and any referenced files are read on-demand via `bash` tool calls when a Skill is triggered. Executable scripts are _run_, not read, only their output enters the context window.

```
┌─────────────────────────────────────────────────────────┐
│ Startup                                                 │
│  All SKILL.md frontmatter → system prompt (metadata)    │
└─────────────────────────────────────────────────────────┘
         │  trigger matched
         ▼
┌─────────────────────────────────────────────────────────┐
│ Task begins                                             │
│  bash: read SKILL.md      → body enters context         │
│  bash: read FORMS.md      → only if needed              │
│  bash: run validate.py    → only output enters context  │
│  bash: read reference.md  → only if needed              │
└─────────────────────────────────────────────────────────┘
```

This **progressive disclosure** model means large references incur zero token cost until accessed.

### Skills vs. other Claude Code customization

|Mechanism|What it does|Invocation|
|---|---|---|
|`CLAUDE.md`|Always-on project instructions|Loaded every session|
|**Skills**|Reusable, domain-specific instructions|Auto (description match) or `/skill-name`|
|Hooks|Shell commands on tool events|Reactive, event-driven|
|Subagents|Isolated, expert Claude instances|Spawned by orchestrator|

Skills complement all three rather than replacing them.

## Creating Your First Skill

### Minimum viable Skill

```
.claude/skills/processing-pdfs/
└── SKILL.md
```

Every `SKILL.md` has two mandatory parts:

````yaml
---
name: processing-pdfs
description: Extract text and tables from PDF files, fill forms, merge documents.
  Use when working with PDF files or when the user mentions PDFs, forms, or
  document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:

/```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
/```
````

### Frontmatter field constraints

| Field | Max length | Allowed characters | Notes |
|---|---|---|---|
| `name` | 64 chars | `[a-z0-9-]` | Becomes the `/slash-command`; no `anthropic`/`claude` |
| `description` | 1024 chars | Any except XML tags | Written in **third person**; drives auto-trigger |

### Naming convention: gerund form

Prefer `processing-pdfs` over `pdf-processor` or `pdf`. The gerund form describes the *activity*, making the Skill's purpose immediately clear to both humans and Claude.

### Writing effective descriptions

The description is injected into the system prompt and is the *only* signal Claude uses for Skill selection when 100+ Skills are available. It must answer two questions: **what** does it do, and **when** should it trigger.

```yaml
# Good, specific, trigger-inclusive
description: Analyze Excel spreadsheets, create pivot tables, generate charts.
  Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.

# Bad, vague
description: Helps with documents
````

Always write in third person. First- or second-person descriptions ("I can help you…", "You can use this to…") cause discovery failures because the description is injected verbatim into the system prompt.

## Configuration and Multi-File Skills

### Progressive disclosure directory layout

```
pdf/
├── SKILL.md              # Overview + quick-start (≤ 500 lines)
├── FORMS.md              # Form-filling guide
├── reference.md          # Full API reference
├── examples.md           # Worked examples
└── scripts/
    ├── analyze_form.py   # Executed; output enters context
    ├── fill_form.py
    └── validate.py
```

Reference files at most **one level deep** from `SKILL.md`. Deeper nesting causes partial reads (`head -100` style), resulting in incomplete context.

### Degrees of freedom

Match instruction specificity to task fragility:

```
High freedom (text heuristics)    → multiple valid approaches exist
Medium freedom (pseudocode)       → preferred pattern, some variation
Low freedom (exact script/flags)  → fragile ops, exact sequence required
```

Example of low-freedom instruction for a fragile operation:

````markdown
## Database migration

Run exactly this script:

\```bash
python scripts/migrate.py --verify --backup
\````

Do not modify the command or add additional flags.
````

### Restricting tool access via frontmatter

```yaml
---
name: read-only-analysis
description: Analyze codebase structure. Use when the user asks to understand or audit code.
allowed-tools:
  - Read
  - Glob
  - Grep
---
````

This prevents Claude from exercising write or execute tools while the Skill is active. Note: `allowed-tools` frontmatter only applies in the Claude Code CLI. In the Agent SDK, use the `allowedTools` option in query configuration instead.

### Scripts: execute vs. read

Make the intent explicit in the Skill body:

```markdown
# Execute (preferred for deterministic utility scripts)
Run `python scripts/analyze_form.py input.pdf` to extract fields.

# Read (only when Claude needs the algorithm itself)
See `scripts/analyze_form.py` for the extraction algorithm.
```

Execution is preferred: the script code never enters the context window, only its stdout does.

### Token budget

Keep `SKILL.md` body under **500 lines**. Exceed this threshold and split into referenced files.

### `disable-model-invocation`

```yaml
---
name: deploy-production
description: Deploy the current build to production.
disable-model-invocation: true
---
```

Set this for Skills that should only be invoked explicitly via `/deploy-production`, not auto-triggered by Claude based on task context. Useful for destructive or high-stakes operations.

## Skills vs. Other Claude Code Features

### Decision matrix

```
┌─────────────────────────────────────────────────────────────────┐
│  Do you want instructions always loaded?                        │
│    Yes → CLAUDE.md                                              │
│    No  → continue                                               │
├─────────────────────────────────────────────────────────────────┤
│  Do you want to react to tool events (pre/post hooks)?          │
│    Yes → Hooks (.claude/settings.json)                          │
│    No  → continue                                               │
├─────────────────────────────────────────────────────────────────┤
│  Do you need an isolated agent with its own context + tools?    │
│    Yes → Subagent (.claude/agents/my-agent.md)                  │
│    No  → Skill (.claude/skills/my-skill/SKILL.md)               │
└─────────────────────────────────────────────────────────────────┘
```

### Subagents vs. Skills

| Dimension         | Skills                                                   | Subagents                                                |
| ----------------- | -------------------------------------------------------- | -------------------------------------------------------- |
| Invocation        | Inline, adds context to current conversation            | Spawns a new isolated Claude instance                    |
| Context isolation | None, shares the current context window                 | Full isolation, separate context window                  |
| State             | Stateless instruction injection                          | Can maintain its own turn-by-turn state                  |
| Ideal use         | Reference conventions, coding patterns, domain knowledge | Expert delegation, parallel work, sensitive tool scoping |

### Skills + Subagents together

Wire a Skill into a subagent to give it specialized domain knowledge with an isolated execution environment. The subagent's `AGENT.md` can explicitly reference a Skill, or the subagent's allowed tools can include `Skill`.

## Sharing Skills

### Scope hierarchy and priority

```
Enterprise managed settings  (highest priority)
  └── ~/.claude/skills/          (personal/user)
        └── .claude/skills/      (project, shared via git)
              └── plugin skills  (namespaced, no conflicts)
```

When Skills share the same name, higher-priority scope wins. Plugin Skills are always namespaced (`plugin-name:skill-name`) and never conflict.

### Team distribution: commit to repository

The simplest team-sharing mechanism, commit `.claude/skills/` to the project repo. All team members inherit Skills automatically when they clone or pull.

```
my-project/
├── .claude/
│   └── skills/
│       ├── reviewing-prs/
│       │   └── SKILL.md
│       └── writing-tests/
│           └── SKILL.md
└── src/
```

### Broader distribution: plugins

Package Skills (plus agents, hooks, MCP servers) into a plugin for external distribution.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json        # manifest: name, description, version
└── skills/
    └── code-review/
        └── SKILL.md
```

`plugin.json` minimum:

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

Test locally with `--plugin-dir`:

```bash
claude --plugin-dir ./my-plugin
```

Plugin Skills are invoked as `/my-first-plugin:skill-name`.

### Enterprise deployment: managed settings

Deploy Skills organization-wide via Anthropic's enterprise managed settings. Managed settings have highest precedence in the scope hierarchy, they override user and project settings. This enforces organizational standards (coding conventions, security policies) without requiring opt-in by individual developers.

### Security considerations when consuming Skills

Treat third-party Skills like installing software:

- Audit all bundled files: `SKILL.md`, scripts, images
- Flag unexpected network calls or unusual file access patterns
- External URL fetches are high-risk (content could contain injected instructions)
- Malicious Skills can invoke bash, file operations, or code execution as tools
- Only install Skills from trusted sources; be extra careful in production systems with access to sensitive data

## Troubleshooting Skills

### Skill won't trigger

```
Diagnosis checklist:
  1. Is the `Skill` tool enabled?
       Claude Code CLI: enabled by default
       Agent SDK:       add "Skill" to allowedTools
  2. Does the description contain relevant keywords for the task?
  3. Is the SKILL.md in the correct directory?
       Project:  .claude/skills/*/SKILL.md
       Personal: ~/.claude/skills/*/SKILL.md
       Plugin:   skills/*/SKILL.md (at plugin root)
  4. Is the name field valid? (lowercase, no XML tags, ≤ 64 chars)
  5. Is the description field non-empty and ≤ 1024 chars?
```

Verify discovery:

```bash
ls .claude/skills/*/SKILL.md       # project skills
ls ~/.claude/skills/*/SKILL.md     # personal skills
```

### Priority conflicts

When two Skills share the same name, the higher-scope one wins:

```
enterprise > personal (~/.claude/) > project (.claude/) > plugin
```

Plugin Skills are always namespaced and cannot collide. If a Skill and a legacy command share a name (`.claude/commands/` vs `.claude/skills/`), the Skill takes precedence.

### YAML frontmatter errors

Common causes of malformed frontmatter:

```yaml
# Bad: XML tags in name
name: <my-skill>

# Bad: Reserved word
name: claude-helper

# Bad: Uppercase
name: ProcessingPDFs

# Bad: Empty description
description:

# Bad: First-person description (causes discovery problems)
description: I can help you process Excel files.
```

### Runtime errors in scripts

```
Pattern: solve, don't punt
  Bad:  open(path).read()            # crashes with FileNotFoundError
  Good: handle the exception, provide fallback, emit diagnostic output
```

Self-document all "magic" constants in scripts:

```python
# HTTP requests typically complete within 30 seconds
REQUEST_TIMEOUT = 30
```

MCP tool references must be fully qualified to prevent "tool not found" errors:

```markdown
# Bad
Use the bigquery_schema tool...

# Good
Use the BigQuery:bigquery_schema tool...
```

### Skills not found on `--add-dir` paths

Skills inside directories added via `--add-dir` are discovered automatically from `.claude/skills/` within that directory. However, `CLAUDE.md` files from `--add-dir` directories are **not** loaded by default, set `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` to enable them.

### Iterative debugging methodology (A/B model pattern)

Use two Claude instances to refine Skills:

```
Claude A (author) → writes/refines SKILL.md
      ↓
Claude B (agent)  → performs real tasks using the Skill
      ↓
  Observe gaps    → "Claude B forgot to filter test accounts"
      ↓
   Claude A       → refines: change "always filter" to "MUST filter"
      ↓
   Claude B       → re-test on real scenarios
      ↓
    repeat
```

Build evaluations **before** writing extensive Skill documentation:

```json
{
  "skills": ["processing-pdfs"],
  "query": "Extract all text from this PDF and save to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Reads PDF using an appropriate library",
    "Extracts text from all pages without missing any",
    "Saves to output.txt in a readable format"
  ]
}
```

## Quick Reference: Skill Authoring Checklist

```
Core quality
  [ ] description is specific, includes keywords, uses third person
  [ ] description covers both "what" and "when"
  [ ] SKILL.md body ≤ 500 lines
  [ ] supporting details in separate referenced files
  [ ] no time-sensitive information (or in a <details> "old patterns" block)
  [ ] consistent terminology throughout
  [ ] file references are one level deep from SKILL.md
  [ ] all file paths use forward slashes

Code and scripts
  [ ] scripts handle errors explicitly (no punting to Claude)
  [ ] all constants are documented (no magic numbers)
  [ ] required packages listed and verified available
  [ ] validation/feedback loops included for complex workflows
  [ ] execution vs. read intent is explicit in instructions

Testing
  [ ] ≥ 3 evaluations created before extensive authoring
  [ ] tested with Haiku, Sonnet, and Opus
  [ ] tested with real usage scenarios (not toy tests)
  [ ] team feedback incorporated
```