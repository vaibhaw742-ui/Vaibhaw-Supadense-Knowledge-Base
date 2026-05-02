# How Hermes Agent Solves Skill Drift and Context Rot as a Self-Improving Agent

![Profile: mem0](https://pbs.twimg.com/profile_images/2038726039153881088/b3jpb6xu_normal.jpg)
[mem0](https://x.com/mem0ai) [@mem0ai](https://x.com/mem0ai)

![Hermes Curator Screenshot](https://pbs.twimg.com/media/HHRMCfCagAAl4Qo?format=jpg&name=small)

Self-improving agents have a skills-hoarding problem. Every skill the agent writes stays forever, even the unused ones. Hermes Agent shipped Hermes Curator to fix it. The skills catalog grows and the agent ends up rereading its own old work on every prompt instead of solving the new one.

Yesterday, [@nousresearch](https://x.com/@nousresearch) just shipped Hermes Curator.

Hermes Curator is a background maintenance pass for agent-created skills. It tracks how often each skill is viewed, used, and patched, moves long-unused skills through active → stale → archived states, and periodically spawns a short auxiliary-model review that proposes consolidations or patches drift.

It exists so that skills created via the [self-improvement loop](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills#agent-managed-skills-skill_manage-tool) don't pile up forever.

---

![Profile: Nous Research](https://pbs.twimg.com/profile_images/1816254738234761216/TX7TW-Mp_normal.jpg)
[Nous Research](https://x.com/NousResearch) @NousResearch · [Apr 30](https://x.com/NousResearch/status/2049956455982182593)

Hermes Agent v0.12.0 - “The Curator Release”

![Curator Release](https://pbs.twimg.com/amplify_video_thumb/2049949877660954624/img/SkqE9Zpgccbqr1sW.jpg)

---

## How does Hermes Curator manage skills?

Hermes agents save skills as they learn. Every "agent-created" saved skill becomes a file in your skills folder. Without maintenance, that folder grows forever, the agent loads more of it on every prompt, and the catalog turns into noise.

Curator is the cleanup system [@nousresearch](https://x.com/@nousresearch) to prevent that.

It only runs when you're not working. Roughly once a week, when the agent has been idle for at least two hours, it wakes up and runs a single review pass in the background. The pass has two parts.

![Skill Lifecycle](https://pbs.twimg.com/media/HHRMdjNbEAAMELK?format=jpg&name=small)

*Figure 1. Hermes Curator skill lifecycle: telemetry-driven state machine plus a forked review pass*

The first part is automatic. Curator checks the last-used time on every skill. Anything untouched for 30 days gets marked stale. Anything untouched for 90 days gets moved into an archive folder. Nothing is ever deleted, and any archived skill can be brought back with one command.

The second part uses a separate, cheaper model. That model reads the skills the agent has written, decides which ones overlap, which ones have drifted, and whether any of them should be merged, patched, or archived. Because it runs on an auxiliary slot, you can point it at a cheap model like Gemini Flash instead of paying main-chat-model rates for housekeeping.

> All of these numbers are configurable in config.yaml under curator:. If 30 days is too aggressive for your workflow, raise it. If you want a daily review pass, drop the interval.

You can manage it through the config.yaml or Hermes Curator CLI.

There are three guardrails worth calling out:

- Curator only ever touches skills the agent wrote or you wrote by hand. Skills that shipped with Hermes or skills you installed from the hub are off-limits.
- You can pin any skill. Pinned skills are invisible to the automatic clock and to the review model. Even the agent's own editing tools refuse to modify a pinned skill. This is the lever you use to protect anything you depend on.
- Every counter and timestamp Curator uses lives in one sidecar file inside the skills folder. You can read it. You can audit it. There is no hidden database.

---

## What Hermes Curator does?

Curator does four things:

1. **It watches.** Every time a skill is loaded into a prompt or read by the agent, a counter ticks up and a timestamp gets written. That sidecar file is the memory of how useful each skill has actually been.
2. **It demotes.** If a skill has not been touched in 30 days, it moves from active to stale. Stale skills still work, but the system has flagged them as suspicious. If 60 more days pass and nothing has touched them, they get moved out of the live folder into an archive folder. Nothing is deleted. You can always pull a skill back with one command.
3. **It reviews.** Roughly once a week, when the agent is idle for a couple of hours, a separate cheap model wakes up and reads the skills folder. It looks for skills that drift, skills that overlap, and skills that should be merged into one cleaner version. It can patch them, combine them, or send them to the archive. Then it goes back to sleep.
4. **It respects pins.** Anything you mark as pinned is invisible to the demotion clock and to the review model. Even the agent itself cannot rewrite a pinned skill. This is the lever that keeps anything you depend on from drifting underneath you.

That is the entire user-visible behavior. Watch, demote, review, respect pins.

---

## Curator CLI

You don't have to wait for the weekly curator cleanup. Curator ships with a small CLI surface that lets you check on it, trigger a run, pause it, or protect specific skills.

```bash
hermes curator status              # last run, counts, pinned list, LRU top 5
hermes curator run                 # trigger a review now (background by default)
hermes curator run --sync          # same, but block until the LLM pass finishes
hermes curator pause               # stop runs until resumed
hermes curator resume
hermes curator pin <skill>         # never auto-transition this skill
hermes curator unpin <skill>
hermes curator restore <skill>     # move an archived skill back to active
```

The most useful one day to day is `hermes curator status`. Beyond the obvious last-run timestamp and counts, it also lists the five least-recently-used skills, which is a quick way to see what is most likely to go stale next. If you spot something on that list you actually want to keep, `hermes curator pin` it before the next run.

---

## Why this matters now

The math is simple. Even at one new skill saved per day, the catalog hits 30 in a month and 365 by year one. With Curator's review pass merging near-duplicates, only about 3 unique skills survive per month. That is roughly 36 a year instead of 365.

![Skills Comparison](https://pbs.twimg.com/media/HHRMyTZakAA75R1?format=png&name=small)

*Figure 2. Comparison of number of agent-created skills with Hermes Curator*

Most of those "agent-created" skills are not unique. The agent will have written fix-a-bug five different ways across five different days, none of them aware the others exist.

Curator's Hermes Curator review pass exists exactly for this: it reads the catalog, recognizes the overlap, and consolidates near-duplicates into one canonical skill while moving the redundant copies to the archive.

Two things go wrong if nothing prunes:

1. The token bill goes up, because every prompt pays to read a bigger and bigger directory.
2. The reasoning gets worse, because five near-duplicate skills all match the same query and the model has no good signal for which to pick.

Sometimes it picks the wrong one. Sometimes it tries to use two at once. Either way, the agent gets slower and dumber as the catalog grows.

Curator's design choices read like scar tissue. It runs only once a week, only when the agent has been idle for at least two hours, and always in a separate process, because the team does not want a cleanup pass interfering with your live work. The pin feature exists because the agent, left alone, will rewrite skills you depend on. Pin is the same contract mem0 ships as core memories: facts the agent is forbidden from overwriting. Different surface, identical idea.

---

## Memory and Skills

It is worth being precise about what kind of "memory" Curator handles, because the field tends to mix two very different things into the same word.

Memory, in the way most people use it, is what the agent knows. Facts about the user, the project, and the domain. Past conversations and the decisions that came out of them. References the agent needs to keep within reach.

Memory holds the context the agent operates inside. The memory layer is where this lives, and mem0 is what manages it.

![Memory and Skills Layer](https://pbs.twimg.com/media/HHRM2H8bwAAvud8?format=jpg&name=small)

*Figure 3. Memory and Skills Layer*

Mem0 is available as one of the Hermes Agent memory provider. You can do a quick 30-second setup, or use our open source GitHub repository and self-host. Read our guide here:

![Profile: mem0](https://pbs.twimg.com/profile_images/2038726039153881088/b3jpb6xu_normal.jpg)
[mem0](https://x.com/mem0ai) @mem0ai · [Apr 3](https://x.com/mem0ai/status/2040149098364580026)

![Article cover image](https://pbs.twimg.com/media/HFAQs9-bcAAHm1I?format=jpg&name=small)

**Article:** How to Add Memory to Your Hermes Agent

Hermes Agent just added 6 memory providers. Mem0 is one of them. Setup takes one command. Circuit breaker if anything fails. The memory system handle storage, retrieval, and context across sessions....

Skills are different. A skill is procedural knowledge live on the core of agent action and projects. It is a how-to file the agent wrote for itself: how to book a flight, how to refactor a file, how to query a particular API. Skills tell the agent how to do work. The skill layer is where this lives, and Hermes Curator is what manages it.

These two systems do not compete. They handle different parts of the same problem: keeping the agent's accumulated state useful instead of just heavy. Memory consolidation handles what the agent knows. Curator handles how the agent works. Together, they cover both halves of what a long-running agent needs to maintain.

> The reason this matters: it is easy to look at Curator and think "this is just memory management". They sit at different layers, decay at different rates, and need different cleanup logic. Building one does not replace the other. Building both is what gets you an agent that actually keeps learning.

---

## The pattern

The shape of an agent runtime keeps growing.

A few years ago an agent was a model and a prompt. Then we added

- a tool registry so it could act
- a memory layer so it could remember across conversations
- long-context window tricks so it could read more in a single shot

Today, a serious agent runtime has more plumbing than agent. There is a serving layer with KV cache eviction. A long-term memory layer with periodic consolidation, run by its own background model. A skills layer with usage tracking, state transitions, pinning, and a separate review pass.

That layer that manage the skills layer is Hermes Curator.

There is a tool layer with permissioning, a prompt cache with expiry, and an orchestrator that decides which background job runs when.

The agent is actually maintaining four different kinds of memory at once:

- Working memory in the cache, used and discarded each turn.
- Semantic memory in the vector store, holding the facts the agent can look up.
- Episodic memory in the conversation log, the record of what happened.
- Procedural memory in the skills folder, the how-to library.

Each of these decays at its own rate, fills up at its own pace, and needs its own quiet cleanup process running in the background.

---

## Takeaway

Hermes Curator is what cleanup should look like for an agent that learns. It watches what gets used, retires what doesn't, merges duplicates, and lets you pin anything you don't want touched. The behavior is simple. The surprising part is that nobody had built it yet.

If you are building agents that save their own skills, a cleanup pass is not optional infrastructure for much longer.

Every week without a context management layer means more skills for your future self to sort through, and a greater chance the skills you depend on get lost in the noise.

> Self-improving agents need self-pruning systems and context management.

---

### In Context #8

This blog is part of In Context, a mem0 blog series covering AI Agent memory and context engineering.

[mem0](https://mem0.ai/?utm_source=x_article_mem0&utm_medium=incontext_article&utm_campaign=curator_memory&utm_content=curator_memory) is an intelligent, open-source memory layer designed for LLMs and AI agents to provide long-term, personalized, and context-aware interactions across sessions.

- Get your free API Key here: [app.mem0.ai](https://app.mem0.ai/?utm_source=x_article_mem0&utm_medium=incontext_article&utm_campaign=curator_memory&utm_content=curator_memory)
- or self-host mem0 from our open source [github repository](https://github.com/mem0ai/mem0)
