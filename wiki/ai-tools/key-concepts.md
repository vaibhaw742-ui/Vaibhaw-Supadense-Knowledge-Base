# AI Tools

> This file is managed by the supadense agent. Edit via chat only.

Useful AI tools, libraries, and platforms for building and using AI systems.

**Depth:** working

**Resources:** 4

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

---

_Built: 2026-05-03_