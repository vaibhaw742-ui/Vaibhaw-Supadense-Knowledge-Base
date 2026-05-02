# Agent Memory Engineering

by [Nicolas Bustamante](https://x.com/nicbstme) [@nicbstme](https://x.com/nicbstme)

![Nicolas Bustamante](https://pbs.twimg.com/profile_images/1859460270613200898/thqbDVjW_normal.jpg)

![Agent Memory Engineering](https://pbs.twimg.com/media/HHQhwqoa4AAZcS4?format=png&name=small)

## How do agents actually remember me and my instructions? And why is moving from one agent's memory to another's so much harder than just copying files?

I often use Claude Code and Codex side by side. At work, I use the GitHub Copilot CLI routing tasks between Anthropic and OpenAI models depending on what I am doing. Same workstation. Same files. Same bash. Three different agent harnesses and I noticed something off about memory.

Feedback rules I had patiently taught Claude Code over hundreds of sessions, the kind that live in `~/.claude/projects/<encoded-cwd>/memory/` as little typed markdown files, did not seem to land the same way when I switched into a Codex session. A Codex memory citation about a workflow did not get the same weight when I crossed back into Claude Code. The two agents technically had access to similar information through similar tools. The behavior around memory was visibly different.

That sent me down a rabbit hole. I expected it to be a config detail, the kind of thing you fix with a setting. I think it's bigger than that.

The reason memory does not transfer cleanly between agents is that models are post trained on their harness. Claude was post trained against Claude Code's memory layer: the typed file taxonomy, the always loaded MEMORY.md index, the age aware `<system-reminder>` framing on every body read.

GPT-5 was post trained against Codex's memory layer: the always loaded `memory_summary.md`, the on demand grep into MEMORY.md, the `<oai-mem-citation>` block format the model uses to mark which memory it actually applied. The model's instinct for "remember this for next time" is shaped by the exact UI it saw during post training.

Which means switching is not a file copy. A user with 64 well loved memory entries built up against Claude Code cannot drop them into Codex's folder and expect them to behave the same. The bytes land but the behavior differs. The model does not know to read them with the same discipline, does not know to verify them with the same skepticism, does not know to cite them with the same tag. Annoying!

So it's not about raw model capability, not tool calling. Memory is the layer where the model and the harness fuse, and once that fusion is cooked into your daily flow, going back is unbearable. With memory, I outsource the persona of "what the user wants" to the agent. Without memory, I am the persona, every single turn, forever. And once the persona is fused with a specific harness, the switching cost compounds session over session.

> So how does memory actually work under the hood? Why is each agent's harness its own little universe? And what does the implementation look like when you read the code?

I dug into three open implementations that ship in production today: Hermes (Nous Research, Python, fully open source), Codex CLI (OpenAI, Rust, fully open source), and Claude Code (Anthropic, closed binary but the auto memory artifacts and live system reminders are visible from inside any session). I played with the harness and audited my own `~/.claude/projects/` directory of 64 memory files, and stress tested the edges.

Here is what I learned. The TL;DR up front: every clever architecture lost. The simple thing won. LLM plus markdown plus a bash tool. That is the entire stack. The interesting question is not "what data structure" but "what discipline does the agent follow when reading and writing it."

Here's what I'll cover:

- Why the Clever Architectures Lost — Vector DBs, knowledge graphs, dedicated memory agents, all came in second to a markdown file
- The Three Architectures — Bounded snapshot vs two phase async pipeline vs typed live writes
- Storage Layer — Section sign delimiters vs YAML frontmatter vs strict block schemas
- How Memory Loads Into the System Prompt — Where the bytes go and why placement matters
- The Prefix Cache Problem — Why Hermes freezes the snapshot and what it sacrifices
- The Two Phase Pipeline — Cron jobs, small extraction models, and big consolidation models
- The Signal Gate — Telling the agent when NOT to remember
- Memory Limits and Eviction — Char caps vs usage decay vs no cap at all
- The Verification Discipline — Why Claude Code wraps every read with an age warning
- Day 1 Bootstrap — The cold start problem nobody has solved yet
- What This Means for Agent Design — Five questions every memory system must answer

---

## Why the Clever Architectures Lost

For two years, every memory startup pitched the same idea. The agent has a vector database. Inferences are embedded. Retrieval happens via semantic similarity. A background "memory agent" runs separately, watches the conversation, decides what to encode, writes it into the store, runs RAG over the embedding space at retrieval time. Sometimes there is a knowledge graph layered on top. Sometimes a relational store. Sometimes a temporal index. Every memory company you have ever heard of had a slide deck with this architecture.

It works just well enough to ship a demo and just poorly enough that nobody actually keeps using it.

The reasons are by now well rehearsed. Embeddings are lossy. Semantic similarity over short fact strings is noisy. Retrieval misses the obvious thing and surfaces the irrelevant thing. The background agent never knows when to fire. Knowledge graphs require schemas, and the schemas never survive contact with real conversation. The cost of running an embedding model on every turn adds up. Debugging is a nightmare because the store is opaque, the retrieval ranking is opaque, and when the agent says something wrong, you cannot point at the bytes that produced the answer.

Now look at what is winning in production:

![No vector database](https://pbs.twimg.com/media/HHQY1HWb0AAbt_g?format=jpg&name=small)

No vector database. No embedding store. No semantic search. No background memory agent watching every turn. The agent has a Read tool, a Write tool, an Edit tool, and a bash tool, and it uses these to read and write markdown files just like a human would.

The lesson generalizes. Agents do not need bespoke memory infrastructure. They need primitive filesystem tools, a markdown convention, and prompt discipline. That is it. The same pattern is now showing up in skills (markdown files in folders), in plans (markdown files in folders), in checklists (markdown todo files). The infrastructure that won is the same infrastructure software engineers have used for forty years: text files plus grep.

The interesting design questions live one level up. Where does the markdown live in the prompt? Who decides what to write? How do you keep the prompt cache from breaking every turn? When does an old memory get pruned? That is the rest of this article.

---

## The Three Architectures

The model matters less than the write path. All three systems use frontier models for the live agent loop. The differences are in when memory gets written, who writes it, and how it gets back into the next turn.

![Three architectures](https://pbs.twimg.com/media/HHQY7ncagAAh-Bj?format=jpg&name=small)

Three completely different bets.

- **Hermes** bets on simplicity and prefix cache stability. One file. Two stores. Char ceiling. Snapshot frozen at session start. The agent writes synchronously inside the turn. The bytes hit disk immediately, but the system prompt does not change for the rest of the session. New writes become visible on the next session boot. Total prompt budget for memory: ~2200 chars on MEMORY.md plus ~1375 chars on USER.md. That is the whole thing.

- **Codex** bets that the live turn should be cheap and the offline pipeline should be heavy. The live agent never writes memory directly. Instead, after each session goes idle for 6 or more hours, a small extraction model (gpt-5.4-mini) reads the entire rollout transcript and emits a structured raw_memory artifact. Then a heavier consolidation model (gpt-5.4) runs as a sandboxed sub agent inside the memory folder itself, with its own bash and Read / Write / Edit tools, and edits the canonical MEMORY.md handbook plus a skills/ tree. The folder has its own .git/ so the consolidation agent can diff its work against the previous baseline. The next session sees only memory_summary.md (capped at 5K tokens) injected into the prompt. The full handbook is loaded on demand by the agent issuing grep calls.

- **Claude Code** bets on user oversight. Memory is written inside the live turn, by the live agent, using the same Write and Edit tools the agent uses for any other file. The user is at the keyboard during the write, can see the file land, can object on the spot. There is no background extractor. There is no consolidation phase. The MEMORY.md index is always in the system prompt, every turn, and the bodies are read on demand via the standard Read tool when the agent judges them relevant.

The same architectural axes that mattered for Excel agents matter again here. Heavy upfront investment in tool design (Codex's structured Phase 1 / Phase 2 prompts) versus minimal scaffolding (Hermes's two flat files). Synchronous in turn writes (Claude Code, Hermes) versus deferred batch writes (Codex). Always loaded context (Claude Code, Hermes) versus on demand grep (Codex's full handbook). Each choice trades latency, cost, freshness, and consistency in different proportions.

---

## The Storage Layer

What does a memory actually look like on disk?

### Hermes: section sign delimiters in two flat files

Hermes uses two markdown files, both UTF 8 plaintext, both stored under `~/.hermes/memories/`. Entries are separated by a single delimiter constant:

```text
# tools/memory_tool.py:57
ENTRY_DELIMITER = "\n§\n"
```

Why §? Because U+00A7 almost never appears in user authored text, so it is safe to use as an in band record separator without escaping. The file looks like a flat list of paragraphs (probably optimized for grep?):

```markdown
User likes pour over coffee, hates espresso machines.
§
User is based in San Francisco, mostly works async.
§
When the user says "ship it", they mean push to main without further review.
```

No header. No JSON envelope. No metadata. An entry is just a string. Entries can be multiline. Splitting on the full delimiter (not just § alone) means an entry that happens to contain a section sign in its content is preserved correctly.

The two files split along a clean axis: MEMORY.md is "what the agent learned" (environment facts, project conventions, tool quirks), USER.md is "who the user is" (preferences, communication style, expectations). The header rendering reminds the model where it is writing:

```markdown
══════════════════════════════════════════════
USER PROFILE (who the user is) [73% — 1,612/2,200 chars]
══════════════════════════════════════════════
Based in San Francisco. Background in fintech.
§
Prefers replying to all recipients on every email thread.
```

That [73% — 1,612/2,200 chars] is rendered fresh on every read. The model sees its own budget pressure and is supposed to prune itself before the limit is hit.

### Codex: strict block schema with required frontmatter

Codex is the opposite extreme. Every memory has a strict structure imposed by the consolidation prompt. The canonical handbook lives at `~/.codex/memories/MEMORY.md` and is organized by `# Task Group:` headings. Each task block has subsections that must surface in a specific order:

![Codex block schema](https://pbs.twimg.com/media/HHQZqUcaEAAvWPP?format=jpg&name=small)

The Phase 1 extraction model is forced via JSON schema validation to emit raw memories with required frontmatter:

![Codex JSON schema](https://pbs.twimg.com/media/HHQZsjkbMAApUBA?format=jpg&name=small)

`additionalProperties: false` and `deny_unknown_fields` reject malformed output at parse time. The schema is so strict that the consolidation prompt is 841 lines, much of it teaching the model how to maintain the schema across updates.

The benefit: the handbook is machine readable enough that the consolidation agent can target specific subsections without rewriting unrelated content, and the read path can grep on stable field names like `applies_to:` to find the right block. The cost: prompt complexity. Keeping a model on schema across model upgrades is a constant prompt engineering tax.

### Claude Code: typed file taxonomy with YAML frontmatter

Claude Code goes a third direction. One file per memory, named by type prefix, all stored under a per project encoded path. My own machine looks like this:

![Claude Code file taxonomy](https://pbs.twimg.com/media/HHQZ2f-aIAA0bTh?format=jpg&name=small)

Every file has the same YAML frontmatter shape:

![Claude Code YAML frontmatter](https://pbs.twimg.com/media/HHQZ4NkaoAALBeo?format=jpg&name=small)

Four types observed across my 64 live files: user (biographical, rare writes), feedback (behavior corrections, dominant by count, more than half of all entries on my disk), project (codename and project mappings), reference (technical deep dives for repeated lookup).

The body convention varies by type. Feedback files follow a rigid `<rule statement> / **Why:** / **How to apply:**` shape. Project files do the same. Reference files are freeform with `##` headings. User files are short biographical notes. The discipline lives in the prompt, not the parser. There is no validator that rejects a file with `type: foo`. But the prompt convention has held: across 64 files written over months of sessions, all four types are observed cleanly.

The encoded path is its own quirk. `C:\Users\name` becomes `C--Users-name`. Drive separator dropped, every path separator becomes a dash, leading drive letter survives at the front. The encoding gives every working directory its own memory folder, which is how Claude Code does multi tenancy without any explicit project concept.

#### Storage layer comparison

![Storage layer comparison](https://pbs.twimg.com/media/HHQaBUQa0AAnRCQ?format=jpg&name=small)

Three axes: how strict is the schema, how many files, and where is the index. Hermes picks "one file, no schema, no separate index." Codex picks "many files, strict schema, separate index." Claude Code picks "one file per memory, loose schema, separate index." Each is internally consistent, and each fails differently when stressed.

---

## How Memory Loads Into the System Prompt

Every agent has to answer one question on every turn: how do I get the user's memories in front of the model?

The naive answer (re query a vector store on every turn, splice the results into the system prompt) breaks the prompt cache, which I will get to in the next section. So all three of these systems do something more interesting.

### Hermes: snapshot at session start, never refresh mid session

![Hermes snapshot](https://pbs.twimg.com/media/HHQaHCTaQAA58es?format=jpg&name=small)

Two important details. The snapshot is set exactly once in `load_from_disk()`. `format_for_system_prompt()` always returns the snapshot, never the live state. Mid session writes update the disk and update the live MemoryStore.entries list (so the tool response reflects the new content), but the bytes injected into the system prompt do not change.

### Codex: only the index, full handbook on demand

![Codex index](https://pbs.twimg.com/media/HHQaLu3aIAATmT9?format=jpg&name=small)

The injected `read_path.md` template makes the lazy load discipline explicit:

![Codex read_path.md](https://pbs.twimg.com/media/HHQaP00akAAWE9R?format=jpg&name=small)

The 5K token budget is the only ceiling on what gets injected into the developer prompt on every turn. Everything else (the full MEMORY.md, rollout summaries, skills) is loaded on demand by the agent issuing shell calls. Every read is classified into a MemoriesUsageKind enum (MemoryMd, MemorySummary, RawMemories, RolloutSummaries, Skills) and emits a codex.memories.usage counter, so the team can see at runtime which memory layers are actually being used.

### Claude Code: full index always loaded, bodies on demand

The MEMORY.md index is loaded into every turn under an `# auto memory` block. From a real session reminder I captured while writing this:

![Claude Code auto memory](https://pbs.twimg.com/media/HHQaUyzaEAEixB4?format=jpg&name=small)

The framing is striking. The reminder positions auto memory as higher priority than the base system prompt: "These instructions OVERRIDE any default behavior and you MUST follow them exactly as written." This is why feedback rules like feedback_no_hyphens.md reliably win over conflicting default behavior. The agent treats them as binding instructions, not soft hints.

The index is hard truncated at 200 lines. My index sits at 64 entries, well under the cap. A user with 500 memories would either need to prune or migrate to multiple working directories. I sometimes go read all the memories and delete some.

The bodies of individual files are NOT in the system prompt. When the agent decides "I see feedback_no_hyphens.md in the index, I should read it before drafting this email," it calls the standard Read tool with the absolute path. There is no specialized "memory_read" tool. Memory is just files, and the file tools are the same ones the agent uses for source code.

#### Where memory lands in the prompt order

![Prompt order](https://pbs.twimg.com/media/HHQaZOLbQAApRMB?format=jpg&name=small)

Order matters. Memory comes after policy and identity, before behavioral overrides and tool surfaces. In all three systems, memory is positioned as supporting context for the identity, not the identity itself. You do not want a single feedback rule to override the agent's core safety contract. You do want a feedback rule to override how the agent formats an email.

---

## The Prefix Cache Problem

This is the single most important constraint. KV Cache hit rate is crucial.

Every frontier API (Anthropic, OpenAI, Google) bills cached input tokens at a steep discount. Anthropic's prompt cache hits cost roughly one tenth of the uncached price. OpenAI's Responses API has automatic prefix caching with similar economics. The catch: cache hits require byte for byte prefix equality between turns. If the system prompt changes by even a single character at position N, every token after N is re billed at full rate.

A long Hermes session might have:

![Hermes session](https://pbs.twimg.com/media/HHQaeKDagAAoY8J?format=jpg&name=small)

22K tokens of system prompt. If you re query a vector store on every turn and re inject results into the system prompt, every turn pays full price for those 22K tokens. At ~$3 per million input tokens for the headline rate vs ~$0.30 for cached, that is a 10x cost multiplier on the entire prompt. Over a 50 turn session, you have just turned a $1 conversation into a $10 conversation, for no semantic gain.

This is why Hermes freezes the snapshot at session start. It is not an optimization; it is the load bearing design choice that makes long sessions economically viable.

![Hermes cache](https://pbs.twimg.com/media/HHQai3IakAEnMWS?format=jpg&name=small)

Hermes pays for this in freshness. A memory written on turn 5 is not visible to the model in the prompt for turns 6 through end of session. The model can see it briefly via the tool response on turn 5 (which echoes back the live entry list), but on turn 7 the system prompt still shows the snapshot from session start. The new entry only becomes prompt visible on the next session boot.

Codex sidesteps the issue differently. Memory is consolidated between sessions, not during them. The 5K token memory_summary.md is only written when Phase 2 finishes a consolidation run. Mid session, it does not change. The full MEMORY.md handbook is loaded on demand inside the user message, not in the system prompt, so per turn lookups do not invalidate the cache.

Claude Code is the most aggressive about prompt cache friendliness. Mid session, the auto memory block in the system prompt is byte stable. New memories written during a turn land on disk and update the index file, but the system prompt for the rest of the session keeps showing the index as it was at session start. The next session boot picks up the new entries by re reading the index from disk.

The pattern across all three: per turn dynamic data goes in the user message, not the system prompt. Hermes external providers inject recall context as a `<memory-context>` block in the user message:

![Hermes memory-context](https://pbs.twimg.com/media/HHQamJgbQAAcun7?format=jpg&name=small)

The system note is a defense against prompt injection from the recall channel. It tells the model the wrapped block is informational, not a new instruction. The `<memory-context>` tag wrapping is consistent across turns so the user message itself can still partially cache, but the inner content is allowed to change without breaking the system prompt cache.

If you take only one lesson from this section: **never inject dynamic memory into the system prompt!!!** Either freeze a snapshot at session start, or inject in the user message, or load on demand via a tool call. Mutating the system prompt mid session is what breaks the economics of long agent runs.

---

## The Two Phase Pipeline: Cron Jobs Meet Small Models

Codex picks the most architecturally interesting answer to "when do we write memory." The live agent never writes. Writes are deferred until after the session is idle for 6 or more hours, then handled by an asynchronous pipeline that runs as a background job at the start of the next session.

![Codex two phase pipeline](https://pbs.twimg.com/media/HHQa3r3aEAA4Atn?format=png&name=small)

The Phase 1 model is the small one: gpt-5.4-mini with low reasoning effort. The job is mechanical. Read a transcript, decide if anything happened that future agents should know about, emit a structured artifact. If nothing happened, emit empty strings (more on the signal gate below).

![Codex phase 1](https://pbs.twimg.com/media/HHQa5yDaoAApDHu?format=jpg&name=small)

Phase 2 uses the bigger model. The job is hard. Read the previous handbook, read the new evidence, decide what to add, what to update, what to supersede, what to forget, and write a coherent handbook back out. The git diff against the previous baseline tells the model what changed since last consolidation, so it can detect deletions (rollout summaries that are gone) and emit corresponding "forget this" moves on the handbook.

The consolidation agent is just an LLM with the same primitive tools the live agent has. Read, Write, Edit, bash. No special "consolidate memory" API. No proprietary diff format. The agent reads markdown, edits markdown, commits markdown to git. The complexity lives in the prompt (842 lines explaining the schema and the workflow), not in any custom infrastructure.

This is the cron jobs and small models pattern in its purest form. Live turn cost stays low because writes are deferred. Quality stays high because consolidation runs offline with a heavier model and a longer prompt. The system stays simple because both phases are just "spawn an agent with the right tools and the right prompt."

The cost is freshness. Memory written from today's session is not available until tomorrow's session, after the 6 hour idle window has passed and the cron job has fired on next boot. For users who hit the same problem in the same session, this is invisible. For users with rapidly evolving preferences (a new project, a new codename, a new rule), the lag matters. The `<oai-mem-citation>` pattern partially mitigates this: when the agent writes memory citations into its own response, the citation parser increments the usage_count immediately, even before the memory is consolidated.

#### Why this works only for cloud rollouts

Codex's pattern requires a few preconditions that are not always met. First, sessions have to be rollout shaped: a finite transcript that ends, with a clear idle window. Interactive Hermes and Claude Code sessions are open ended. The user keeps coming back. There is no clean boundary at which to fire Phase 1. Second, the pipeline assumes you have a state database for lease semantics and watermarking. SQLite works fine for a single user CLI; for a multi tenant cloud product, this is more involved. Third, the small model has to be actually small and fast. gpt-5.4-mini at low reasoning effort is cheap enough to run on every rollout boot. If you are budget constrained, you cannot afford to extract memory from every session.

For a synchronous interactive agent like Claude Code, the right pattern is probably the synchronous live writes Claude Code already uses. It's also the simplest. For a deferred batch agent like Codex (or any coding agent that runs on cloud workers), the two phase pipeline pays for itself.

---

## The Signal Gate

The most underrated part of Codex's design.

Every memory system has the same failure mode: noise. The model writes too many memories, none of them load bearing, and the index becomes a Wikipedia article on the user's behavior with no signal to extract. Once the noise to signal ratio crosses some threshold, the agent stops trusting memory, and the whole feature is dead.

Hermes solves this with a hard char cap. Once you hit 2200 chars on MEMORY.md, you cannot add anything new without removing something old, so the model is forced to triage. The cap doubles as a quality gate: if the new memory is not worth more than what is already there, do not write it.

Claude Code solves this with prompt discipline. The `<types>` block tells the agent what NOT to save:

> Do not save trivial corrections that apply to one task only. Do not save facts already obvious from the codebase or CLAUDE.md. Do not save user statements that are likely to flip in the next session. Do not duplicate; grep first and update existing memories rather than create new ones.

It works most of the time but is fragile against paraphrase. Two of my own files (feedback_reply_all.md and feedback_never_use_reply.md) are about closely related topics and could plausibly have been one file. The agent had to decide on each write whether the new rule was an extension of the existing one or a fresh rule. Sometimes it splits when it should have merged. The cluster of feedback_no_* files (no_hyphens, no_calls, no_mcp, no_color_default, no_recommendations_pptx, no_speculative_numbers) is healthy fan out, but the line between fan out and duplication is blurry.

Codex solves it with an explicit gate. The Phase 1 system prompt opens with this:

![Codex signal gate](https://pbs.twimg.com/media/HHQbCQUbwAAQlyA?format=jpg&name=small)

And it is enforced at runtime. The Phase 1 worker checks the output:

![Codex phase 1 worker](https://pbs.twimg.com/media/HHQbEBcaIAAI4FL?format=jpg&name=small)

A no op rollout is recorded as succeeded_no_output in the state DB, distinct from a hard failure. It clears the watermark and won't be retried. The session is marked as "we looked at it and decided nothing was worth saving."

The prompt also tells the model what high signal looks like:

> Stable user operating preferences High leverage procedural knowledge Reliable task maps and decision triggers Durable evidence about the user's environment and workflow Core principle: optimize for future user time saved, not just future agent time saved.

This is the hardest part of memory design. It is not a data structure problem. It is a judgment problem. What is worth remembering? Codex pays the cost upfront in the prompt: 570 lines of stage one extraction prompt, much of it teaching the small model the difference between a load bearing memory and a noise memory. The cost is real. Maintaining a 570 line prompt across model upgrades is a constant prompt engineering tax. The benefit is that the model exits a session with empty hands much more often than it should, by default, and noise memories never make it into the handbook in the first place.

For any agent serving a power user, this is the most transferable pattern from Codex. Default to no op. Make the model justify writing. Reward the empty output.

---

## Memory Limits and Eviction

Once memory exists, you have to decide what to throw away.

### Hermes: hard char cap, manual eviction

![Hermes char cap](https://pbs.twimg.com/media/HHQbN3jbQAAOHsl?format=jpg&name=small)

No automated decay. No LRU. No TTL. Entries persist forever until explicitly removed. The forcing function is the char limit error. The model is expected to consolidate.

This is a strong choice. The user can cat `~/.hermes/memories/MEMORY.md` and read the entire contents in 30 seconds. Nothing is hidden. The cost is precision: a memory that mattered once and never again sits in the file forever, taking up budget. The benefit is auditability: you always know exactly what the agent thinks it knows.

### Codex: usage decay with grace period

Codex tracks usage explicitly. Every memory has two columns in the SQLite state DB:

```sql
ALTER TABLE stage1_outputs ADD COLUMN usage_count INTEGER;
ALTER TABLE stage1_outputs ADD COLUMN last_usage INTEGER;
```

When the live agent emits an `<oai-mem-citation>` block citing a specific rollout (memory was actually used to generate the response), a parser fires and bumps the count:

```sql
UPDATE stage1_outputs
SET
    usage_count = COALESCE(usage_count, 0) + 1,
    last_usage = ?
WHERE thread_id = ?
```

Phase 2 selection ranks memories by usage, and the cutoff is now - max_unused_days (default 30):

```sql
WHERE t.memory_mode = 'enabled'
  AND (length(trim(so.raw_memory)) > 0 OR length(trim(so.rollout_summary)) > 0)
  AND (
        (so.last_usage IS NOT NULL AND so.last_usage >= ?)
        OR (so.last_usage IS NULL AND so.source_updated_at >= ?)
  )
ORDER BY
    COALESCE(so.usage_count, 0) DESC,
    COALESCE(so.last_usage, so.source_updated_at) DESC,
    so.source_updated_at DESC,
    so.thread_id DESC
LIMIT ?
```

A used memory falls out of selection only after 30 days of no further citation. A never used memory falls out 30 days after creation. So fresh memories get a 30 day "trial" window. Hard deletion happens later, in batches of 200, only for rows not in the latest consolidated baseline (selected_for_phase2 = 0).

The risk: usage_count increments only on explicit `<oai-mem-citation>` emission. If the agent uses memory but forgets to cite, the signal is lost. The decay loop depends on prompt compliance. In practice this seems to mostly work, but it is the kind of thing that breaks silently if the model upgrades and citation behavior shifts.

### Claude Code: no decay, only verification

This is the cleanest contrast. Claude Code has no usage_count, no last_usage, no max_unused_days knob. A memory file written on day 1 will still be in MEMORY.md on day 365 unless the agent or user manually deletes it.

What Claude Code does instead is verification. Every individual memory file is wrapped in a `<system-reminder>` when read by the agent, with text like:

> This memory is N days old. Memories are point in time observations, not live state. Claims about code behavior or file:line citations may be outdated. Verify against current code before asserting as fact.

The age in days is rendered dynamically on every read. This is the load bearing piece. The model is told this every time it touches a memory body, not just at session start. Stale memories do not get auto trimmed; they get ignored when verification fails.

![Claude Code verification](https://pbs.twimg.com/media/HHQbrqmaUAA6CJI?format=jpg&name=small)

The cost is wasted tokens on every read (the warning text plus the verification grep). The benefit is that the agent never silently asserts a stale fact. Even Codex, with all its consolidation machinery, does not have an equivalent of the per memory dynamic age reminder.

![Claude Code verification 2](https://pbs.twimg.com/media/HHQbtM5a0AAic66?format=jpg&name=small)

Three completely different forcing functions. Char cap pressures the model to consolidate. Usage decay rewards memories that actually get cited. Verification reminders make staleness visible at use time rather than storage time. Each works for its own architecture.

---

## The Verification Discipline

This is the part of Claude Code's design that is most worth porting to other agents.

A memory is a claim about something at a moment in time. The user said X. The codebase has function Y on line 42. The team's preferred Slack channel is Z. By the time you read the memory back, any of these claims could be stale. The user changed their mind. The codebase refactored. The team migrated to Discord.

Most memory systems do not address this directly. Hermes will happily inject a 6 month old memory into the system prompt as if it is current. Codex will rank an old memory below a new one but still ship it to the agent if it has high usage_count. Both treat memory as authoritative once written.

Claude Code treats memory as a hint surface. Two things make this work.

First, the always loaded index (MEMORY.md) carries only the description, not the body. So at the system prompt level, the agent sees:

```markdown
- [feedback_no_hyphens.md](feedback_no_hyphens.md) — Never use hyphens in any
  written content
- [reference_codebase_architecture.md](reference_codebase_architecture.md)
  — Codebase architecture: model routing, prompt structure, skills system,
  caching, tool surface, key settings
```

That is enough information for the agent to decide "is this memory relevant to the current request." It is not enough information to act on. Acting requires reading the body.

Second, every body read is wrapped in the age reminder. Every. Single. Read. The reminder text:

> Records can become stale over time. Use memory as context for what was true at a given point in time. Before answering the user or building assumptions based solely on information in memory records, verify that the memory is still correct and up to date by reading the current state of the files or resources.

And critically:

> A memory that names a specific function, file, or flag is a claim that it existed when the memory was written. It may have been renamed, removed, or never merged. Before recommending it: if the memory names a file path, check the file exists. If the memory names a function or flag, grep for it. If the user is about to act on your recommendation, verify first.

The composite design philosophy: memory is a hint surface, not an authority surface. The system makes it easy to write hints, easy to read hints, and impossible to read a hint without being told to verify. That is the contract Claude Code is offering, and it is the contract every memory system should match as a baseline before adding any heavier infrastructure.

#### Where this matters for code agents

Half my memory file body reads are about codebases that are evolving. References to file paths, function names, configuration flags. If the agent recommended these from memory without verification, it would silently regress toward old behavior every time the codebase moved. With verification, it catches itself: "the memory says altic_skill_loader.py defines load_skill, but grep returns no results, so this memory is stale, let me update it." The cost is one extra tool call per memory read. The benefit is correctness on a moving target.

For any agent designer, the lesson is: wrap every memory body read in a dynamic freshness reminder. Write the age in days into the reminder. Tell the agent to verify before asserting. This costs nothing at storage time and pays compound interest at retrieval time, especially as the codebase or workspace evolves under the agent's feet.

---

## Day 1 Bootstrap: The Cold Start Problem

This is the hardest part, and nobody has solved it.

Imagine a new user opens an agent for the first time. The memory directory is empty. The agent has no idea who this person is, what they care about, what their codebase conventions are, what their team looks like, what their prior preferences are. The first 10 sessions feel useless because the agent is still learning. By session 50 it knows them well. By session 200 it is irreplaceable. But the first 10 sessions are the ones that decide whether the user keeps using the product.

Codex does not address this at all. The bootstrap is mechanical: a fresh user starts with an empty `~/.codex/memories/` folder, and the first Phase 2 run (after the first eligible session) builds the artifacts from scratch. There is no synthetic priming from external sources. The user profile is built up over time from rollout signals only. From the consolidation prompt:

> Phase 2 has two operating styles: INIT phase: first time build of Phase 2 artifacts. INCREMENTAL UPDATE: integrate new memory into existing artifacts.

The INIT phase still requires real prior sessions to extract from.

Hermes does not address it either. New profile, empty MEMORY.md, empty USER.md. The user has to manually seed or the agent has to learn from scratch.

Claude Code is the most interesting because it punts: instead of bootstrapping the auto memory system, it relies on CLAUDE.md to carry the static "who am I" context that should not change across sessions. My own CLAUDE.md is around 200 lines describing my role, my key contacts, my repos, my email, my output format defaults. This is the seed. The auto memory system layers on top with feedback rules and project facts learned over time.

The Day 1 problem for any new agent product is: how do you bootstrap from external sources the user has already invested in? Cloud drive files. Email contacts. Calendar history. Chat threads. Code repos. The user's existing digital footprint contains thousands of "facts about the user" already. A good Day 1 bootstrap would seed the memory with reference and project files from these sources, so the agent walks into session 1 already knowing the user's role, key working relationships, and core preferences.

None of the three open systems do this today. It is the open problem in agent memory design. The right answer probably looks like:

![Day 1 bootstrap](https://pbs.twimg.com/media/HHQcBRfaEAAkt5x?format=jpg&name=small)

This is the next obvious step in agent memory and the area I am most excited about. The user's data is sitting right there. Bootstrapping from it is just a matter of building the right one shot extractor and trusting the user to approve the output.

---

## Cross Project Scoping

How does memory work when you have many projects?

- **Hermes** has profiles. Each profile is a separate `~/.hermes/profiles/<name>/` directory with its own memories/ subdirectory. There is no cross profile sharing. The coder profile and the default profile have completely separate MEMORY.md files. This works well for users who want clean separation (work vs personal, say) but does not handle the "I have a global rule that applies across all profiles" case. There is no `~/.hermes/profiles/_global/memories/` overlay.

- **Codex** picks the opposite extreme. There is one global folder at `~/.codex/memories/` regardless of what project you are working in. Per project signal is preserved inside the content. Every block in MEMORY.md carries an `applies_to: cwd=<path>` line, and every raw memory has a `cwd:` frontmatter field. So a single handbook holds memories for every project the user has ever worked in, separated by cwd annotations. The read path is supposed to filter by cwd; the consolidation prompt is supposed to write blocks scoped by cwd. In practice, cross project leakage is possible: a feedback rule about formatting in project A could plausibly get applied in project B if the agent does not check the applies_to: line carefully.

- **Claude Code** goes t