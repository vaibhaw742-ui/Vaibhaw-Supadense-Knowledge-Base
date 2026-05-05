# AI Tools

> This file is managed by the supadense agent. Edit via chat only.

Useful AI tools, libraries, and platforms for building and using AI systems.

**Depth:** working

**Resources:** 8

## Key Concepts

<details>
<summary>Key Concept 1 — Systems Engineering: Building Agentic Software That Works</summary>

- **Dash Data Agent** — An open-source, self-learning data agent that accepts plain English questions, writes SQL, runs it, and explains the results. Demonstrates systems engineering across five layers: agent engineering (multi-agent team), data engineering (six context layers + learning loop), security engineering (RBAC with JWT, database-enforced permissions), interface engineering (REST API, Slack bot, web UI, CLI), and infrastructure engineering (Docker container, SSE streaming).

- **Agent Learning Loop** — Pattern where an agent runs a query, encounters an error, diagnoses the fix, saves it to a knowledge base, and recalls it on future similar queries. Query quality improves because the data layer improves, not because the model improves.

> [Source](https://x.com/ashpreetbedi/status/2041568919085854847)

</details>

<details>
<summary>Key Concept 2 — Dash v2: The Multi-Agent Data System Every Company Needs</summary>

- **Dash v2** — Free, open-source self-learning multi-agent data system designed for companies with 30+ employees. Integrates with Slack and AgentOS UI, handles ~80% of routine data questions, sends daily reports, and detects metric anomalies. Runs in private cloud, uses a dual-tier knowledge/learning system, and automatically builds analytics infrastructure for repeated questions.
- **AgentOS UI** — Agent platform for interacting with Dash, supporting agent chat, session viewing, trace inspection, metrics tracking, and schedule management. Powered by Dash's `ai` schema that stores operational data.
- **Dash Knowledge Base** — PgVector-backed storage of curated context: table metadata (schema, column meanings, gotchas), validated SQL queries (battle-tested with correct JOINs and edge case handling), and business rules (metric definitions, calculation logic, common gotchas).
- **Dash Learning Base** — Automatically captured improvements: SQL type fixes, user result corrections, discovered data quirks. Stored separately from the knowledge base, used by the Analyst agent to avoid repeating past mistakes.

> [Source](https://x.com/ashpreetbedi/status/2041901460523270409)

</details>

<details>
<summary>Key Concept 3 — How Hermes Agent Solves Skill Drift and Context Rot as a Self-Improving Agent</summary>

- **Hermes Curator**: A background maintenance tool for self-improving agents, designed to manage agent-created skills by tracking usage (via sidecar telemetry files in the skills folder), automating state transitions (active → stale → archived), and running periodic auxiliary model reviews (using cheaper models like Gemini Flash) to merge, patch, or archive redundant or drifted skills. It includes a CLI for manual triggers, status checks, pinning, and restoration of archived skills.
- **Skill Pinning**: A tool feature that protects critical skills from automated cleanup: pinned skills are invisible to the Curator's usage clock and review model, and cannot be modified by the agent's own editing tools, preventing accidental changes to dependencies.

> [Source](https://x.com/mem0ai/status/2050351798142288050)

</details>

<details>
<summary>Key Concept 4 — The Database Is No Longer Storage - It Is Becoming the Runtime for AI</summary>

- **Object-Storage-Native Persistence** — A database architecture pattern where object storage (like S3) serves as the primary persistence layer rather than traditional block storage or local disks. Enables better elasticity and separation of storage/compute for agent workloads.

- **Elastic Compute for Databases** — The ability to dynamically scale database compute resources up and down to handle bursty, unpredictable agent-generated workloads, as opposed to provisioning for steady-state human traffic.

- **Unified Transactional and Analytical Workloads (HTAP)** — Database systems that can handle both OLTP (transactional) and OLAP (analytical) patterns simultaneously, which is essential for agent systems that mix state updates with context queries in the same interaction loop.

- **Usage-Based Pricing for Database Infrastructure** — Economic models for database services that align with agent workloads, where machines are the main users generating variable, spikey traffic patterns rather than predictable per-seat human pricing.

> [Source](https://x.com/siddontang/status/2050409724949270783)

</details>

<details>
<summary>Key Concept 5 — Coding Agents are Effective Long-Context Processors</summary>

- **Coding Agent Long-Context Processing**: Framing long-context text processing tasks as file system navigation and manipulation delegated to off-the-shelf coding agents (e.g., Codex), replacing latent attention or fixed RAG retrieval.
- **Text Processing as File System Operation**: Methodology of structuring text corpora as directory hierarchies (individual txt files, JSONL, or single txt) and delegating processing to coding agents with full autonomy over exploration strategies.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

<details>
<summary>Key Concept 6 — Swarm Management of Agent Harnesses</summary>

- **OpenClaw** — AI agent runtime with production-grade swarm management: durable subagent session keys, run IDs, lifecycle records, parent-child lineage, cleanup policy, push-based `task_completion` events, queue policy enforcement, steering/cascade cancellation, role-based access control, subagent registry, and background sweeper for recovery.
- **Hermes** — AI agent framework with `delegate_task` primitive: spawns child AIAgent instances, runs them in parallel, streams progress, applies timeouts, interrupts them, and returns structured summaries to the parent. Children live within the parent tool call, lacking decoupled lifecycle management.
- **Devin** — Cognition's AI agent system capable of managing other Devin instances, cited as an early example of multi-agent delegation.
- **Anthropic Managed Agents** — Anthropic's implementation of managed agent runtimes, referenced as an example of external swarm management systems.
- **Cursor Long-Running Agents** — Cursor's design for agents with extended execution windows, cited as an example of systems requiring swarm management for long-running tasks.

> [Source](https://x.com/aparnadhinak/status/2051014879449157952)

</details>

<details>
<summary>Key Concept 7 — We don't need more agents. We need a better system. So we built one.</summary>

- **Augment Cosmos**: Public preview operating system for agentic software development, supporting agents running in local or cloud environments, with team-shared context, self-improving agent loops, multi-model support, and integrations to existing development tools.
- **Prism**: Augment's model routing system that delivers ~20–30% token savings without sacrificing output quality, by dynamically selecting optimal models for tasks to balance cost and performance.
- **Model-agnostic agentic system**: Agentic tooling that supports multiple LLM providers, avoiding vendor lock-in, and enabling cost optimization by matching tasks to the most appropriate model for the job.

> [Source](https://x.com/augmentcode/status/2051350118360891584)

</details>

<details>
<summary>Key Concept 8 — Recursive Agent Optimization Actually Works</summary>

- **HALO (Hierarchical Agent Loop Optimization)** — Open-source toolset and methodology for recursively optimizing agent harnesses, including the HALO-Engine (specialized RLM), CLI, and pre-built demos (e.g., AppWorld integration).
- **HALO-Engine** — Specialized Recursive Language Model (RLM) powering HALO, available as an open-source engine or hosted service via Catalyst, capable of analyzing large agent trace datasets to diagnose failures.
- **Recursive Agent Optimization** — Tool-driven paradigm for agent harness improvement, automating trace analysis, failure diagnosis, and fix proposal to reduce reliance on manual debugging or custom benchmarks.

> [Source](https://x.com/AmarSVS/status/2051470760947159197)

</details>

---

_Built: 2026-05-05_