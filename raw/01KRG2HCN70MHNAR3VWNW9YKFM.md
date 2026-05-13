# Agent Platform That Builds Itself

[![Ashpreet Bedi](https://pbs.twimg.com/profile_images/2024047696827273217/vlW-RvPT_normal.jpg)](https://x.com/ashpreetbedi)

By [Ashpreet Bedi](https://x.com/ashpreetbedi)

[![Image](https://pbs.twimg.com/media/HIIGmzyaUAA-tZE?format=png&name=small)](https://x.com/ashpreetbedi/article/2054222428025614829/media/2054211643840811008)

Every team building agents ends up building an agent platform from scratch.

Coding agents can do all of that work now. Today I'll share how to build an agent platform built, managed and improved entirely by coding agents.

The entire agent development lifecycle is managed using five prompts:

- **Create.** Scaffolds a new agent.
- **Improve.** Hardens an existing agent against its own spec.
- **Extend.** Adds new capabilities to an existing agent.
- **Hill Climb.** Runs the eval suite, diagnoses failures, fixes what's in scope.
- **Review.** Sweeps the repo for drift between docs, code, and config.

---

## What is an agent platform

Let's say you built an agent in your favorite framework. How do you take it live? How do you host it in the cloud, send requests to it, secure it?

Your agent platform is the service responsible for running your agents. It takes requests, runs the agent and streams the responses. It collects data and metrics, manages security by preventing unauthorized access, and stops one agent from accessing or polluting the data of another.

If you think of agents as mini-applications, it becomes clear that they need a system to run on, like an OS. Your agent platform is that OS.

---

## What we're building

Today we'll build an agent platform that you can run locally using docker, your own cloud, or on Railway with 1 command. The platform has five parts:

1. **Runtime:** the service that runs your agents. It handles requests, runs the agent loop, streams responses, writes to storage, handles auth.
2. **Storage:** the database where our data lives: agent sessions, memory, knowledge, traces and eval history.
3. **Connectors:** tools for agents to connect with external systems via MCP, API, or CLI. Having them in one place is a big win for security.
4. **Interfaces:** Slack, Discord, Telegram, custom UIs. One place to resolve identity across surfaces, so the same person is the same user_id whether they ping you in Slack or hit the web app.
5. **Infrastructure:** where everything runs. We'll use Docker for local and Railway for production. You're free to run production anywhere.

Once it's running, you should be able to ship a new agent in ~10 minutes without writing any code. I know this is a crazy claim so let's give it a shot.

---

## Let's get started

I'm going to share a foundational codebase that you can build upon.

Clone, configure, and run:

[agent-platform-railway](https://github.com/agno-agi/agent-platform-railway)

This brings up two containers: a FastAPI server and a Postgres database.

Now let's give our platform a UI.

1. Head to [os.agno.com](https://os.agno.com/) and sign in.
2. Connect to your local OS at http://localhost:8000.

You should see something like:

[![UI Screenshot](https://pbs.twimg.com/media/HIGPx1oa4AI9I7D?format=jpg&name=small)](https://x.com/ashpreetbedi/article/2054222428025614829/media/2054080991430631426)

---

## Agent Development Lifecycle

Because everything is in one place, Claude Code can manage my entire agent development lifecycle.

### 1. Create an agent

To create a new agent, I open Claude Code and type:

> Run create-new-agent.md in a new branch.

Claude starts by asking a few questions about what the agent should do, which tools it needs. It then searches the Agno docs via MCP for the right toolkit, generates the agent file, registers it in app/main.py, restarts the container, and smoke-tests via cURL. 5-10 minutes from prompt to agent.

### 2. Improve an agent

To improve an existing agent, I type:

> Run improve-agent.md on code-search agent.

Claude reads the agent's INSTRUCTIONS and derives 8-12 probes from them. Some golden-path. Some edge cases. Some tool-selection. A couple of adversarial ones thrown in: prompt injections, malformed input, attempts to pull the agent off-purpose.

It runs each probe against the live container via cURL. Reads the response. Reads the tool calls from the container logs. Judges PASS or FAIL against what the INSTRUCTIONS actually promise.

For every failure it picks a lever. Tighten a rule. Add a rule. Swap a tool. Bump num_history_runs. Whatever fits the failure mode. It edits agents/<slug>.py, hot-reloads, and re-runs only the probes that failed.

Zero input from me beyond kicking off the task. This used to take a day of manually clicking things around and now it's fully automated.

### 3. Extend an agent

To add capabilities to an existing agent, I type:

> Run extend-agent.md on code-search agent.

Extend runs with my guidance. I describe a change: add a tool, refine a prompt, fix a bug. Claude executes. The Agno docs MCP is loaded so toolkit research is grounded in the real API.

### 4. Hill Climb

Over time we collect a lot of evals, and it would be a shame to fix failures manually. I simply type:

> Run eval-and-improve.md.

Hill Climb runs the eval suite, diagnoses every failure, and fixes what's in scope.

### 5. Review

Because the repo is managed primarily by coding agents, it moves fast. To bring everything up to speed, I type:

> Run review-and-improve.md.

Claude sweeps the whole repo for drift between docs, code, and config. Fixes what it can. Best run before a release or after a refactor.

Drift between docs and code has always been a tax on production software. Now it costs nothing.

---

## Run in production

When you're ready to ship, the codebase comes with deploy-to-Railway scripts:

This provisions Postgres and the app service on the same private network.

Read the full Railway guide in the [README](https://github.com/agno-agi/agent-platform-railway).

---

## Wrapping up

Congratulations. If you made it this far, you have an auto-improving agent platform running securely in your cloud.

Technical users on your team can create and deploy agents using Claude Code. Non-technical users can use the no-code UI.

Sessions, traces, and knowledge live in your database. Your infrastructure is gated behind JWT-based RBAC and API keys are managed in one place.

The agents you ship today are the smallest part of what you've built. The platform underneath them, and the iteration loop it enables, is the thing that matters.

Thanks for reading. Built with 🧡 using Agno.
