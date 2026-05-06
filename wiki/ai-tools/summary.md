# AI Tools

> This file is managed by the supadense agent. Edit via chat only.

Useful AI tools, libraries, and platforms for building and using AI systems.

**Depth:** working

**Resources:** 9

## Key Concepts

<details>
<summary>Key Concept 1 — Systems Engineering: Building Agentic Software That Works</summary>

- **Dash** — Open-source self-learning data agent by agno-agi. Accepts natural language questions, writes and executes SQL, explains results. Built on systems engineering principles with multi-agent architecture (Leader routes to Analyst for read-only queries and Engineer for creating computed assets).
  - GitHub: https://github.com/agno-agi/dash
  - Implements six layers of grounded context and a learning loop where agents save error patterns and recall them on future queries

> [Source](https://x.com/ashpreetbedi/status/2041568919085854847)

</details>

<details>
<summary>Key Concept 2 — Dash v2: The Multi-Agent Data System Every Company Needs</summary>

- **Dash v2** — Free, open-source self-learning multi-agent data system for companies with 30+ employees. Deploys privately in your cloud, integrates with Slack and AgentOS UI, handles ~80% of routine data questions, sends daily reports, and detects metric anomalies before users ask.
- **Core capabilities** — Self-learning loop (improves from corrections and errors), automatic analytics infrastructure building (creates views/summary tables for repeated queries), dual-tier knowledge system (curated business context + captured learnings), strict schema-level permission controls.
- **Deployment** — Docker-based deployment, requires OpenAI API key. Connects to Slack via public URL/ngrok, or to Web UI at os.agno.com. Replace sample data by loading your tables into the `public` schema and adding knowledge files (table metadata, validated queries, business rules).

> [Source](https://x.com/ashpreetbedi/status/2041901460523270409)

</details>

<details>
<summary>Key Concept 3 — How Hermes Agent Solves Skill Drift and Context Rot as a Self-Improving Agent</summary>

- **Hermes Curator**: A background maintenance tool for self-improving agents, part of Hermes Agent v0.12.0 ("The Curator Release"), addressing skill drift and context rot by managing agent-created skills via usage tracking, state transitions, and auxiliary model reviews.
- **Skill Pinning**: A guardrail feature for skill management tools that marks critical skills as immutable, excluding them from automated cleanup and modifications to protect dependencies.

> [Source](https://x.com/mem0ai/status/2050351798142288050)

</details>

<details>
<summary>Key Concept 4 — Coding Agents are Effective Long-Context Processors</summary>

- **Coding Agent Long-Context Processing**: Framing long-context text processing tasks as file system navigation and manipulation delegated to off-the-shelf coding agents (e.g., Codex), replacing latent attention or fixed RAG retrieval.
- **Text Processing as File System Operation**: Methodology of structuring text corpora as directory hierarchies and delegating processing to coding agents with full autonomy over exploration strategies.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

<details>
<summary>Key Concept 5 — Recursive Agent Optimization Actually Works</summary>

- **HALO (Hierarchical Agent Loop Optimization)** — Open-source, recursively self-improving agent harness methodology that improved AppWorld scenario completion rate from 73.7 to 89.5 in 5 optimization cycles.
- **HALO-Engine** — Specialized RLM at the core of HALO, outperforming off-the-shelf RLMs and coding agents like Claude Code for trace diagnosis tasks.
- **Recursive Agent Optimization** — Validated paradigm where AI systems improve other AI systems, with diagnostic value comparable to an athlete reviewing game footage post-match.

> [Source](https://x.com/AmarSVS/status/2051470760947159197)

</details>

<details>
<summary>Key Concept 6 — What I Use Hermes Agent For (And How I Use It)</summary>

- **Hermes Agent**: A multi-agent AI framework with TUI and Telegram interfaces, supporting configurable agent profiles with different LLM providers/models (OpenRouter, Nous Portal, local endpoints, ChatGPT Plus). Users run specialized agent crews (research, task execution, lifestyle) rather than a single general-purpose agent.
- **Cost-conscious tool strategy**: Avoiding high per-request API costs (e.g., Anthropic) by combining free tiers (OpenRouter 1,000 req/day with $10 credit, NVIDIA NIM free models), low-cost subscriptions (Nous Portal $10/mo, ChatGPT Plus $20/mo non-API access), and local inference on consumer hardware.
- **Agent use case methodology**: Identify value by auditing daily tasks for a week, finding time-consuming/low-value work and life friction points (forgetfulness, health reminders), then building agents around those specific needs rather than starting with technology.

> [Source](https://x.com/vmiss33/status/2050984556790939731)

</details>

<details>
<summary>Key Concept 7 — Mercury: The AI Agent We All Wanted - Where Control, Permissions, and Autonomy Finally Got Real</summary>

- **Mercury** — Open-source, background-native AI agent orchestrator by Cosmic Stack that addresses three core agent problems: (1) insecure permissions with unvetted third-party skills, (2) context-window bloat causing uncontrolled API costs, and (3) opaque identity systems buried in SQLite blobs. Features include permission-hardened architecture (scoped folder access, hard-blocked destructive commands), token discipline (400 tokens of core persona per request, daily budget with Auto-Concise mode), a four-file soul system (`soul.md`, `persona.md`, `taste.md`, `heartbeat.md`) for version-controlled identity, and zero-dependency daemon operation across macOS, Linux, and Windows.

> [Source](https://x.com/Ctrl_Alt_Zaid/status/2046902326657749114)

</details>

<details>
<summary>Key Concept 8 — Hermes Kanban Turns Multi-Agent Work Into a Real Board</summary>

- **Hermes Kanban**: An AI tool for coordinating multi-agent workflows, providing a durable Kanban board system with persistent task state, named agent profiles, and human supervision interfaces to manage long-running, complex agent work.

> [Source](https://x.com/NeoAIForecast/status/2051443615768228062)

</details>

<details>
<summary>Key Concept 9 — Building a Virtual Filesystem for Mintlify's AI Assistant</summary>

- **ChromaFs**: A virtual filesystem tool for AI agents that replaces sandbox-based filesystem access with a Chroma vector database-backed interface. Reduces latency from 46 seconds to 100ms, eliminates per-session infrastructure costs, and includes built-in RBAC via file tree pruning.

> [Source](https://x.com/densumesh/status/2039765361533637016)

</details>

---

_Built: 2026-05-05_