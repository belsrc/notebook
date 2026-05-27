---
tags:
  - ai
  - agents
gardening: 🌳
date: 2026-03-19
reference:
  - https://tacnode.io/post/stateful-vs-stateless-ai-agents-practical-architecture-guide-for-developers
  - https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/tool-based-agents-for-calling-functions.html
  - https://www.letta.com/blog/programmatic-tool-calling-with-any-llm
  - https://zapier.com/blog/human-in-the-loop/
  - https://www.youtube.com/watch?v=cmEJ-5zYKHA
---
## What the model actually does

Before distinguishing a skill from an agent, it helps to understand what the model itself does at the lowest level.

A language model behaves like a pure function at inference time: it accepts a sequence of text (the prompt) and produces a sequence of text (the completion). That is the whole mechanism. There is no memory between calls, no persistent state, no connection to the outside world. When you see “chat history,” that history is client‑side state your system passes back into each call; the model itself does not remember previous requests. Every call is stateless: the model receives everything it needs to know in the input, produces an output, and stops.

```
  ┌──────────────┐        ┌──────────────┐
  │ Input text   │──────► │   Model      │──────► Output text
  └──────────────┘        └──────────────┘
         (prompt)                                 (completion)
```

This is the atom from which everything else is built.

## A skill invocation: text in, text out

A **skill invocation** is a single call to the model with a carefully prepared prompt. The model produces a response, and execution stops. There is no loop, no access to external systems, and no ability to take action.

Consider a skill that drafts a commit message. The host system:

1. Reads the current `git diff`
2. Constructs a prompt: `"Given this diff: <diff>, write a conventional commit message."`
3. Calls the model
4. Receives the commit message as text

```
  ┌───────────────────────────────────────────┐
  │  Skill invocation                         │
  │                                           │
  │  prompt = system_prompt + diff_text       │
  │                         │                 │
  │                         ▼                 │
  │                    ┌─────────┐            │
  │                    │  Model  │            │
  │                    └────┬────┘            │
  │                         │                 │
  │                         ▼                 │
  │                   commit message          │
  │                   (text only)             │
  └───────────────────────────────────────────┘
```

The model does not run `git diff` itself. The host gathered that context, handed it over, and received text back. The model's output is a suggestion. Whether that suggestion becomes a commit depends entirely on what the host system does with the text.

A skill invocation cannot cause side effects on its own. The model has no direct authority over the environment; the host decides whether to turn its text output into an action.

## What is a tool?

A **tool** is a host‑implemented function (often an API or local capability) that the model can request via a structured tool call, and that the host executes on its behalf. Instead of the model only being able to produce plain text, it can signal: "I want to invoke function X with these arguments." The host process intercepts that signal, runs the function, and returns the result back to the model as more input.

Some representative tools from Claude Code:

| Tool | What it does |
|---|---|
| `Read` | Returns the contents of a file |
| `Bash` | Executes a shell command and returns stdout/stderr |
| `Write` | Writes content to a file on disk |
| `Grep` | Searches file contents with a regex pattern |
| `WebFetch` | Fetches a URL and returns its content |
| `Task` | Spawns a separate model instance (a subagent) |

The model never directly touches your filesystem, network, or shell. It emits a structured request. The host process decides whether to honor it, executes it, and feeds the result back.

```
  ┌────────────────────────────────────────────────────┐
  │  Tool call sequence                                │
  │                                                    │
  │  Model output: "call Read('src/auth.ts')"          │
  │                         │                          │
  │                         ▼                          │
  │               ┌───────────────────┐                │
  │               │   Host process    │                │
  │               │ executes Read()   │                │
  │               └────────┬──────────┘                │
  │                        │                           │
  │                        ▼                           │
  │  Model receives: contents of src/auth.ts           │
  └────────────────────────────────────────────────────┘
```

The set of tools available to a model invocation is always explicitly configured. A model instance only has access to the tools the host system chooses to give it. A read-only agent has `Read`, `Grep`, and `Glob`. A full agent has `Read`, `Write`, `Bash`, and more. This is not a convention; it is a hard boundary set by the host.

## An agent: the model in a loop

When a model has access to tools, a single call is no longer the whole picture. The model can:

1. Receive a task
2. Decide it needs more information (invoke a tool)
3. Receive the result
4. Do more reasoning
5. Invoke another tool
6. Repeat until it decides the task is complete

This is the **agent loop**: the model calls tools, receives results, and those results become new input that the model reasons over. The loop terminates when the model produces a final answer instead of a tool call.

