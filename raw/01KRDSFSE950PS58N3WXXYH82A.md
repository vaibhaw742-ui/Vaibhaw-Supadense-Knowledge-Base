# LLM Inference Optimization

[![Amit Shekhar](https://pbs.twimg.com/profile_images/1550394637075943424/PAdpHzNV_normal.jpg)](https://x.com/amitiitbhu)

**Amit Shekhar** [@amitiitbhu](https://x.com/amitiitbhu)

[![LLM Inference Optimization](https://pbs.twimg.com/media/HIGbZHxagAATpgv?format=jpg&name=small)](https://x.com/amitiitbhu/article/2054100147546837154/media/2054093760942997504)

Techniques like KV Cache, Paged Attention, Flash Attention, Speculative Decoding, and Continuous Batching are what make LLMs fast and scalable in production.

Let’s learn all of these techniques one by one.

---

## KV Cache in LLMs

In this blog, we will learn about KV Cache - where K stands for Key and V stands for Value - and why it is used in Large Language Models (LLMs) to speed up text generation.

We will start with how LLMs generate text one token at a time, understand the role of Key, Value, and Query inside the model, see the problem of repeated computation through an example, and then walk through how KV Cache solves this problem by storing and reusing past results.

Read: [https://outcomeschool.com/blog/kv-cache-in-llms](https://outcomeschool.com/blog/kv-cache-in-llms)

---

## Paged Attention in LLMs

In this blog, we will learn about Paged Attention, a technique that solves the memory waste problem of KV Cache, allowing LLMs to serve many more users at the same time.

We will start with a quick recap of KV Cache, understand the memory problem it creates, see how traditional memory allocation wastes space through an example, and then walk through how Paged Attention solves this problem by borrowing an idea from how computers manage memory.

Read: [https://outcomeschool.com/blog/paged-attention-in-llms](https://outcomeschool.com/blog/paged-attention-in-llms)

---

## Decoding Flash Attention in LLMs

In this blog, we will learn about Flash Attention by decoding it piece by piece - understanding why standard attention is slow, what makes Flash Attention fast, how it uses GPU memory cleverly, and why it is used in almost every modern Large Language Model (LLM).

We will cover the following:

- A quick recap of standard attention
- Why standard attention is slow
- How GPU memory actually works (HBM vs SRAM)
- The core idea behind Flash Attention
- Tiling: breaking the work into small blocks
- Online softmax: computing softmax without the full matrix
- Recomputation in the backward pass
- Flash Attention 2
- Flash Attention 3
- Advantages and impact of Flash Attention

Read: [https://outcomeschool.com/blog/decoding-flash-attention](https://outcomeschool.com/blog/decoding-flash-attention)

---

## Grouped Query Attention

In this blog, we will learn about Grouped-Query Attention (GQA) and how it differs from Multi-Head Attention (MHA).

Today, we will cover the following topics:

- Quick Recap: Multi-Head Attention (MHA)
- The Problem with Multi-Head Attention
- What is Multi-Query Attention (MQA)?
- What is Grouped-Query Attention (GQA)?
- How Grouped-Query Attention Works
- GQA is a Generalization of MHA and MQA
- GQA vs MHA vs MQA
- Real-World Use Cases
- A Note on Terminology
- Uptraining: Converting MHA to GQA
- Quick Summary

Read: [https://outcomeschool.com/blog/grouped-query-attention](https://outcomeschool.com/blog/grouped-query-attention)

---

## Speculative Decoding

In this blog, we will learn about Speculative Decoding - what it is, why LLM generation is slow without it, how a small draft model and a big target model work together to produce tokens faster, the rejection sampling math that guarantees no quality loss, real numbers showing the 2x to 3x speedup, where it is used in production, and the trade-offs to watch out for.

We will cover the following:

- What problem does Speculative Decoding solve?
- The Big Picture
- Why is LLM generation slow?
- The core idea behind Speculative Decoding
- Step-by-step walkthrough
- The verification step
- Real numbers and speedup
- Where it is used
- Trade-offs
- Quick Summary

Read: [https://outcomeschool.com/blog/speculative-decoding](https://outcomeschool.com/blog/speculative-decoding)

---

## Continuous Batching in LLMs

In this blog, we will learn about Continuous Batching, a technique that lets LLM servers handle many more users at the same time by keeping the GPU busy at every single step of generation.

We will cover the following:

- Quick Recap: How an LLM Generates Tokens
- Why Batching Matters for LLMs
- The Old Way: Static Batching
- The Problem with Static Batching
- What is Continuous Batching?
- The Ride-Share Analogy
- How Continuous Batching Works Step by Step
- A Numeric Example
- Real Numbers and Speedup
- Benefits of Continuous Batching
- A Few Important Notes
- Quick Summary

Read: [https://outcomeschool.com/blog/continuous-batching-in-llms](https://outcomeschool.com/blog/continuous-batching-in-llms)

---

That's it for now.

Thanks

Amit Shekhar Founder @ [Outcome School](https://outcomeschool.com/)