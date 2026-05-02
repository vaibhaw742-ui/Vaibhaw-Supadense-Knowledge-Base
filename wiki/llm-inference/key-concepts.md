# ⚡ LLM Inference

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** working

**Resources:** 1

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

---

_Built: 2026-05-02_