# ⚡ LLM Inference

> This file is managed by the supadense agent. Edit via chat only.

**Depth:** working

**Resources:** 8

## Key Concepts

<details>
<summary>Key Concept 1 — Hidden Technical Debt of AI Systems: Agent Runtime</summary>

Agent runtime is the serving infrastructure layer for agentic AI — analogous to how model serving infrastructure surrounds the ML code in traditional MLOps. The agentic model call is the small box; the runtime (compute substrate, filesystem, tools, network boundary, state model, lifecycle controller) is the large box where technical debt accumulates.

Key serving infrastructure considerations for agent workloads:
- **Sandboxing is mandatory** — agents execute arbitrary code and read untrusted tool outputs, so kernel-level isolation (VM per session, not shared-kernel containers) is required
- **Cold start matters more than throughput** — agent productivity is bounded by runtime startup time, not model tokens-per-second; sub-second snapshot restores are critical
- **Multi-tenancy at scale** — RL training spins up thousands of concurrent rollouts, each needing isolated filesystem, process tree, and network namespace
- **Firecracker microVMs** are the de facto industry primitive for serving untrusted agent code at high density (~125ms boot, ~5MB VMM overhead, dedicated kernel per VM)

> [Source](https://leehanchung.github.io/blogs/2026/04/24/hidden-technical-debt-agent-runtime/)

</details>

<details>
<summary>Key Concept 2 — Coding Agents are Effective Long-Context Processors</summary>

- **Long-Context Access vs Processing Gap**: LLMs can scale to access massive contexts (up to millions of tokens) via expanded context windows, but fail to effectively process long context, with performance degrading as context length increases (context rot).
- **Externalized Long-Context Processing**: Moving long-context processing from latent, uninterpretable attention mechanisms to explicit, executable interactions via coding agents, improving both performance and interpretability.

> [Source](https://arxiv.org/html/2603.20432v1)

</details>

<details>
<summary>Key Concept 3 — We don't need more agents. We need a better system. So we built one.</summary>

- **Model routing**: Process of dynamically selecting the most appropriate LLM for a given task based on cost, quality, and task requirements, to minimize token usage and operational cost without sacrificing output quality.
- **Multi-model inference**: Using multiple LLM providers/models in a single workflow, optimizing for cost and performance by routing tasks to the best-fit model rather than defaulting to a single frontier model for all tasks.

> [Source](https://x.com/augmentcode/status/2051350118360891584)

</details>

<details>
<summary>Key Concept 4 — 99% of Hermes Agent Users Have Never Touched These 15 Features</summary>

- **Mid-Session Model Swapping** — Switching the active LLM provider/model mid-session without restarting the agent, preserving all existing conversation state and context. Supports providers including Anthropic, OpenAI, OpenRouter, NVIDIA NIM, Gemini, and others.
- **Auxiliary Model Routing** — Routing non-core LLM tasks (context compression, session summarization, title generation, vision processing) to separate, task-optimized models to reduce inference costs and improve performance. For example, using a lightweight model for title generation instead of a high-cost frontier model.
- **Fast Inference Toggles** — Quick commands (e.g., `/fast`) to switch to priority or fast inference modes (e.g., OpenAI Priority, Anthropic Fast Mode) mid-session to reduce latency for time-sensitive tasks.

> [Source](https://x.com/shmidtqq/status/2051307460208578864)

</details>

<details>
<summary>Key Concept 5 — What I Use Hermes Agent For (And How I Use It)</summary>

- **Consumer-grade local inference**: Running 9B quantized models (Qwen 3.5 9B) with 64k context on 8GB VRAM (RTX 4070) or 16GB RAM (M1 MacBook) via llama.cpp, challenging the assumption that local LLMs require high-end hardware.
- **Multi-provider inference strategy**: Leveraging multiple LLM providers (OpenRouter free tier, Nous Portal $10/mo API, NVIDIA NIM free, DeepSeek API, ChatGPT Plus $20/mo non-API) to minimize costs while maintaining capability.
- **OpenRouter free tier optimization**: Adding $10 credits unlocks 1,000 requests/day (vs 50/day free); NVIDIA Nemotron 3 Super 120B-A12B highlighted as capable free model option.
- **Non-API LLM access**: Using ChatGPT Plus subscription to access GPT-5.5 through Hermes, avoiding per-request API costs while maintaining high capability for task execution.
- **Local model serving**: Using llama.cpp to serve local models with extended context (64k) on consumer hardware, accessible to agents over wireless network.

> [Source](https://x.com/vmiss33/status/2050984556790939731)

</details>

<details>
<summary>Key Concept 6 — Boosting multimodal inference performance by >10% with a single Python dictionary</summary>

- **Host Overhead** — Time spent in host-side (CPU) inference components like schedulers, which stalls GPU work submission; a critical bottleneck for multimodal inference workloads where schedulers are single-threaded.
- **SGLang Scheduler** — Single-threaded host-side loop in the SGLang inference engine that gates GPU work submission; processes incoming requests, forms batches, and dispatches them to the GPU.
- **CUDA IPC (Interprocess Communication)** — CUDA mechanism for sharing GPU tensors between processes without memory copies, used by SGLang to transfer preprocessed tensors from tokenizer workers to the scheduler.
- **CUDA IPC Pool Handle Cache** — Simple Python dictionary cache implemented in SGLang v0.5.10 to store reusable CUDA IPC memory pool handles, eliminating repeated `_new_shared_cuda` calls that caused ~3% of scheduler runtime overhead.
- **Mixed-Chunk Scheduling** — Scheduling strategy allowing prefill and decode tokens to share a single batch; delays in prefill input processing (e.g., multimodal feature hashing) also slow decode operations.
- **process_input_requests** — SGLang scheduler function responsible for preparing incoming multimodal requests for batching; previously consumed ~13% of scheduler CPU time due to redundant CUDA IPC handle lookups.
- **hash_feature** — SGLang function that maps input images to hash-based IDs for efficient KV cache lookups, part of the multimodal input processing path that required shared GPU memory book-keeping.
- **py-spy** — Low-overhead sampling profiler for running Python processes, used to identify SGLang scheduler bottlenecks by generating flamegraphs of CPU time usage.

> [Source](https://x.com/modal/status/2051435165097091085)

</details>

<details>
<summary>Key Concept 7 — Causal Masking in Attention</summary>

- **Causal Masking** — A mechanism in Transformer attention layers that restricts each token to attending only to itself and past tokens, never future ones. Critical for autoregressive inference, where tokens are generated sequentially and future tokens do not exist, preventing invalid attention to non-existent context.
- **Causal Mask Matrix** — Binary matrix applied to attention score matrices during inference, with 0s in upper triangular positions (future tokens) set to −∞ before softmax to zero out attention to invalid positions. Ensures inference behavior matches training conditions.
- **Attention Score Matrix** — A 2D matrix generated in attention layers where rows represent query tokens and columns represent key tokens. Each value indicates the raw relevance score between a query-key token pair before normalization or softmax is applied.

> [Source](https://x.com/amitiitbhu/status/2038572952648815008)

</details>

<details>
<summary>Key Concept 8 — LLM Inference Optimization</summary>

- **KV Cache** — Stores the Key (K) and Value (V) matrices computed at each decoding step, so that every new token only computes attention against the cached keys/values rather than recomputing them for the full sequence. This reduces per-token attention complexity from O(n²) to O(n) and is the foundation of efficient autoregressive inference in Transformers.

- **Paged Attention** — A memory management technique that eliminates KV Cache fragmentation by allocating fixed-size "pages" (blocks) of memory, similar to virtual memory paging in operating systems. This allows non-contiguous storage of KV Cache blocks, dramatically reducing wasted memory and enabling LLM servers to serve many more concurrent users.

- **Flash Attention** — An IO-aware attention algorithm that tiles the attention computation into small blocks that fit in fast GPU on-chip SRAM instead of slow HBM. Uses online softmax to avoid materializing the full N×N attention matrix, and recomputes attention in the backward pass to save memory bandwidth. Achieves 2–4× speedup over standard attention and is used in virtually every modern LLM training and inference pipeline (Flash Attention 2 and 3 introduce further optimizations).

- **Grouped-Query Attention (GQA)** — An attention variant where multiple query heads share a single key-value head group (e.g., 2:1 or 8:1 ratio), reducing the memory footprint of KV Cache during inference. Generalizes Multi-Head Attention (MHA, where G=H, each query head has its own KV head) and Multi-Query Attention (MQA, where G=1, all queries share one KV head). Used in production by models like Llama 2 70B, PaLM, and others.

- **Speculative Decoding** — A two-model inference strategy where a small, fast "draft" model proposes multiple candidate tokens in a single forward pass, and a large "target" model verifies them in parallel using rejection sampling. Guarantees identical output distribution to the target model alone while achieving 2–3× wall-clock speedup in production (used by TGI, vLLM, TensorRT-LLM, and others).

- **Continuous Batching** — A serving technique where the inference engine dynamically adds new sequences to the GPU batch and evicts completed ones at every single generation step (instead of waiting for all sequences in a batch to finish). This keeps GPU utilization high throughout the decode phase, significantly improving overall throughput for LLM serving compared to static batching.

> [Source](https://x.com/amitiitbhu/status/2054100147546837154)

</details>

---

_Built: 2026-05-13_