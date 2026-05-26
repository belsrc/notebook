═══════════════════════════════════════════════════════════
**MODULE 0: Mathematical and Computational Prerequisites**
═══════════════════════════════════════════════════════════

- [ ] **0.1  Linear Algebra for ML.** Vectors, matrices, dot products, matrix multiplication semantics ($AB$ vs $A \odot B$), transposes, rank, eigenvalues, singular value decomposition. Geometric intuition: vectors as points and directions, matrices as linear transformations.
- [ ] **0.2  Probability and Statistics.** Random variables, discrete vs continuous distributions, joint/marginal/conditional probability, Bayes' theorem $P(A \mid B) = \frac{P(B \mid A) P(A)}{P(B)}$, expectation, variance, the law of large numbers, central limit theorem.
- [ ] **0.3  Calculus and Optimization.** Derivatives, partial derivatives, gradients ($\nabla f$), the chain rule, convexity, gradient descent: $\theta_{t+1} = \theta_t - \eta \nabla_\theta \mathcal{L}(\theta_t)$, stochastic gradient descent, momentum, Adam.
- [ ] **0.4  Information Theory.** Entropy $H(X) = -\sum p(x) \log p(x)$, cross-entropy, KL divergence $D_{KL}(P \| Q)$, mutual information. Why cross-entropy is the natural loss for classification and language modeling.
- [ ] **0.5  Computational Foundations.** Computational graphs, automatic differentiation (forward and reverse mode), the GPU memory hierarchy, tensor layout, FLOPs vs memory bandwidth as the bottleneck.

═══════════════════════════════════════════════════════════
**MODULE 1: Foundations of Machine Learning**
═══════════════════════════════════════════════════════════

- [ ] **1.1  What Is Learning From Data.** Function approximation framing: $f: \mathcal{X} \to \mathcal{Y}$ learned from samples. Supervised, unsupervised, self-supervised, reinforcement learning — define each precisely.
- [ ] **1.2  Loss Functions and Empirical Risk.** Empirical risk minimization $\hat{R}(\theta) = \frac{1}{N} \sum_{i=1}^{N} \ell(f_\theta(x_i), y_i)$. MSE for regression, cross-entropy for classification, why squared error implies Gaussian noise assumptions.
- [ ] **1.3  Generalization.** Bias-variance decomposition, overfitting and underfitting, train/validation/test splits, k-fold cross-validation, the i.i.d. assumption and when it breaks.
- [ ] **1.4  Regularization.** L1/L2 regularization, dropout, early stopping, data augmentation, label smoothing — each as inductive bias.
- [ ] **1.5  Classical Models as Stepping Stones.** Linear regression, logistic regression, decision trees, k-nearest neighbors. Why these matter for understanding what neural nets generalize.

═══════════════════════════════════════════════════════════
**MODULE 2: Neural Networks**
═══════════════════════════════════════════════════════════

- [ ] **2.1  The Perceptron and Multi-Layer Perceptron.** The neuron as $y = \sigma(Wx + b)$, why depth matters (universal approximation theorem and its caveats).
- [ ] **2.2  Backpropagation From Scratch.** Derive the algorithm. Implement forward and backward passes for a 2-layer MLP by hand in C or Rust; verify against autograd.
- [ ] **2.3  Activation Functions.** Sigmoid, tanh, ReLU, GELU, SwiGLU. Vanishing gradients, dying ReLU, the modern preference for GELU/SwiGLU in transformers.
- [ ] **2.4  Architectures Worth Knowing (Brief Tour).** CNNs (convolution as weight-sharing), RNNs (state as sequence summary), residual connections. These exist as context for what transformers replaced and what they kept.
- [ ] **2.5  Embeddings.** Token, word, and sentence embeddings. Distributed representations. Why $\text{vec}(\text{king}) - \text{vec}(\text{man}) + \text{vec}(\text{woman}) \approx \text{vec}(\text{queen})$ and what that actually means geometrically.

═══════════════════════════════════════════════════════════
**MODULE 3: Sequence Models and the Transformer**
═══════════════════════════════════════════════════════════

