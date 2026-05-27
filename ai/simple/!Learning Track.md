**1. Foundation**


- [~] 1.1 Token Economics - What tokens are, how they map to cost, why output tokens cost more than input, and how to think about prompt efficiency. This is the economic model everything else is built on.
- [~] 1.2 Context Window Mechanics - What the context window is, how it fills, what truncation and attention dilution look like, and why these matter for skill design. Read the degradation modes section carefully.
- [~] 1.3 Inference - What the agent can already infer from context and what it cannot. This changes how you write every skill, every CLAUDE.md, and every briefing document.
- [~] 1.4 Agent Memory - The four canonical memory types (in-weights, in-context, external, in-cache), how they compose, and the capacity/latency/precision trade-offs. Foundation for understanding how agents persist and retrieve information.
- [~] 1.5 Hallucination in LLMs - What hallucination is, why it happens mechanically, how to recognize it by task type, and how to reduce it. Finish this document before writing or reviewing any AI output in production.
- [~] 1.6 Domain Knowledge - The public / applied / tribal knowledge stack. The single most important concept for understanding why agent output drifts from team expectations.


**2. Tooling**


- [~] 2.1 Introduction to Agent Skills - What a skill is in this codebase, how skills are structured, how they load, and how they are distributed. This is the primary authoring reference.
- [~] 2.2 Claude Code in Action - The full Claude Code reference: the agent loop, built-in tools, MCP servers, hooks, and GitHub Actions integration.
- [~] 2.3 Auto-Memory - Claude Code's persistent memory system: MEMORY.md vs CLAUDE.md, the 200-line constraint, what gets saved automatically, and how to manage it. Practical implementation of agent memory concepts.


**3. Agent architecture**


- [~] 3.1 Degrees of Freedom - Why agent task complexity multiplies rather than adds, and how to collapse degrees of freedom using constraints, examples, and templates.
- [~] 3.2 Determinism in Agents - Why agents cannot be fully deterministic, the four layers of non-determinism, and practical convergence strategies. Companion to the above.
- [~] 3.3 Spec Driven Development - Why vague instructions produce silent assumptions and how writing a formal specification before you build collapses that failure mode. Covers spec anatomy, behavioral rules, success criteria
- [~] 3.4 Sub-Agents - Built-in and custom subagents, invocation modes, parallelism, worktree isolation, and SDK integration. Read this when you are ready to design Tier 3 orchestrated workflows.
- [~] 3.5 Agents and Tools - The conceptual distinction between a skill invocation and an agent, what tool access means for failure modes and review requirements, and when a human checkpoint is mandatory.