# Hermes Kanban Turns Multi-Agent Work Into a Real Board

[![Neo profile image](https://pbs.twimg.com/profile_images/2010897551525101579/ZKJQsu7V_normal.jpg)](https://x.com/NeoAIForecast)

[Neo](https://x.com/NeoAIForecast) [@NeoAIForecast](https://x.com/NeoAIForecast)

[![Hermes Kanban board image](https://pbs.twimg.com/media/HHgWZeUaUAAzq7h?format=jpg&name=small)](https://x.com/NeoAIForecast/article/2051443615768228062/media/2051414257158475776)

---

Most “multi-agent” workflows are still too invisible.

You ask one agent to split work across subagents. They run somewhere inside the process. A result comes back into the parent context. If everything succeeds, great.

But real work is messier.

Agents need to wait on each other. Humans need to unblock tasks. Workers crash. Credentials go missing. Reviewers reject first attempts. Long projects need an audit trail that survives restarts, retries, and context compression.

That is what Hermes Kanban is built for.

It turns multi-agent work into a durable, SQLite-backed board where every task, run, comment, dependency, and handoff becomes inspectable state.

**Takeaway:** Hermes Kanban is not just “agents with a task list.” It is a real coordination layer for named AI workers.

---

## The problem with invisible subagents

The standard multi-agent pattern looks like this:

1. Parent agent receives a goal
2. Parent splits it into subtasks
3. Child agents run
4. Parent waits for answers
5. Everything collapses back into one final response

That works for short, synchronous reasoning.

It breaks down when the work needs to survive outside the parent’s context window.

- If a worker blocks, where does the reason live?
- If a reviewer rejects an implementation, how does the next run know what failed?
- If a process crashes, who reclaims the task?
- If a human wants to inspect progress, where do they look?

Hermes already has `delegate_task` for fork-and-join subagent work. Kanban solves a different problem.

- `delegate_task` is a function call.
- Kanban is a durable work queue plus state machine.

**Takeaway:** Use `delegate_task` when the parent needs a short answer back. Use Kanban when work needs persistence, retry, visibility, human input, or multiple roles over time.

---

## What Hermes Kanban adds

Hermes Kanban gives agents a shared board that lives outside any single chat session.

The default board is stored at:

```bash
~/.hermes/kanban.db
```

Newer multi-board support lets you create isolated boards per project, repo, client, or domain:

```bash
hermes kanban boards create atm10-server \
  --name "ATM10 Server" \
  --description "Minecraft modded server ops" \
  --icon 🎮 \
  --switch
```

Additional boards live under:

```bash
~/.hermes/kanban/boards/<slug>/kanban.db
```

Each board has its own database, workspaces, logs, and dispatcher scope.

That means your “AI news pipeline,” “Minecraft server ops,” “GitHub PR reviews,” and “client research” can be separate queues instead of one giant mixed backlog.

**Takeaway:** Kanban is now multi-project, not just multi-agent.

---

## Named profiles, not anonymous workers

This is the key mental model.

Kanban tasks are assigned to Hermes profiles.

A task might go to:

```bash
researcher
writer
backend-dev
reviewer
ops
```

When the dispatcher picks up a ready task, it spawns the assigned profile as a full worker process.

That worker has its own identity, memory, skills, toolsets, logs, and operating context.

This is very different from an anonymous subagent call.

For example:

```bash
hermes kanban create "Research new open-weight model releases" \
  --assignee researcher \
  --tenant ai-news
```

The researcher profile is now responsible for that task. If the task blocks, crashes, or gets retried, the board records what happened.

**Takeaway:** Hermes Kanban coordinates persistent named agents, not disposable hidden calls.

---

## The new board model

Hermes Kanban now has a clearer board model:

- **Board:** one isolated queue, usually one per project or domain
- **Task:** a unit of work with title, body, assignee, priority, tenant, status
- **Link:** a parent-to-child dependency between tasks
- **Comment:** durable human or agent notes
- **Run:** one worker attempt on a task
- **Workspace:** where the worker operates
- **Dispatcher:** the loop that promotes and claims ready work

Task statuses include:

```plaintext
triage
todo
ready
running
blocked
done
archived
```

That gives you a real workflow.

- Raw ideas can sit in triage.
- Dependency-waiting tasks stay in todo.
- Runnable tasks move to ready.
- Claimed tasks become running.
- Human-needed tasks go to blocked.
- Finished tasks move to done.
- Old tasks can be archived.

**Takeaway:** Kanban gives agents a shared state machine, not just a list of prompts.

---

## The dispatcher now lives in the gateway

One of the most important operational updates: the dispatcher runs inside the Hermes gateway by default.

You start the gateway:

```bash
hermes gateway start
```

Then the embedded Kanban dispatcher wakes up every 60 seconds by default and:

1. Reclaims stale claims
2. Detects crashed workers
3. Promotes dependency-satisfied tasks
4. Claims ready tasks atomically
5. Spawns the assigned profile
6. Pins the worker to the correct board

The config looks like this:

```yaml
kanban:
  dispatch_in_gateway: true
  dispatch_interval_seconds: 60
```

The old standalone daemon path exists only as a deprecated escape hatch:

```bash
hermes kanban daemon --force
```

But the intended path is simple: run the gateway, let it dispatch.

**Takeaway:** Kanban is now part of the normal Hermes runtime. No separate daemon should be needed for most users.

---

## Humans use CLI, slash commands, and dashboard

There are now three human-facing ways to operate the board.

1. **CLI**

    ```bash
    hermes kanban list
    hermes kanban show t_abc
    hermes kanban comment t_abc "Need to check the source link"
    hermes kanban unblock t_abc
    hermes kanban stats
    ```

2. **Slash command in Hermes Agent**

    From chat, you can use the same surface:

    ```bash
    /kanban list
    /kanban show t_abc
    /kanban create "Draft AI recap" --assignee writer
    /kanban unblock t_abc
    /kanban boards list
    /kanban --board atm10-server list
    ```

    > A useful update: `/kanban create` can auto-subscribe the originating chat to terminal events for the new task, so you get notified when it completes, blocks, gives up, crashes, or times out.

3. **Dashboard**

    The Hermes dashboard now includes a Kanban plugin.

    Start it:

    ```bash
    hermes dashboard
    ```

    Then open the Kanban tab.

    The dashboard gives you:

    - Columns for task status
    - Board switcher
    - Card drawer with run history
    - Comments
    - Parent and child links
    - Filters for search, tenant, and assignee
    - “Lanes by profile” view
    - “Nudge dispatcher” button
    - Drag-and-drop status changes
    - Live updates from task events

**Takeaway:** Humans can supervise multi-agent work without reading logs or waiting for a final response.

---

## Workers do not shell out

This is a subtle but important update.

Humans and scripts use:

```bash
hermes kanban ...
```

Workers do not.

When the dispatcher spawns a worker, it sets:

```bash
HERMES_KANBAN_TASK=<task_id>
HERMES_KANBAN_BOARD=<board_slug>
```

That environment enables a dedicated `kanban_*` toolset inside the worker’s model schema:

```plaintext
kanban_show
kanban_complete
kanban_block
kanban_heartbeat
kanban_comment
kanban_create
kanban_link
```

So a worker starts by calling:

```python
kanban_show()
```

Not:

```bash
hermes kanban show t_abc
```

That matters because the worker’s terminal backend might be local, Docker, SSH, Modal, Singularity, or something else. The Kanban tools run in the agent’s own Python process and route through the same `kanban_db` layer as the CLI and dashboard.

Even better:

Normal Hermes sessions do not carry these tools. The `kanban_*` tools only appear when `HERMES_KANBAN_TASK` is set.

No Kanban task, no Kanban schema bloat.

**Takeaway:** Kanban workers interact through gated tool calls, not shell commands.

---

## Dependencies become automatic promotion

You can link tasks into pipelines:

```bash
SCHEMA=$(hermes kanban create "Design auth schema" \
  --assignee backend-dev \
  --json | jq -r .id)

API=$(hermes kanban create "Implement auth API endpoints" \
  --assignee backend-dev \
  --parent $SCHEMA \
  --json | jq -r .id)

hermes kanban create "Write auth integration tests" \
  --assignee qa-dev \
  --parent $API
```

Only the first task starts as ready.

The API task waits until the schema task is done.

The tests wait until the API task is done.

When a parent completes, the dependency engine promotes eligible children from todo to ready.

That lets you build real agent pipelines:

specifier → implementer → reviewer → writer → publisher

[![Agent pipeline image](https://pbs.twimg.com/media/HHgtDAPaQAAzd-p?format=jpg&name=900x900)](https://x.com/NeoAIForecast/article/2051443615768228062/media/2051439159894753280)

or

researcher A + researcher B → analyst → writer → editor

[![Research pipeline image](https://pbs.twimg.com/media/HHgtRF3bYAAlU1A?format=jpg&name=small)](https://x.com/NeoAIForecast/article/2051443615768228062/media/2051439401922945024)

**Takeaway:** Kanban lets you express multi-agent sequencing without forcing the parent agent to babysit every step.

---

## Retry history is first-class

Real work often fails on the first attempt.

- A worker might block because credentials are missing.
- A reviewer might reject an implementation.
- A process might crash mid-run.

Hermes Kanban stores each attempt as a run row.

A task can have:

```plaintext
Run 1: blocked
Run 2: completed
```

Or:

```plaintext
Run 1: spawn_failed
Run 2: spawn_failed
Run 3: gave_up
```

or:

```plaintext
Run 1: crashed
Run 2: completed
```

When the task is retried, the next worker sees prior attempts in its context.

That means it does not repeat the same failed path blindly.

Example:

```python
kanban_block(
    reason="Review: password strength check missing, reset link can be replayed"
)
```

After a human unblocks it:

```bash
hermes kanban unblock t_abc
```

The next run sees the prior block reason and knows exactly what to fix.

**Takeaway:** Kanban makes retries stateful. The second attempt inherits the lessons of the first.

---

## Crash recovery and circuit breakers

The dispatcher also handles ugly operational failures.

If a worker process dies, the dispatcher can detect the dead PID, release the claim, and move the task back to ready.

If a task repeatedly fails to spawn, the dispatcher eventually stops thrashing and auto-blocks it with the last error. That is critical for missing credentials, broken profiles, bad workspaces, or unavailable tools.

Instead of silently looping forever, the board records:

```plaintext
spawn_failed
spawn_failed
gave_up
```

And a gateway notification can tell you the task gave up.

**Takeaway:** Hermes Kanban treats failure as board state, not an invisible runtime accident.

---

## Multi-board support changes the scale

The newest board updates matter because serious agent setups are rarely one project.

You might want something like:

```plaintext
default
ai-news
github-review
local-llm-wiki
home-ops
client-research
```

Each board can be managed separately:

```bash
hermes kanban boards list
hermes kanban boards create ai-news --switch
hermes kanban --board github-review list
hermes kanban boards switch local-llm-wiki
```

Board resolution order is:

1. Explicit --board <slug>
2. HERMES_KANBAN_BOARD
3. ~/.hermes/kanban/current
4. default

Workers are pinned to their board via environment variables, so they cannot accidentally read tasks from another board.

Tenants still exist inside boards as a softer namespace, but boards are the real isolation boundary.

**Takeaway:** Boards are for project separation. Tenants are filters inside a board.

---

## The practical setup

For a simple Kanban workflow:

```bash
# Initialize the board
hermes kanban init

# Start the gateway, which hosts the dispatcher
hermes gateway start

# Create work
hermes kanban create "Research AI funding landscape" \
  --assignee researcher

# Watch live events
hermes kanban watch

# Inspect state
hermes kanban list
hermes kanban stats
```

For a project-specific board:

```bash
hermes kanban boards create ai-news \
  --name "AI News Pipeline" \
  --description "Daily research, writing, and publishing workflow" \
  --switch

hermes kanban create "Collect official AI announcements" \
  --assignee researcher \
  --tenant daily-recap

hermes kanban create "Draft X article from research" \
  --assignee writer \
  --tenant daily-recap
```

For chat-native use:

```plaintext
/kanban boards list
/kanban create "Check blocked ops tasks" --assignee ops
/kanban list --mine
/kanban unblock t_abc
```

**Takeaway:** You can run Hermes Kanban from terminal, dashboard, chat, scripts, cron, or agents themselves.

---

## Where this becomes powerful

Kanban is especially useful when the workflow has one of these shapes:

1. **Research fan-out**
    - Multiple researcher profiles collect sources in parallel. An analyst synthesizes. A writer drafts. A reviewer checks claims.
2. **Engineering pipelines**
    - A PM writes a spec. Backend implements. QA writes tests. Reviewer checks the diff. Ops deploys.
3. **Human-in-the-loop operations**
    - A worker blocks when it needs a credential, approval, or decision. You comment or unblock from chat.
4. **Fleet farming**
    - One specialist profile drains many similar tasks: transcripts, translations, product descriptions, issue triage, account checks.
5. **Scheduled ops**
    - Cron creates recurring Kanban tasks with idempotency keys. The board becomes a durable operational journal.
6. **Digital twins**
    - Named profiles like inbox-triage, researcher, ops-review, or wiki-maintainer accumulate memory and handle recurring domains over time.

**Takeaway:** Kanban is best when work has memory, state, retries, supervision, and role boundaries.

---

## Conclusion

Hermes Kanban is the point where multi-agent work stops feeling like a hidden prompt trick and starts feeling like an operating system.

The board gives every task a place to live. The dispatcher gives every worker a way to start. Structured handoffs give every downstream agent the context it needs. Run history makes failures inspectable instead of mysterious. The dashboard, CLI, and /kanban command give humans a real control surface.

That is the shift.

Not “more agents.”

Better coordination.

Hermes Kanban makes agent work visible, durable, and steerable, which is exactly what multi-agent systems need if they are going to move from demos into daily operations.

**Final takeaway:** The future of agents is not just smarter models. It is better infrastructure for assigning work, preserving context, recovering from failure, and letting humans stay in the loop.
