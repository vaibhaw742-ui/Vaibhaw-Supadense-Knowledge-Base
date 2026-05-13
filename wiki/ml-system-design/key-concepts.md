# ML System Design

> This file is managed by the supadense agent. Edit via chat only.

building ML pipeline end to end at production with data + feature engineering + training + inference etc

**Depth:** working

**Resources:** 3

## Key Concepts

<details>
<summary>Key Concept 1 — Agent observability needs feedback to power learning</summary>

- **Agent Observability Platform Requirements** — Three core capabilities to power learning: (1) Store full agent traces, (2) Store feedback directly attached to traces, (3) Generate feedback via rules, LLM-as-judge, or automated evaluators.
- **Deterministic Feedback** — Rule-based (regex, heuristics) signals for known failure patterns (e.g., detecting user frustration via keyword matching); low-cost alternative to model-based feedback for high-signal, well-understood failure modes.
- **LLM-as-Judge Feedback** — Using an LLM to automatically score agent outputs or trajectories for helpfulness, policy adherence, or anomalies; scalable for production traces but requires calibration.
- **Implicit User Feedback** — Indirect signals of agent performance: code accepted/reverted, tests passed, tickets reopened, user repeating questions; noisier than explicit ratings but higher volume.
- **Explicit User Feedback** — Direct user signals: thumbs up/down, star ratings, written corrections; high clarity but typically sparse.

> [Source](https://x.com/hwchase17/status/2051708710859501807)

</details>

<details>
<summary>Key Concept 2 — How to Build a Self-Improving AI Lead Gen Agent on Hermes</summary>

- **Pattern-over-fluke belief updating** — A design rule for self-learning agent systems where belief updates (e.g., copy patterns, content formats) only trigger when a pattern repeats enough times to cross a validation threshold, avoiding updates based on single viral posts or one-off successes.
- **Version-controlled agent beliefs** — A design principle where agent beliefs (validated patterns about audience, formats, markets) are version-controlled, allowing old beliefs to be overwritten when new experimental data contradicts them, preventing the system from locking into outdated playbooks.
- **Multi-layer content engine** — A content generation architecture for agents with three input layers (signal layer monitoring trends, understanding layer ingesting long-form content for depth, internal context layer pulling from operational data) and one output layer that assembles inputs into personalized Substack, LinkedIn, and YouTube content.

> [Source](https://x.com/MichLieben/status/2051707320699396454)

</details>

<details>
<summary>Key Concept 3 — Agent Platform That Builds Itself</summary>

- **Agent Platform Architecture**: A 5-component design pattern for running agents in production:
  1. **Runtime**: The core service that runs agents — handles requests, executes agent loops, streams responses, writes to storage, and manages authentication
  2. **Storage**: Database layer for persisting agent sessions, memory, knowledge, traces, and evaluation history
  3. **Connectors**: Centralized tools for agents to interface with external systems via MCP, API, or CLI — provides a single point for security enforcement
  4. **Interfaces**: Multi-surface access layer (Slack, Discord, Telegram, custom UIs) with unified identity resolution (same user_id across different surfaces)
  5. **Infrastructure**: Deployment layer — Docker for local development, Railway or any cloud provider for production

- **Unified Identity Resolution**: A design pattern where a single user is mapped to the same user_id regardless of which interface they use (Slack, web app, Discord, etc.), enabling consistent agent interactions across surfaces.

> [Source](https://x.com/ashpreetbedi/status/2054222428025614829?s=20)

</details>

---

_Built: 2026-05-13_