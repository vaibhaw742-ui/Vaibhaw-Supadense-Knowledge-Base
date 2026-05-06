# AI Research Papers

> This file is managed by the supadense agent. Edit via chat only.

AI research papers, publications, and academic insights.

**Depth:** working

**Resources:** 3

## Key Concepts

<details>
<summary>Key Concept 1 — On SFT, RL, and on-policy distillation</summary>

- **On-policy distillation (OPD)** (Lu et al. 2025, also in Qwen3 technical report): Student samples own rollouts, each token graded by same-family teacher via per-token reverse KL. Achieves ~9–30× compute savings vs RL on AIME-style benchmarks. Ceiling bounded by teacher performance. Requires tokenizer and recipe match.

- **SDFT** (Shenfeld et al. 2026): Self-distillation variant where teacher is conditioned on an expert demonstration (not seen by student at sampling time). Teacher computes per-token logprobs over student trajectories; student updated via reverse KL. Provides distributional pull without leaking answers.

- **OPSD** (Zhao et al. 2026): Self-distillation variant where teacher is conditioned on the ground-truth answer. Produces sharper distributional shift than SDFT. Same setup otherwise — fully on-policy with reverse-KL updates.

- **SFT-RL Tipping Point**: Theory that SFT is most efficient when student performance is far below teacher; once near teacher level, RL (with compounding sampling) becomes more effective. RL ceiling determined by verifier, not data.

- **Exposure Bias in Off-Policy SFT**: Training on teacher's state distribution while evaluating on student's own distribution creates a gap that caps practical performance below teacher level on long rollouts. On-policy methods (OPD, RL) avoid this.

> [Source](https://x.com/willccbb/status/2050038277454143918)

</details>

<details>
<summary>Key Concept 2 — Coding Agents are Effective Long-Context Processors</summary>

- **Text Processing as File System Operation**: Methodology of structuring text corpora as directory hierarchies and delegating processing to coding agents with full autonomy over exploration and manipulation strategies.
- **Corpus Formatting for Coding Agents**: For corpora >100M tokens: each document as individual txt file in a directory; for prohibitively large corpora (e.g., Natural Questions): single JSONL file; for single long documents: single txt file.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

<details>
<summary>Key Concept 3 — Boosting multimodal inference performance by >10% with a single Python dictionary</summary>

- **Performance Profiling Methodology (Inference)** — When debugging inference performance, check host-side overhead before diving into GPU-level analysis (e.g., CUDA kernels, warp stall reasons). Tools like py-spy can sample call stacks from running Python processes with minimal overhead to generate flamegraphs identifying CPU bottlenecks.
- **Iterative Optimization Process** — Inference performance engineering follows a "relentless grind" pattern: identify bottlenecks via profiling, hypothesize fixes, implement targeted changes (e.g., caching redundant operations), validate with flamegraphs, and measure end-to-end outcomes (throughput, latency).
- **Host-First Debugging Principle** — The "golden rule" of inference performance: never block the GPU. When throughput plateaus below hardware capacity, investigate host-side components (schedulers, preprocessors) before optimizing GPU kernels.

> [Source](https://x.com/modal/status/2051435165097091085)

</details>

---

_Built: 2026-05-05_