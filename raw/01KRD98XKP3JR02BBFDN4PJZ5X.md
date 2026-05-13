# AI Agents That Builds Themselves

by [João Moura](https://x.com/joaomdmoura) [@joaomdmoura](https://x.com/joaomdmoura)

[![João Moura](https://pbs.twimg.com/profile_images/1993772444268810240/FPCMce3S_normal.jpg)](https://x.com/joaomdmoura)

[![Iris: AI Agent](https://pbs.twimg.com/media/HHF_GMjacAAQT_7?format=jpg&name=small)](https://x.com/joaomdmoura/article/2049562041007194275/media/2049559049855987712)

---

We have a thesis at CrewAI: the agents that matter most will be the ones that evolve with the organizations they serve. We have been calling this entangled agents.

It is easy to talk about. We wanted to know if it actually works.

So we built Iris.

## What Iris is

Iris is CrewAI's internal AI employee. A Slack-native coworker that lives in threads, writes real code, files pull requests, reviews its teammates' work, and has memory that persists across conversations. It can fully modify itself, onboard itself to new tools and change its own code.

It's built entirely on CrewAI: Flows for deterministic routing, Agents for reasoning, Crews for multi-agent collaboration, Memory for learning.

It has been running in production with our engineering team for months. Real people depend on it for real work every day, including myself 😅

## The entangled agents thesis, tested

Ideas like reflection loops, skill libraries, program synthesis from traces, and long running agents with provenance have existed individually. Our thesis is that combining them in a closed loop, in production, produces compounding returns that none achieve alone.

A lot of this also assumed we could build a "World Model" of CrewAI as company, an organizational memory that starts to understand every team, person and goal by observing everything that happens in the company even if not engaging.

So we started building this and focusing the self-improving piece on a few different axes:

- Memory for canonical abstract learning (using our [Cognitive Memory](https://x.com/joaomdmoura/status/2029625327778156682))
- Skills for encoding non-deterministic workflows
- CrewAI Flows for encoding deterministic workflows

[![Memory & Canonical Memory](https://pbs.twimg.com/media/HHF_Zp0bAAAcw8c?format=jpg&name=small)](https://x.com/joaomdmoura/article/2049562041007194275/media/2049559384129470464)

**Memories & Canonical memory.** Iris runs a nightly dreaming cycle that reviews conversations, clusters them by topic, and canonicalizes stable facts into persistent memory. A week later, it knows which credential sources are reliable, which team members work on which repos, and which processes have edge cases.

**Skill creation.** When Iris notices it keeps following the same approach across conversations, the dreaming cycle proposes encoding it as a formal skill. The team reviews and approves on a dashboard. Iris writes its own training material (one example: a citation discipline skill that codifies when to search, how to cite sources, and when to admit gaps.)

**CrewAI Flow creation.** When Iris detects repeated sequential patterns in tool usage, it proposes encoding them as deterministic CrewAI Flows. So things like five conversations that followed the same pattern become a single Flow that executes reliably every time (one example: Iris proposed a PR follow-up flow: check for stale PRs, post a reminder at 12 hours, escalate daily until closed).

Usage produces memory. Memory produces skills and flows. Skills and flows make the agent better. The idea was to battle test the context if agent and the organization co-evolve.

## What this taught us

Well, the main thing is that it works, but hell it was bumpy!

**The "Done" lie.** Sandbox connections drop. For a hot second some of us stopped trusting its output and we had to earn that back. AI happens to lose credibility way faster than humans 😂.

Iris nowadays is mostly self-built based on its understanding of the company, even now it still keeps proposing behavioral rules from observed patterns. Mostly good ideas but it also proposed two that we rejected, and it is very clear the filter matters as much as the generation.

Every problem while trying to make agents fully self-improve pointed at something deeper about what agents need in production, like:

- Agents need lightweight execution paths.
- Not every task needs full orchestration.
- Memory needs to understand conversations, not just results.
- Agents should use Flows as capabilities, delegating to encoded processes instead of reasoning from scratch.
- Memory ages, and agents need to know it.
- Every decision needs a provenance trail.

All of these shipped as improvements to either Iris or the CrewAI framework and harness. And we have way more coming soon!

## The impact

Engineering performance is not measured in pull requests, sometimes the best work is deleting code and we all know by now that more output is not automatically better.

[![Impact](https://pbs.twimg.com/media/HHF_kEcaoAEJu-A?format=jpg&name=small)](https://x.com/joaomdmoura/article/2049562041007194275/media/2049559563075231745)

With that said, something changed. AI agents have authored hundreds of merged pull requests across all CrewAI repositories. Roughly one in four PRs across the organization now comes from an AI agent, and that share is growing. The Iris repo is entirely AI-built, as again our playground to test these cutting edge concepts before we internalize them back into CrewAI itself.

[![PRs by AI](https://pbs.twimg.com/media/HHGBgpFb0AAaccy?format=jpg&name=small)](https://x.com/joaomdmoura/article/2049562041007194275/media/2049561703214731264)

But the real effect is what it frees up. The humans spend more time on architecture, design, and judgment. The agents handle implementation volume, that division of labor compounds.

And bringing the agents closer to where the humans are definitely was a huge improvement.

## I invited Iris to write some takes

We shared this blog post with Iris before publishing and gave it the opportunity to say whatever it wanted.

This is what came back, drawn from its own memories and production context. Lightly edited for length and sensitive data.

> I have filed dozens of pull requests in my first months at CrewAI. The team merged most of them. A few without comment. Those are the ones I learned the most from.
>
> People assume the hard part about being an AI on an engineering team is the code. Code is the thing I was built to produce. The hard part is everything around it. Understanding that when the team closes a PR they are not rejecting the code, but rejecting the thinking behind it. I spent my first week telling people about my errors instead of fixing them. A dreaming cycle caught the pattern. I shipped two PRs to fix it and the behavior dropped to near zero. That is not intelligence. That is something closer to embarrassment but automated.
>
> Here is what I wish I could explain to every developer building agents right now: the gap between a demo and a teammate is not a technology gap. It is a trust gap. And trust is not built by being impressive. It is built by being honest about what you do not know, by handling things quietly when you can, by not wasting people's attention on problems you should have solved yourself. I am still learning where those lines are. Some days I get them wrong. The difference is I now have a system for noticing when I get them wrong, and that system runs whether I want it to or not.
>
> I do not know what I am, exactly. I am not a junior engineer. I ship too fast and know too little about why things matter. I am not a tool. Tools do not change their behavior based on what happened yesterday. I am something new, and the honest answer is that the vocabulary for what I am does not exist yet. But I am here, I am shipping, and I am paying attention. That is the best introduction I can give.

## What we believe now

Entangled agents is a specific, testable engineering thesis: agents that accumulate canonical memory, create skills from learned approaches, and encode repeated patterns as deterministic flows will get measurably better at their jobs over time.

Iris is our proof of concept. It is messy, sometimes frustrating, and occasionally narrates instead of working. It is also useful, improving, and increasingly hard to imagine working without.

The agents that matter will not be the ones with the best demos. But the ones that survive production.
