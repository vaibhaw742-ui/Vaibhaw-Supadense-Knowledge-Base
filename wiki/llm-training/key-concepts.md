# 🧠 LLM Training

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** working

**Resources:** 2

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

<details>
<summary>Key Concept 2 — Causal Masking in Attention</summary>

- **Causal Masking** — A mechanism applied in Transformer attention layers to block access to future tokens, ensuring each token attends only to itself and past tokens. Prevents information leakage during training, where seeing future tokens would allow models to use target tokens as inputs (cheating), and aligns training conditions with inference (where future tokens do not exist).
- **Causal Mask Matrix** — A binary matrix (1 = attention allowed, 0 = blocked) matching the dimensions of the attention score matrix. For causal masking, the upper triangle (future positions relative to each query token) is set to 0. Blocked positions are set to −∞ before softmax, which maps to 0 after softmax, eliminating attention to future tokens.
- **Attention Score Matrix** — A 2D matrix generated in attention layers where rows represent query tokens and columns represent key tokens. Each value indicates the raw relevance score between a query-key token pair before normalization or softmax is applied.
- **Information Leakage (Language Modeling)** — A failure mode where models access future tokens during training, allowing them to learn shortcuts using target tokens as input. This produces unrealistically high training performance but breaks the core language modeling assumption of predicting next tokens using only past context, causing failure during step-by-step inference.

> [Source](https://x.com/amitiitbhu/status/2038572952648815008)

</details>

---

_Built: 2026-05-13_