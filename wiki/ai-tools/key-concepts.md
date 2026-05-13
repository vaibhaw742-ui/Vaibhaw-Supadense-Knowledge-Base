# AI Tools

> This file is managed by the supadense agent. Edit via chat only.

Useful AI tools, libraries, and platforms for building and using AI systems.

**Depth:** working

**Resources:** 17

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

<details>
<summary>Key Concept 9 — 10 Principles for Agent-Native CLIs</summary>

- **Agent-Native CLI** — A command-line interface designed specifically for interaction with AI agents, optimized to reduce agent token usage, minimize retries, and avoid failure modes common with traditional human-centric CLIs.
- **Tier 1 (Table Stakes) Principles for Agent-Native CLIs** — Foundational, mandatory requirements that prevent agents from breaking, including: non-interactive operation by default, structured parseable output, actionable enumerated errors, safe retriable mutations with explicit boundaries, and bounded response sizes across all layers (runtime output and MCP description surfaces).
- **Tier 2 (Compounding) Principles for Agent-Native CLIs** — Advanced requirements that increase CLI utility as agent usage scales, including cross-CLI vocabulary consistency, support for persistent agent identity, async workflow handling, artifact routing, and maintainer feedback loops for agent friction points.
- **Non-interactive by default (CLI design principle)** — CLI commands must run without interactive prompts when invoked by agents, using standardized flags (e.g., --force, --yes) to bypass confirmations, honest TTY detection to treat non-TTY environments as headless, and structured input via flags/files instead of interactive menus.
- **Structured, parseable output (CLI design principle)** — All data-returning commands must support a consistent flag (e.g., --json) to return machine-readable output to stdout, route errors to stderr, use a stable exit code taxonomy, and suppress ANSI formatting for non-terminal output.
- **Actionable enumerated errors (CLI design principle)** — CLI errors must include the valid set of values when input fails against an enum or schema, provide correct invocation syntax, and offer concrete guidance instead of vague messages or stack traces, enabling agents to self-correct in a single retry.
- **Safe retries and explicit mutation boundaries (CLI design principle)** — Mutation operations must be idempotent (using natural keys or idempotency tokens), include --dry-run flags for destructive actions, return resource identifiers on completion, and maintain durable job state for async operations to avoid duplicate work on retry.
- **Bounded responses (CLI design principle)** — Commands must default to narrow output via pagination, filtering, and limits, include truncation hints to guide agents in narrowing queries, and keep MCP tool description surfaces concise (e.g., Cloudflare's 3000-operation MCP fits in under 1000 tokens) to minimize agent token costs.
- **Cross-CLI vocabulary consistency (CLI design principle)** — CLIs must use community-standard verbs (e.g., get instead of info, list instead of ls) and flags (e.g., --force instead of --skip-confirmations, --json instead of --format=json), enforced mechanically at the schema or codegen layer to avoid agents relearning per tool.

> [Source](https://x.com/trevin/status/2051316002730991795)

</details>

<details>
<summary>Key Concept 10 — 99% of Hermes Agent Users Have Never Touched These 15 Features</summary>

- **Custom Slash Commands** — Extensible AI tool feature where pre-built or user-created skills are mapped to slash commands (e.g., `/architecture-diagram`, `/codex`, `/sage`) for quick access across sessions and platforms.
- **Ephemeral Side Queries** — AI tool feature (e.g., `/btw` command) for quick, context-aware questions that use session context but call no tools and are not persisted to long-term memory.
- **Agent Power Toggles** — Quick command toggles (e.g., `/yolo`, `/fast`, `/reasoning`) to adjust agent behavior mid-session, such as skipping dangerous command approvals, switching to fast inference modes, or setting reasoning effort levels.
- **Hermes Agent** — Provider-agnostic AI agent with 100+ pre-built skills, persistent memory, session branching, multi-platform support, and custom slash command extensibility. Supports 17+ messaging platforms and mid-session LLM provider swapping.

> [Source](https://x.com/shmidtqq/status/2051307460208578864)

</details>

<details>
<summary>Key Concept 11 — What I Use Hermes Agent For (And How I Use It)</summary>

- **Cost-optimized AI tool usage**: Minimizing AI tool costs by leveraging free LLM tiers (e.g., OpenRouter free models, NVIDIA NIM free offerings), low-cost monthly subscriptions (e.g., Nous Portal $10/mo, ChatGPT Plus $20/mo) instead of high per-request API costs (e.g., Anthropic API).
- **Local AI tool hosting**: Running LLM-powered tools on consumer-grade hardware (e.g., 8GB NVIDIA RTX 4070, 16GB M1 MacBook RAM) using quantized models and tools like llama.cpp or LMStudio for privacy and offline access.
- **Multi-provider tool configuration**: AI tools that support connecting to multiple LLM providers (OpenRouter, Nous Portal, local endpoints, ChatGPT Plus non-API access) per agent profile, enabling flexible model selection.

> [Source](https://x.com/vmiss33/status/2050984556790939731)

</details>

<details>
<summary>Key Concept 12 — Mercury: The AI Agent We All Wanted - Where Control, Permissions, and Autonomy Finally Got Real</summary>

- **Mercury (AI agent tool)** — Open-source, background-native AI agent orchestrator built by Cosmic Stack, designed to address common agent pain points: insecure permissions, context-window bloat, and opaque identity systems.
- **Auto-Concise mode** — Mercury feature that automatically tightens injected context when 70% of the daily token budget is exceeded, maintaining task performance while keeping API costs flat.
- **Four-file soul system** — Mercury's plain-text configuration system using `soul.md` (core reasoning), `persona.md` (response tone), `taste.md` (user preferences like dark UI themes), and `heartbeat.md` (operational rules), fully version-controllable via Git.
- **Zero-dependency agent daemon** — Mercury's runtime that requires no external dependencies (e.g., Docker, VPS) and installs as a system service across macOS, Linux, and Windows, supporting one-command `mercury up` deployment.

> [Source](https://x.com/Ctrl_Alt_Zaid/status/2046902326657749114)

</details>

<details>
<summary>Key Concept 13 — Hermes Kanban Turns Multi-Agent Work Into a Real Board</summary>

- **Hermes Kanban**: An AI agent coordination tool that provides a multi-board, SQLite-backed Kanban system for managing persistent multi-agent workflows. Features include named agent profiles, task dependencies, stateful retries, crash recovery, and human supervision interfaces.
- **Multi-board isolation**: Support for creating per-project, per-domain, or per-client isolated Kanban boards, each with their own database, workspaces, logs, and dispatcher scope. Prevents task cross-contamination between unrelated workflows (e.g., AI news pipeline vs Minecraft server ops).
- **Hermes gateway dispatcher**: An embedded dispatcher loop that runs inside the Hermes gateway by default, waking every 60 seconds to reclaim stale claims, detect crashed workers, promote eligible tasks, and spawn assigned agent profiles. Replaces the deprecated standalone Kanban daemon for most users.
- **Gated Kanban toolset**: A set of agent-accessible tools (`kanban_show`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, etc.) that are only injected into a worker's model schema when the `HERMES_KANBAN_TASK` environment variable is set. Avoids schema bloat for non-Kanban Hermes sessions.
- **Kanban human interfaces**: Three supervision interfaces for humans: (1) CLI commands (`hermes kanban list`, `unblock`, etc.), (2) in-chat slash commands (`/kanban create`, `list`, etc. with auto-subscription to task events), (3) web dashboard with drag-and-drop status changes, run history, filters, and live updates.

> [Source](https://x.com/NeoAIForecast/status/2051443615768228062)

</details>

<details>
<summary>Key Concept 14 — Building a Virtual Filesystem for Mintlify's AI Assistant</summary>

- **ChromaFs**: A virtual filesystem tool built by Mintlify to enable instant, low-cost filesystem access for AI assistants. It intercepts UNIX commands (grep, cat, ls, find) and translates them into queries against an existing Chroma vector database, eliminating the need for high-latency, expensive sandbox environments. Session creation drops from ~46 seconds to ~100ms with zero marginal compute cost.
- **just-bash**: A TypeScript reimplementation of bash by Vercel Labs, providing a pluggable IFileSystem interface that handles command parsing, piping, and flag logic. It is used as the foundation for ChromaFs, allowing custom filesystem implementations to focus solely on translating filesystem calls.

> [Source](https://x.com/densumesh/status/2039765361533637016)

</details>

<details>
<summary>Key Concept 15 — How to Build a Self-Improving AI Lead Gen Agent on Hermes</summary>

- **Modular agent tool stack** — A design principle for AI agent systems where individual tools (signal monitors, enrichment providers, LLMs, CRMs) are interchangeable without disrupting core workflows, as the architecture is built on reusable patterns rather than vendor-locked tooling. Validated by the resource's example of swapping Trigify for alternative signal tools or Instantly for alternative outreach tools without breaking the agent's lead gen workflow.
- **Lead enrichment layer** — A component of AI SDR agent workflows that uses parallel third-party data providers (e.g., Lead Magic, Surfe) to gather additional company and contact context for qualified leads, with multiple providers used to leverage different scale-specific data strengths.
- **Signal monitoring layer** — A tool layer that ingests external intent signals (social media trends, funding announcements, role/company changes across X, LinkedIn, YouTube, Substack, Hacker News) and internal operational signals (sales call transcripts via Fireflies, team call notes via Granola, Slack conversations, Stripe transaction data) to feed agent content engines and prospecting workflows.

> [Source](https://x.com/MichLieben/status/2051707320699396454)

</details>

<details>
<summary>Key Concept 16 — AI Agents That Builds Themselves</summary>

- **Iris** — CrewAI's internal AI employee, a Slack-native agent that lives in threads, writes real code, files pull requests, reviews teammates' work, and has persistent cross-conversation memory. Built entirely on CrewAI (Flows for deterministic routing, Agents for reasoning, Crews for multi-agent collaboration, Memory for learning). It can fully modify itself, onboard itself to new tools, and change its own code. In production for months at CrewAI, roughly one in four PRs across all CrewAI repositories now originates from Iris, and the Iris repo is entirely AI-built as a testbed for self-improving agent concepts before they are incorporated back into CrewAI.

> [Source](https://x.com/joaomdmoura/status/2049562041007194275)

</details>

<details>
<summary>Key Concept 17 — Agent Platform That Builds Itself</summary>

- **Agno OS**: An open-source agent platform that can be run locally via Docker or deployed to production (e.g., Railway) with one command. Comprises 5 core components: Runtime, Storage, Connectors, Interfaces, and Infrastructure.

- **Claude Code**: A coding agent used to manage the entire agent development lifecycle (create, improve, extend, hill climb, review) through natural language prompts. It can:
  - Scaffold agents by asking requirements, researching toolkits via MCP, generating code
  - Derive and run probes against live agents, then fix failures
  - Add capabilities with human guidance
  - Run eval suites and perform hill climbing
  - Detect and fix drift between docs, code, and config

> [Source](https://x.com/ashpreetbedi/status/2054222428025614829?s=20)

</details>

---

_Built: 2026-05-13_