# 🔍 RAG

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** deep

**Resources:** 1

## Key Concepts

<details>
<summary>Key Concept 1 — Coding Agents are Effective Long-Context Processors</summary>

- **RAG Long-Context Limitations**: Standard RAG pipelines rely on fixed, shallow retrieval mechanisms that cannot support iterative, multi-hop reasoning where intermediate findings must guide subsequent queries, limiting flexibility for complex long-context tasks.
- **Coding Agent Superiority Over RAG**: Coding agents outperform published RAG state-of-the-art by 17.3% average across long-context benchmarks, as they avoid fixed retrieval stages and enable iterative, programmatic processing.
- **Retrieval Tool Integration Limitation**: Equipping coding agents with standard retrieval tools (BM25, dense embeddings) does not consistently improve performance, as agents already leverage more flexible native tooling.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

---

_Built: 2026-05-04_