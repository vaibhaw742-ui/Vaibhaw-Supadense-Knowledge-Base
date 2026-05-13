# 🔍 RAG

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** deep

**Resources:** 2

## Key Concepts

<details>
<summary>Key Concept 1 — Coding Agents are Effective Long-Context Processors</summary>

- **RAG Long-Context Limitations**: Standard RAG pipelines rely on fixed, shallow retrieval mechanisms that cannot support iterative, multi-hop reasoning where intermediate findings must guide subsequent queries, limiting flexibility for complex long-context tasks.
- **Coding Agent Superiority Over RAG**: Coding agents outperform published RAG state-of-the-art by 17.3% average across long-context benchmarks, as they avoid fixed retrieval stages and enable iterative, programmatic processing.
- **Retrieval Tool Integration Limitation**: Equipping coding agents with standard retrieval tools (BM25, dense embeddings) does not consistently improve performance, as agents already leverage more flexible native tooling.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

<details>
<summary>Key Concept 2 — Building a Virtual Filesystem for Mintlify's AI Assistant</summary>

- **RAG chunk reassembly**: For vector databases that split pages into chunks for embedding, full pages can be reconstructed by fetching all chunks matching a page slug, sorting by chunk index, and joining the content. This is used in ChromaFs to serve full documentation pages via `cat` commands.
- **ChromaFs**: A virtual filesystem backed by a Chroma vector database (commonly used for RAG workloads) that translates UNIX filesystem commands into Chroma queries. It allows RAG-powered agents to explore documentation using standard filesystem tools, overcoming traditional RAG limitations where answers span multiple chunks or require exact syntax not present in top-K results.
- **Coarse-to-fine grep optimization**: For vector-backed virtual filesystems, grep commands are optimized by first using the vector database as a coarse filter to identify candidate files matching the search pattern, bulk-prefetching matching chunks into a cache (e.g., Redis), then running fine-grained in-memory grep only on matched files to avoid slow network-wide scans.

> [Source](https://x.com/densumesh/status/2039765361533637016)

</details>

---

_Built: 2026-05-13_