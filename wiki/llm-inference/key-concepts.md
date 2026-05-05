# ⚡ LLM Inference

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** working

**Resources:** 3

## Key Concepts

<details>
<summary>Key Concept 1 — Hidden Technical Debt of AI Systems: Agent Runtime</summary>

Agent runtime is the serving infrastructure layer for agentic AI — analogous to how model serving infrastructure surrounds the ML code in traditional MLOps. The agentic model call is the small box; the runtime (compute substrate, filesystem, tools, network boundary, state model, lifecycle controller) is the large box where technical debt accumulates.

Key serving infrastructure considerations for agent workloads:
- **Sandboxing is mandatory** — agents execute arbitrary code and read untrusted tool outputs, so kernel-level isolation (VM per session, not shared-kernel containers) is required
- **Cold start matters more than throughput** — agent productivity is bounded by runtime startup time, not model tokens-per-second; sub-second snapshot restores are critical
- **Multi-tenancy at scale** — RL training spins up thousands of concurrent rollouts, each needing isolated filesystem, process tree, and network namespace
- **Firecracker microVMs** are the de facto industry primitive for serving untrusted agent code at high density (~125ms boot, ~5MB VMM overhead, dedicated kernel per VM)

> [Source](https://leehanchung.github.io/blogs/2026/04/24/hidden-technical-debt-agent-runtime/)

</details>

<details>
<summary>Key Concept 2 — Coding Agents are Effective Long-Context Processors</summary>

- **Long-Context Access vs Processing Gap**: LLMs can scale to access massive contexts (up to millions of tokens) via expanded context windows, but fail to effectively process long context, with performance degrading as context length increases (context rot).
- **Externalized Long-Context Processing**: Moving long-context processing from latent, uninterpretable attention mechanisms to explicit, executable interactions via coding agents, improving both performance and interpretability.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

<details>
<summary>Key Concept 3 — We don't need more agents. We need a better system. So we built one.</summary>

- **Model routing**: Process of dynamically selecting the most appropriate LLM for a given task based on cost, quality, and task requirements, to minimize token usage and operational cost without sacrificing output quality.
- **Multi-model inference**: Using multiple LLM providers/models in a single workflow, optimizing for cost and performance by routing tasks to the best-fit model rather than defaulting to a single frontier model for all tasks.

> [Source](https://x.com/augmentcode/status/2051350118360891584)

</details>

---

_Built: 2026-05-05_