- [ ] **3.1  The Sequence Modeling Problem.** Why $P(x_1, \ldots, x_T) = \prod_t P(x_t \mid x_{<t})$ is the universal factorization. Autoregressive vs masked vs encoder-decoder framings.
- [ ] **3.2  RNN/LSTM (Historical Context).** Sequential bottleneck, gradient flow problems, why attention killed them for most tasks.
- [ ] **3.3  Attention Mechanism.** $\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$. Build the intuition: query asks, keys answer, values are the payload.
- [ ] **3.4  Self-Attention and Multi-Head Attention.** Why split into $h$ heads, what each head can learn, computational cost $O(T^2 d)$ in sequence length.
- [ ] **3.5  The Full Transformer Block.** Pre-norm vs post-norm, residual connections, feed-forward network (MLP) per token, why this exact recipe works.
- [ ] **3.6  Positional Encodings.** Sinusoidal, learned, RoPE (rotary), ALiBi. Why position is a hard problem when attention is set-equivariant.

═══════════════════════════════════════════════════════════
**MODULE 4: Large Language Models**
═══════════════════════════════════════════════════════════

- [ ] **4.1  Tokenization.** Byte-Pair Encoding (BPE), WordPiece, SentencePiece, Unicode pitfalls, why tokenization choices leak into model behavior (math, code, non-Latin scripts).
- [ ] **4.2  Pretraining Objectives.** Next-token prediction at scale, masked LM (BERT-style), why decoder-only autoregressive won.
- [ ] **4.3  The Context Window.** Mechanically: KV cache structure, what fits, the $O(T^2)$ attention cost and what techniques (FlashAttention, sliding window, ring attention) attempt.
- [ ] **4.4  Sampling and Decoding.** Greedy, temperature scaling $p_i \propto \exp(z_i / T)$, top-$k$, top-$p$ (nucleus), beam search, speculative decoding.
- [ ] **4.5  Scaling Laws.** Chinchilla-optimal compute allocation $N \approx 20D$ (parameters vs tokens), loss vs compute power laws, the implications for model design.
- [ ] **4.6  Emergent Behaviors.** What "emergence" means (and the debate over whether it's an artifact of metric choice), in-context learning as the surprise.

═══════════════════════════════════════════════════════════
**MODULE 5: Post-Training and Alignment**
═══════════════════════════════════════════════════════════

- [ ] **5.1  Supervised Fine-Tuning (SFT).** Instruction tuning, dataset curation, catastrophic forgetting, the role of high-quality demonstrations.
- [ ] **5.2  Reinforcement Learning From Human Feedback (RLHF).** Reward model training from preference data, PPO loop, KL penalty against reference policy.
- [ ] **5.3  Constitutional AI and RLAIF.** Self-critique loops, principle-based feedback, scalable oversight.
- [ ] **5.4  Direct Preference Optimization (DPO).** The reformulation that skips the reward model. Why $\mathcal{L}_{DPO}$ is mathematically equivalent under assumptions.
- [ ] **5.5  Refusal Training and Safety Fine-Tuning.** How refusal behavior is shaped, why it generalizes (and where it doesn't), the harmlessness-helpfulness tension.

═══════════════════════════════════════════════════════════
**MODULE 6: Prompting and In-Context Learning**
═══════════════════════════════════════════════════════════

- [ ] **6.1  Zero-shot, Few-shot, and the Demonstration Format.** How example structure shapes outputs, the order sensitivity problem.
- [ ] **6.2  Chain-of-Thought (CoT).** Eliciting step-by-step reasoning, when it helps and when it doesn't, zero-shot CoT ("let's think step by step").
- [ ] **6.3  System Prompts and Role Conditioning.** The distinction between system, user, and assistant turns; how providers compose these into the actual context.
- [ ] **6.4  Structured Output.** JSON mode, schema-constrained decoding, grammar-based sampling (GBNF), when to use each.
- [ ] **6.5  In-Context Learning Mechanics.** What's actually happening — induction heads, implicit gradient descent interpretations, the limits of these analogies.
- [ ] **6.6  Prompt Engineering as Engineering.** Versioning, evals on prompts, the discipline of prompt iteration. Why "prompt engineering" is real work, not magic incantation.

═══════════════════════════════════════════════════════════
**MODULE 7: Retrieval and External Knowledge**
═══════════════════════════════════════════════════════════

- [ ] **7.1  Embedding Models.** Bi-encoders vs cross-encoders, sentence-transformers, contrastive training objectives, dimension trade-offs.
- [ ] **7.2  Vector Databases.** HNSW, IVF, product quantization. Recall vs latency curves. When to use Pinecone/Weaviate/pgvector vs a flat index.
- [ ] **7.3  Retrieval-Augmented Generation (RAG).** The full pipeline: chunk → embed → index → retrieve → rerank → generate.
- [ ] **7.4  Chunking Strategies.** Fixed-size, semantic, hierarchical, sliding-window with overlap. The chunk-boundary failure mode.
- [ ] **7.5  Reranking and Hybrid Search.** BM25 + vector hybrid, cross-encoder rerankers, reciprocal rank fusion.
- [ ] **7.6  Failure Modes.** Hallucination despite retrieval, retrieval misses, context dilution, "lost in the middle" attention pathology.

═══════════════════════════════════════════════════════════
**MODULE 8: Tool Use and Function Calling**
═══════════════════════════════════════════════════════════

- [ ] **8.1  Why Tools Turn an LLM Into Something New.** The fundamental shift: from text-completer to system that *acts*.
- [ ] **8.2  Function Calling Mechanics.** JSON Schema for tool definitions, structured output for argument generation, how providers (OpenAI/Anthropic/Gemini) implement this.
- [ ] **8.3  Tool Selection.** Multi-tool routing, tool descriptions as prompts, the cost of putting many tools in context.
- [ ] **8.4  Tool Result Handling.** Streaming, error recovery, retries with reflection, truncation strategies for large results.
- [ ] **8.5  Model Context Protocol (MCP).** Server architecture, transport layers (stdio, SSE, streamable HTTP), resource vs tool vs prompt primitives. Why a standard matters.
- [ ] **8.6  Code Execution as the Universal Tool.** Sandboxed Python/JS execution, the "code is the tool" insight, security boundaries.

═══════════════════════════════════════════════════════════
**MODULE 9: Agent Architectures**
═══════════════════════════════════════════════════════════

- [x] **9.1  Defining "Agent".** Perception-action loop, the spectrum from workflow → agent (Anthropic's framing). Why the word is contested.
- [x] **9.2  The ReAct Pattern.** Interleaved reasoning and action: $\text{Thought} \to \text{Action} \to \text{Observation} \to \text{Thought} \to \ldots$
- [x] **9.3  Plan-and-Execute.** Two-stage: planner produces task graph, executor walks it. Trade-offs vs ReAct.
- [x] **9.4  Reflection and Self-Critique.** Self-Refine, Reflexion, critic models. The pattern: act → critique → revise.
- [x] **9.5  Tree of Thoughts and Search-Based Reasoning.** Exploration over thought trees with backtracking; the connection to classical search.
- [ ] **9.6  Multi-Agent Systems.** Orchestrator-worker, debate, role-specialized agents. When this helps and when it adds latency without value.
- [ ] **9.7  The Anthropic Taxonomy.** Augmented LLM → prompt chaining → routing → parallelization → orchestrator-workers → evaluator-optimizer → autonomous agent. Know each and when to choose it.

═══════════════════════════════════════════════════════════
**MODULE 10: Agent Engineering Patterns**
═══════════════════════════════════════════════════════════

- [ ] **10.1  Context Engineering as a Discipline.** Distinct from prompt engineering: managing what is in context, when, and why. Context as the single most important agent design surface.
- [ ] **10.2  Subagent Spawning and Context Isolation.** The Stripe Minions pattern, Claude Code's subagents, why isolated contexts prevent premature design and reduce token cost.
- [ ] **10.3  Artifact Chaining.** Phase-separated work where each phase outputs an artifact consumed by the next. Decouples context windows; enables auditing.
- [ ] **10.4  Developer-in-the-Loop Workflows.** Local CLI tool → skill wrapper → agent layering. Structured JSON output from CLI tools to reduce token cost via filtering, grouping, deduplication.
- [ ] **10.5  Skill Systems.** Claude Code skills, the SKILL.md pattern, progressive disclosure of context.
- [ ] **10.6  Workflow Orchestration.** Spec-driven development (OpenSpec, QRSPI), explicit phase gates, human approval checkpoints.
- [ ] **10.7  Failure Recovery.** Retries with state, idempotency, transactional tool calls, compensating actions.

═══════════════════════════════════════════════════════════
**MODULE 11: Agent Memory Systems**
═══════════════════════════════════════════════════════════

- [ ] **11.1  The Memory Taxonomy.** Working (context window), episodic (past interactions), semantic (consolidated facts), procedural (skills/tools).
- [ ] **11.2  Memory Architectures.** MemGPT-style virtual context, summarization buffers, vector-stored episodic memory, knowledge graphs for semantic memory.
- [ ] **11.3  Memory Operations.** Write (what to commit), read (what to retrieve), update (consolidation), forget (decay/eviction).
- [ ] **11.4  Reflection-Driven Consolidation.** Generative Agents' pattern: periodic reflection over recent episodes produces higher-level semantic memories.
- [ ] **11.5  Cross-Session Continuity.** Identifying the same user, the same project, the same task across conversations; the privacy and accuracy trade-offs.

═══════════════════════════════════════════════════════════
**MODULE 12: Evaluation and Observability**
═══════════════════════════════════════════════════════════

- [ ] **12.1  LLM Benchmarks and Their Limits.** MMLU, HellaSwag, HumanEval, GSM8K, BIG-bench. Why benchmark saturation doesn't mean solved.
- [ ] **12.2  Agent Benchmarks.** SWE-bench (verified, multimodal), GAIA, AgentBench, WebArena, $\tau$-bench. What each actually measures.
- [ ] **12.3  LLM-as-Judge.** Pairwise vs pointwise grading, judge calibration, position bias, the meta-evaluation problem.
- [ ] **12.4  Tracing and Observability.** Spans for every model call and tool invocation, latency/cost breakdowns, tools like LangSmith, Arize, Helicone, OpenTelemetry-based traces.
- [ ] **12.5  Production Metrics.** Task success rate, escalation rate, time-to-resolution, cost per task. The shift from "model accuracy" to "system effectiveness".
- [ ] **12.6  Eval-Driven Development.** Building an eval set before optimizing the agent, the test-driven development analog.

═══════════════════════════════════════════════════════════
**MODULE 13: Safety, Alignment, and Failure Modes**
═══════════════════════════════════════════════════════════

- [ ] **13.1  Hallucination.** What it actually is (next-token sampling from a distribution that doesn't track ground truth), why retrieval helps but doesn't solve it.
- [ ] **13.2  Prompt Injection.** Direct and indirect, the "lethal trifecta" (private data + untrusted content + external communication), defenses and their limits.
- [ ] **13.3  Jailbreaking.** Adversarial suffixes, role-play attacks, many-shot jailbreaking, why robust refusal is open research.
- [ ] **13.4  Goal Misgeneralization and Specification Gaming.** Reward hacking, Goodhart's law in RL, classic examples (boat-racing).
- [ ] **13.5  Agent-Specific Risks.** Excessive autonomy, tool misuse, exfiltration via tool calls, the principle of least privilege.
- [ ] **13.6  Sandboxing and Permissions.** Process isolation, network egress control, filesystem boundaries, capability-based security models.
- [ ] **13.7  Human Oversight Patterns.** Approval gates, dry-run modes, undo/rollback, audit logs.

═══════════════════════════════════════════════════════════
**MODULE 14: Production Systems**
═══════════════════════════════════════════════════════════

- [ ] **14.1  Deployment Topologies.** Direct API, gateway (LiteLLM, OpenRouter), self-hosted (vLLM, TGI), the latency/cost/control trade-off.
- [ ] **14.2  Caching Strategies.** Prompt caching (Anthropic's beta and equivalents), semantic caching, embedding caches, when caching is dangerous.
- [ ] **14.3  Streaming.** SSE vs WebSocket, partial JSON parsing, UX implications, how to stream tool calls.
- [ ] **14.4  Cost Optimization.** Routing simple queries to cheaper models, prompt compression, context pruning, batch inference for non-interactive work.
- [ ] **14.5  Concurrency, Rate Limits, and Backpressure.** Token-bucket limits, request queuing, graceful degradation under quota exhaustion.
- [ ] **14.6  Versioning and Rollout.** Model version pinning, shadow deployment, A/B testing prompts/agents, the regression problem when model providers update.
- [ ] **14.7  Cost and Latency Profiling.** Per-call accounting, the long tail of slow requests, p99 vs mean.

═══════════════════════════════════════════════════════════
**MODULE 15: Synthesis and Frontiers**
═══════════════════════════════════════════════════════════

- [ ] **15.1  Multimodal Agents.** Vision-language models, audio, the unified token stream framing.
- [ ] **15.2  Computer-Use and Browser Agents.** Screen perception via vision, action via clicks/keystrokes, the accessibility-tree vs pixel debate.
- [ ] **15.3  Coding Agents.** Claude Code, Cursor, Aider, Devin, SWE-agent. Architectural differences. Why coding is the canonical agent domain.
- [ ] **15.4  Long-Horizon Agents.** Persistent state, multi-day tasks, the open problem of error compounding over long trajectories.
- [ ] **15.5  Open Research Problems.** Reliable reasoning, robust tool use, scalable oversight, interpretability of agent decisions, alignment of increasingly capable systems.
- [ ] **15.6  Capstone Synthesis.** Choose a non-trivial agent system from your work context; explain every architectural decision, trace failure modes, propose improvements. Mastery is demonstrated by being able to design from first principles, not by reciting patterns.
