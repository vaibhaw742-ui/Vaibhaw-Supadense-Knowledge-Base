# 🧠 LLM Training

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** working

**Resources:** 1

## Key Concepts

<details>
<summary>Key Concept 1 — On SFT, RL, and on-policy distillation</summary>

- **SFT-RL Pipeline Tipping Point**: Standard post-training runs SFT first (fixed sampling from teacher, ceiling at teacher's level), then RL (compounding sampling from student rollouts, ceiling at verifier's capability). SFT is most efficient when current performance is far below the teacher; once the student approaches the teacher's level, marginal SFT examples become uninformative and RL becomes more effective.

- **SFT Sampling Ceiling**: Teacher SFT uses a fixed sampling distribution set at dataset construction time. Rejection-sampled SFT (SFT-RS) improves over vanilla SFT but retains the same fundamental limitation — once the filter saturates, the ceiling remains the teacher's performance level.

- **RL Gradient Geometry**: RL's apparent sparsity (binary reward spread across thousands of tokens) keeps gradients honest. In GRPO-style updates, most per-token advantages are noise, but destructive interference leaves a small consistent bias along dimensions that actually correlate with reward.

- **Same-Family Requirement for OPD**: On-policy distillation requires tokenizer-matched and recipe-matched teachers (e.g., Qwen3-32B teaching Qwen3-8B-Base). This enables direct per-token logprob comparison and reverse-KL computation. Same-family OPD achieves ~9–30× compute savings vs RL on AIME-style benchmarks.

- **OPD vs RL Ceiling**: OPD's ceiling is bounded by the teacher's performance (targets teacher distribution by construction), while RL's ceiling is bounded by the verifier (which can be much higher). OPD is not "better than RL" — it gets you to the teacher's level much faster, which is usually sufficient.

- **Exposure Bias in Off-Policy SFT**: SFT and SFT-RS train under the teacher's state distribution but evaluate the student under its own distribution. This gap limits practical off-policy performance short of true teacher quality on long rollouts. OPD avoids this by training on student rollouts.

- **Self-Distillation Without External Teacher**: When no same-family teacher is available, self-distillation uses the student as its own teacher with privileged context — SDFT (expert demonstration in teacher context) or OPSD (ground-truth answer in teacher context). Eliminates tokenizer/recipe mismatch but the privileged info shifts the teacher's distribution.

> [Source](https://x.com/willccbb/status/2050038277454143918)

</details>

---

_Built: 2026-05-03_