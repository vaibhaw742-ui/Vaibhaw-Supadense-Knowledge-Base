# AI Tools

> This file is managed by the supadense agent. Edit via chat only.

Useful AI tools, libraries, and platforms for building and using AI systems.

**Depth:** working

**Resources:** 4

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

---

_Built: 2026-05-03_