```
  ┌───────────────────────────────────────────────────┐
  │  Agent loop                                       │
  │                                                   │
  │   Task ──► Model ──► Tool call                    │
  │             ▲              │                      │
  │             │              ▼                      │
  │             └──── Tool result                     │
  │                                                   │
  │   (repeats until model emits a final response,    │
  │    not a tool call)                               │
  └───────────────────────────────────────────────────┘
```

This loop is the defining feature of an agent. A skill invocation is one pass through the model, start to finish. An agent can make dozens of passes, gathering information and taking actions at each step.

### A concrete contrast

Suppose someone asks: "Does the `authenticate()` function handle the case where the token is expired?"

Skill invocation path: The host would need to find the function, extract the source, and insert it into a prompt before calling the model. The model produces text describing what it sees.

Agent path: The model is given the task directly. It calls `Grep` to find `authenticate`, calls `Read` to load the file, inspects the logic, possibly calls `Read` again on a related file to check how expiry errors propagate, and then produces a response. The model gathered all of that context itself through tool use.

The agent path requires no manual context preparation. The tradeoff is that it takes more time, uses more tokens, and can take consequential actions if its tools include write or execute capabilities.

## Why the distinction matters

The gap between a skill invocation and an agent is not just a technical detail. It changes the risk profile of the operation.

### Consequential actions

A skill that produces text cannot break anything. The worst outcome is bad text. An agent with write and execute tools can:

- Overwrite files
- Delete data
- Push commits
- Execute arbitrary shell commands
- Call external APIs

Each tool call is an action in the real world. When the loop runs dozens of times over many files, the aggregate effect can be significant and difficult to reverse. The failure mode is not bad text; it is a codebase in an inconsistent state, a deleted record, or an unintended deployment.

### Failure modes are different

| | Skill invocation | Agent |
|---|---|---|
| Worst case output | Incorrect or unhelpful text | Irreversible system change |
| Blast radius | Zero (text only) | Proportional to tool permissions |
| Reviewability | Review text before acting | Actions happen mid-loop |
| Debuggability | Single prompt/response pair | Multi-step trace required |
| Recovery | Discard the text | May require manual rollback |

### Trust and permission scope

Because agents can act, the tools available to a specific agent instance must be treated as a trust boundary. The principle is the same one used in access control: grant the minimum permissions needed for the task. A code review agent needs read access, not write. A migration agent needs write access to the target schema, not to application code.

Granting an agent broad permissions because it is "easier to set up" is the equivalent of giving a contractor a master key when they only need access to one room.

## Human-in-the-loop

A **human-in-the-loop** checkpoint is a point in an agentic workflow where execution pauses and a person must review and approve before the agent continues.

The need for a checkpoint is directly correlated with the reversibility and scope of the action about to be taken. Low-stakes, reversible actions can safely proceed without a checkpoint. High-stakes or irreversible actions require one.

### When a checkpoint is required

```
  ┌─────────────────────────────────────────────────────────┐
  │  Decision: does this action require a checkpoint?       │
  │                                                         │
  │  Is the action reversible?                              │
  │    Yes, easily (e.g. editing a local file) ─────────────── proceed
  │    No (e.g. dropping a table, pushing to main) ───────── CHECKPOINT
  │                                                         │
  │  What is the blast radius?                              │
  │    Single file ─────────────────────────────────────── proceed
  │    Multiple services or external systems ─────────────── CHECKPOINT
  │                                                         │
  │  Is this the first time this agent runs this task?      │
  │    No, well-tested in a controlled environment ───────── proceed
  │    Yes ──────────────────────────────────────────────── CHECKPOINT
  └─────────────────────────────────────────────────────────┘
```

### Checkpoints in practice

A checkpoint does not mean the agent stops permanently. It means the agent surfaces a plan or a proposed action for human review, waits for approval, and then continues.

Claude Code implements this with `plan` mode. In plan mode, the model researches and produces a structured plan, but cannot execute changes until the user approves. The `AskUserQuestion` tool is the mechanism for mid-loop interruption: the agent pauses, asks a specific question, waits for input, and resumes.

```
  ┌───────────────────────────────────────────────────────────┐
  │  Checkpoint pattern                                       │
  │                                                           │
  │  Agent researches task                                    │
  │         │                                                 │
  │         ▼                                                 │
  │  Agent produces plan: "I will modify these 3 files        │
  │  and run the migration script."                           │
  │         │                                                 │
  │         ▼                                                 │
  │  ┌─────────────────────┐                                  │
  │  │  Human reviews plan │                                  │
  │  └──────────┬──────────┘                                  │
  │             │                                             │
  │     Approve │ Reject/Modify                               │
  │             │                                             │
  │             ▼                                             │
  │  Agent executes (or adjusts and re-plans)                 │
  └───────────────────────────────────────────────────────────┘
```

