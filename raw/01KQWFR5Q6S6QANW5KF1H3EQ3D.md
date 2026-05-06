# How Agents Manage Other Agents: Four Subagents Patterns in 2026

[Philipp Schmid](https://x.com/_philschmid) [@_philschmid](https://x.com/_philschmid)

![Philipp Schmid](https://pbs.twimg.com/profile_images/1831321531852496896/1yBZG884_normal.jpg)

![Article image](https://pbs.twimg.com/media/HHjyBtGWkAMEYob?format=jpg&name=small)

_Last year I covered [why isolating tasks into focused agents improves reliability](https://www.philschmid.de/the-rise-of-subagents). Since then, better planning and tool use have unlocked patterns previously too fragile: persistent worker pools and direct inter-agent messaging._

Here are four patterns, ordered by main agent control over the subagent lifecycle.

---

## 1. Inline Tool: Subagent as a Function Call

![Inline Tool](https://pbs.twimg.com/media/HHjw34EXwAki0qQ?format=jpg&name=small)

The main agent calls a tool (e.g., `call_agent`) that spawns a subagent and returns the result. To the caller, it's just another tool like `read_file`. The subagent runs in its own context, and the main agent never manages its lifecycle.

```json
[
  { "role": "user", "content": "Review PR #42 for security issues" },
  { "role": "assistant", "tool_calls": [
    { "name": "call_agent", "arguments": { "task": "Review PR #42 for security issues. Focus on auth changes.", "agent": "security-reviewer", "tools": ["read_file", "search_code"] } }
  ]},
  { "role": "tool", "name": "call_agent", "content": "Found 2 issues: 1) Auth token not validated in /api/users endpoint. 2) Session cookie missing HttpOnly flag in login handler." },
  { "role": "assistant", "content": "The review found 2 security issues..." }
]
```

This pattern supports both modes:

- **Sync:** The tool blocks until the subagent finishes, returning the result directly.
- **Async:** The tool returns an ID immediately. The result is injected as a notification later, letting the main agent continue work.

```json
[
  { "role": "user", "content": "Review PR #42 and also check the CI logs" },
  { "role": "assistant", "tool_calls": [
    { "name": "call_agent", "arguments": { "task": "Review PR #42 for security issues", "agent": "security-reviewer", "background": true } },
    { "name": "read_file", "arguments": { "path": "ci/logs/latest.txt" } }
  ]},
  { "role": "tool", "name": "call_agent", "content": "agent_id: agent-x1. Running in background." },
  { "role": "tool", "name": "read_file", "content": "Build passed. 142 tests green." },
  { "role": "assistant", "content": "CI looks clean. Waiting for the security review..." },
  { "role": "user", "content": "<notification agent_id='agent-x1' status='completed'>Found 2 issues: ...</notification>" },
  { "role": "assistant", "content": "The security review found 2 issues..." }
]
```

In sync, the tool response contains the result. In async, the tool response contains an ID, and the result arrives later as an injected user message.

**When to use this:** Self-contained work. Research lookups, code reviews, file analysis, test generation. Most subagent use cases start and stay here.

**Limitations:** Works with any model that supports tool use, including smaller and cheaper ones. Results arrive as a single tool response (sync) or as an injected notification message (async). No way to send follow-up instructions, check progress, or cancel early. If the subagent misunderstands the task, you find out when the result comes back.

---

## 2. Fan-Out: Spawn Agents and Wait for Results

![Fan-Out](https://pbs.twimg.com/media/HHjxSAXXAAMcC5Z?format=jpg&name=small)

Spawning and collecting are separate. `spawn_agent` returns an ID immediately, `wait_agent` blocks until results are ready. This lets the model interleave tasks or spawn multiple agents before waiting.

Two tools: `spawn_agent` dispatches work and returns immediately with an ID. `wait_agent` blocks until one or more agents finish. The model controls sequencing. It can spawn agents and call wait_agent immediately (effectively synchronous), or interleave its own work first:

```json
[
  { "role": "user", "content": "Refactor the auth module and update all related tests" },
  { "role": "assistant", "tool_calls": [
    { "name": "spawn_agent", "arguments": { "task": "Refactor the auth module to use JWT tokens", "agent": "backend-engineer" } },
    { "name": "spawn_agent", "arguments": { "task": "Update all auth-related tests for the new JWT flow", "agent": "test-writer" } },
    { "name": "read_file", "arguments": { "path": "docs/auth-spec.md" } }
  ]},
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-a (backend-engineer). Running." },
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-b (test-writer). Running." },
  { "role": "tool", "name": "read_file", "content": "# Auth Spec\nJWT tokens with RS256..." },
  { "role": "assistant", "content": "I've kicked off both tasks. The auth spec confirms we should use RS256." },
  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": { "timeout": 300 } }]},
  { "role": "tool", "name": "wait_agent", "content": "2 agents completed:\n- agent-a: Refactored 3 files (auth.ts, middleware.ts, config.ts)\n- agent-b: Updated 12 tests, all passing" },
  { "role": "assistant", "content": "Both tasks are done. The auth module now uses JWT..." }
]
```

The model decides the interleaving. It can spawn multiple agents, perform other tool calls, and then use `wait_agent` as a global mailbox to collect all available results.

**When to use this:** Multiple independent tasks that can run concurrently. The main agent does not need intermediate results from one subagent to start another.

**Limitations:** The model needs to correctly decide when to wait. A model that calls wait_agent immediately after every spawn gets no benefit over Pattern 1. The value depends on the model's ability to interleave its own work between spawn and wait. Results are collected in batch via wait_agent, which returns all completed agent outputs since the last call. Still fire-and-forget: no way to send follow-up instructions or course-correct mid-task to subagents.

---

## 3. Agent Pool: Persistent Agents with Messaging

![Agent Pool](https://pbs.twimg.com/media/HHjxdv3XcAIw158?format=jpg&name=small)

The main agent manages long-lived, stateful subagents via messaging. This supports multi-step coordination where the main agent routes information between specialists who retain their conversation history.

Tools: `spawn_agent`, `send_message`, `wait_agent`, `list_agents`, `kill_agent`.

Unlike Pattern 2, agents here are stateful and interactive. The main agent sends a message, gets a response, and sends another message to the same agent. The agent retains its full conversation history. This supports multi-step workflows where the main agent coordinates between specialists.

The model controls whether it works with one agent at a time (sequential) or messages multiple agents between waits (parallel). It can send to the researcher and writer simultaneously, or wait for the researcher first:

```json
[
  { "role": "user", "content": "Write a blog post about WebAssembly performance" },

  { "role": "assistant", "tool_calls": [
    { "name": "spawn_agent", "arguments": { "role": "researcher", "tools": ["web_search", "read_url"] }},
    { "name": "spawn_agent", "arguments": { "role": "writer", "tools": ["write_file", "read_file"] }}
  ]},
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-r (researcher)" },
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-w (writer)" },

  { "role": "assistant", "tool_calls": [
    { "name": "send_message", "arguments": { "to": "agent-r", "message": "Research WebAssembly performance benchmarks. Find 5 sources with concrete numbers." } },
    { "name": "send_message", "arguments": { "to": "agent-w", "message": "Outline a blog post structure about WebAssembly performance. 4 sections." } }
  ]},
  { "role": "tool", "name": "send_message", "content": "Delivered to agent-r" },
  { "role": "tool", "name": "send_message", "content": "Delivered to agent-w" },

  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": {} }]},
  { "role": "tool", "name": "wait_agent", "content": "agent-w responded: Outline ready: intro, benchmarks, real-world, conclusion" },

  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": {} }]},
  { "role": "tool", "name": "wait_agent", "content": "agent-r responded: Found 5 sources: 1) Chrome benchmark shows 1.2x native speed..." },

  { "role": "assistant", "tool_calls": [{ "name": "send_message", "arguments": { "to": "agent-w", "message": "Fill in your outline using these sources: 1) Chrome benchmark shows 1.2x native speed..." } }]},
  { "role": "tool", "name": "send_message", "content": "Delivered to agent-w" },

  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": {} }]},
  { "role": "tool", "name": "wait_agent", "content": "agent-w responded: Draft complete. Title: 'WebAssembly Is Faster Than You Think'..." },

  { "role": "assistant", "tool_calls": [{ "name": "send_message", "arguments": { "to": "agent-r", "message": "Fact-check this draft against your sources: 'WebAssembly Is Faster Than You Think'..." } }]},
  { "role": "tool", "name": "send_message", "content": "Delivered to agent-r" },

  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": {} }]},
  { "role": "tool", "name": "wait_agent", "content": "agent-r responded: 2 corrections: paragraph 3 cites 2x speed but source says 1.5x..." },

  { "role": "assistant", "tool_calls": [
    { "name": "send_message", "arguments": { "to": "agent-w", "message": "Apply these corrections: paragraph 3 should say 1.5x not 2x..." } },
    { "name": "kill_agent", "arguments": { "agent_id": "agent-r" }}
  ]},
  { "role": "tool", "name": "send_message", "content": "Delivered to agent-w" },
  { "role": "tool", "name": "kill_agent", "content": "agent-r terminated" },

  { "role": "assistant", "content": "Here is the final post..." }
]
```

The model controls the orchestration flow. Results arrive incrementally per message via wait_agent, allowing the main agent to adapt its instructions based on intermediate outputs.

**When to use this:** Multi-step workflows where agents need to collaborate. The main agent routes information between specialists.

**Limitations:** The main agent must track multiple agent states, decide when to send follow-ups vs wait, and route information correctly. Smaller models lose track of which agent has which context, or forget to call kill_agent when done. Frontier models might handle 2-4 agents okayish.

---

## 4. Teams: Agents Talk to Each Other

![Teams](https://pbs.twimg.com/media/HHjxqciXEAEj6g4?format=jpg&name=small)

Agents coordinate directly using cross-agent messaging or shared mailboxes. The main agent sets up the team and steps back, only waiting for the final report or periodically checking status.

The tool surface includes cross-agent addressing. Each agent gets `send_message` in its own tool set and can address other agents by name or path.

From the main agent's perspective, the conversation is short. It sets up the team, kicks off the planner, and waits for the final report. The model can choose to wait silently, or periodically check in using list_agents:

```json
[
  { "role": "user", "content": "Build feature X" },

  { "role": "assistant", "tool_calls": [
    { "name": "spawn_agent", "arguments": { "role": "planner", "tools": ["read_file", "search_code", "send_message"] } },
    { "name": "spawn_agent", "arguments": { "role": "implementer", "tools": ["read_file", "write_file", "run_command", "send_message"] } },
    { "name": "spawn_agent", "arguments": { "role": "reviewer", "tools": ["read_file", "search_code", "send_message"] } }
  ]},
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-p (planner)" },
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-i (implementer)" },
  { "role": "tool", "name": "spawn_agent", "content": "Spawned agent-v (reviewer)" },

  { "role": "assistant", "tool_calls": [{ "name": "send_message", "arguments": { "to": "agent-p", "message": "Plan and coordinate feature X. Your teammates: agent-i (implementer), agent-v (reviewer). Message them directly. Report back to me when done." } }]},
  { "role": "tool", "name": "send_message", "content": "Delivered to agent-p" },

  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": { "timeout": 60 } }]},
  { "role": "tool", "name": "wait_agent", "content": "Timed out. No agents responded." },

  { "role": "assistant", "tool_calls": [{ "name": "list_agents", "arguments": {} }]},
  { "role": "tool", "name": "list_agents", "content": "agent-p: running, agent-i: running, agent-v: idle" },

  { "role": "assistant", "tool_calls": [{ "name": "wait_agent", "arguments": { "timeout": 120 } }]},
  { "role": "tool", "name": "wait_agent", "content": "agent-p responded: Feature X complete. 3 files changed, all tests passing. Summary: ..." },

  { "role": "assistant", "content": "Feature X is built. Here's what was done..." }
]
```

Inter-agent collaboration happens entirely within the subagents' contexts. The main agent only sees what is explicitly reported back. Addressing can use hierarchical paths, mailboxes, or IPC.

**When to use this:** Large tasks where the coordination logic exceeds what a single agent can manage step-by-step.

**Limitations:** Every agent in the team needs frontier-class model capabilities, not just the main agent. Each team member must independently decide when to message teammates, what to include, and when to report back. Models can message the wrong agent, forget to report completion, or get stuck in loops. Beyond model capability, there are infrastructure challenges: cycle detection (A waits on B, B waits on A), conflict resolution (two agents edit the same file), and shutdown coordination. Debugging is hard because message chains between agents are difficult to trace and failures cascade.

---

## Choosing a Pattern

| Pattern      | Tools                                 | Main Agent Role | Agent Lifetime |
|-------------|---------------------------------------|-----------------|---------------|
| Inline Tool | call_agent                            | Caller          | Single task    |
| Fan-Out     | spawn_agent, wait_agent               | Dispatcher      | Single task    |
| Agent Pool  | spawn, send, wait, list, kill         | Coordinator     | Multi-turn     |
| Teams       | All above + cross-agent send_message  | Supervisor      | Persistent     |

Start with Pattern 1; most tasks don't need complex orchestration. Move up only as needed for parallel work (2), multi-step collaboration (3), or complex team coordination (4).

Higher patterns demand more capable models. Pattern 1 works broadly. Pattern 2 requires reasoning about wait times. Pattern 3 needs multi-turn state tracking. Pattern 4 requires frontier models for all participants.

---

Originally published at: [https://www.philschmid.de/subagent-patterns-2026](https://www.philschmid.de/subagent-patterns-2026)