This pattern is essential when running agents in automated pipelines such as CI/CD, where there is no interactive user present. The agent must either be pre-constrained to safe-only actions, or the pipeline must be designed with an explicit approval gate before any destructive operation proceeds.

A useful mental model: the agent is a contractor who will do exactly what they are asked. The checkpoint is the moment you review the blueprint before they break ground. Skipping the review is not more efficient; it is moving the risk from "we caught a mistake on paper" to "we have to fix it in concrete."

## Task decomposition: choosing the right approach

Not every task warrants an agent with tool access. The question to ask is: what does the task actually need in order to be completed correctly?

A single skill call works when all of the required context can be assembled by the host before calling the model. The task is self-contained, and the output is text that a human will review before acting on. Examples: drafting a PR description given a diff, explaining an error message, generating a unit test scaffold for a function whose source you already have.

A chained workflow works when the task involves multiple steps, but the steps are predictable and can be wired together in advance. Each step is a skill invocation, and the output of one step feeds into the prompt of the next. There is no loop; the sequence is fixed.

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Chained workflow                                           │
  │                                                             │
  │  Step 1: Summarize failing tests (text out)                 │
  │               │                                             │
  │               ▼                                             │
  │  Step 2: Given summary, propose fixes (text out)            │
  │               │                                             │
  │               ▼                                             │
  │  Step 3: Given proposals, write a JIRA ticket (text out)    │
  └─────────────────────────────────────────────────────────────┘
```

Each step is a skill invocation. The chain is deterministic. A human still reviews and acts on the final output. No step has write access to any system. Examples: a pipeline that reads test output, summarizes failures, and drafts follow-up tickets; a pipeline that reviews a PR description and suggests edits.

A full agent is needed when the task cannot be completed without the model itself gathering information or taking action mid-task. The host cannot predict in advance exactly what context will be needed, or the task inherently involves making changes to a system.

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Full agent                                                 │
  │                                                             │
  │  Task: "Fix the flaky test in the auth module"              │
  │               │                                             │
  │               ▼                                             │
  │  Agent reads test file, reads source file,                  │
  │  reads related utility, runs the test,                      │
  │  reads the failure output, patches the source,              │
  │  runs the test again, confirms it passes.                   │
  │               │                                             │
  │               ▼                                             │
  │  Agent reports: "Fixed. Changed line 47 in auth.ts."        │
  └─────────────────────────────────────────────────────────────┘
```

No amount of pre-assembly by the host could have predicted exactly which files to read or in what order. The model had to navigate the codebase reactively.

### Decision guide

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Task decomposition decision                                 │
  │                                                              │
  │  Can you provide all needed context in the prompt?           │
  │    Yes ──────────────────────────────── Single skill call    │
  │    No, but the steps are predictable ── Chained workflow     │
  │    No, and steps depend on findings ─── Full agent           │
  │                                                              │
  │  Does the task require making changes to a system?           │
  │    No ──────────────────────────────── Skill or chain        │
  │    Yes, with human review of output ─── Skill or chain       │
  │    Yes, autonomously ────────────────── Full agent           │
  │                                                              │
  │  Is the task scope well-defined and bounded?                 │
  │    Yes ──────────────────────────────── Skill or chain       │
  │    No, requires open-ended exploration── Full agent          │
  └──────────────────────────────────────────────────────────────┘
```

Use the least powerful approach that can correctly complete the task. Agents introduce loop complexity, token cost, and action risk that simpler approaches do not. A chained workflow of skill calls is easier to test, easier to debug, and carries lower risk than a full agent doing the same work inside a loop.

## The hierarchy

The three approaches sit at different positions on a capability and risk scale:

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Capability and risk hierarchy                               │
  │                                                              │
  │  Skill invocation                                            │
  │    Single model call, text in / text out                     │
  │    No tool access, no loop                                   │
  │    Risk: bad text                                            │
  │                           │                                  │
  │                           ▼                                  │
  │  Chained workflow                                            │
  │    Multiple skill calls wired in sequence                    │
  │    No loop, no autonomous tool access                        │
  │    Risk: bad text at each step                               │
  │                           │                                  │
  │                           ▼                                  │
  │  Agent with tool access                                      │
  │    Model in a loop, can call tools, can take actions         │
  │    Gathers context autonomously, can modify systems          │
  │    Risk: proportional to tool permissions and loop depth     │
  └──────────────────────────────────────────────────────────────┘
```

Knowing where a given workflow sits tells you what can go wrong and how badly, where to put checkpoints, and how to scope tool permissions to limit blast radius. The model itself does not change across these three patterns. What changes is whether it loops, which tools it can reach, and whether a human can intervene before consequential actions are taken.