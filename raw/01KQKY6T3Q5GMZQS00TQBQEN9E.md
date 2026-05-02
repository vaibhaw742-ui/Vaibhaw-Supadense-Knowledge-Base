# Systems Engineering: Building Agentic Software That Works

![Ashpreet Bedi](https://pbs.twimg.com/profile_images/2024047696827273217/vlW-RvPT_normal.jpg)

By [Ashpreet Bedi](https://x.com/ashpreetbedi) [@ashpreetbedi](https://x.com/ashpreetbedi)

![Image](https://pbs.twimg.com/media/HFUUjU_WwAA4H2P?format=png&name=small)

---

In the early 1940s, Bell Labs was building the national telephone network, the most complex technical system in the world at the time. Millions of switches, cables, relays, and operators had to work together. The engineers discovered something that would become an 80-year-old lesson: you can't optimize a system by optimizing individual components. The behavior of the whole (call routing, reliability, capacity, cost) emerged from how the parts interacted. They needed a discipline focused on the interactions between components.

They called it **systems engineering**.

The agentic software ecosystem is repeating the same mistakes that spawned that discipline. The current wave of harness engineering will ask you to use the filesystem for storage and memory, then try to get around its limitations by building a virtualized file system over a database. It'll ask you to use bash as a general-purpose tool, then force you into per-request sandboxes to handle security. These are symptoms of optimizing one part without considering the system as a whole. And we're buying into it without taking a minute to ask if it's the right approach.

## Software Engineering Is Systems Engineering

Coding agents have lowered the barrier to writing code, but they haven't lowered the requirements of production software. Software engineering is, and has always been, systems engineering and if you're building agentic software, your system needs to bridge these five layers:

1. **Agent Engineering:** Your agent or multi-agent logic and execution flow.
   
   Model, system instructions, tool configurations, handoffs, context management, observability. This is where you define what your agent does, how it runs and how it responds. Your agent's behavior should be deterministic where possible and observable where it isn't.

2. **Data Engineering:** Your agent is only as good as the context it has access to, and context is just data under the hood. Memory, storage, knowledge: all should be managed with data engineering principles. This means well designed schemas, structured querying, databases for fast read/writes, object storage for long-term storage, and pipelines that keep your knowledge and memory up to date. The patterns are decades old. Use them.

3. **Security Engineering:** Auth, RBAC, governance, data isolation, audit trails.
   
   Your agent's capabilities are defined by its tools, and those tools should be scoped with JWT-backed permissions. Read-only access IS NOT a prompt instruction, it's a tool configuration. Actions should have approval tiers: reads run freely, writes need user approval, sensitive operations need admin sign-off. Most actions should be logged and queryable for the life of the product.

   And please, isolate requests. One user's context bleeding into another's is a data breach, not a bug. It has serious consequences and there are laws protecting user data. So filesystem backed memory on a shared sandbox might not be a good idea.

4. **Interface Engineering:** How users and other agents reach your agent.
   
   REST API, Slack, MCP server, terminal. In the old world, you had one API and one client. Now you have multiple surfaces, each with its own identity system. A Slack user ID is not your product's user ID. An MCP client authenticating as another agent is not a human user. Interface engineering is about making sure your auth, policies, and access controls hold consistently across every surface your agent is reachable from.

5. **Infrastructure Engineering:** How you run and scale your software. Containers, cloud deployment, horizontal scaling. Generally called DevOps.

   The good news: 95% of this is identical to running any other service. Re-use existing patterns, they'll serve you well. The 5% that's different: agent requests take longer (increase your load balancer timeouts), responses stream (plan for SSE or WebSockets), and the best agents are proactive (scheduled tasks, background execution). None of this is new.

The key unlock for AI engineers is realizing that agentic software is just regular software, with the business logic replaced by agents, and interfaces going from request/response to streaming across multiple surfaces.

Systems engineering is the discipline of making these parts work together, and is the key to building agentic software that works.

When you look at your software from a systems perspective, the right decisions become obvious and you stop debating MCP vs CLI. You give your agent well-scoped tools, not unfettered bash access. You store sessions, memory, and knowledge in a database, not files.

When you design one layer in isolation, you inherit constraints that cascade through the rest of the system and you waste time and resources patching those constraints. When you design from the system's perspective, each layer reinforces the others.

## Systems Engineering in Practice

I can't make a claim this big and not give you working code.

[Dash is an open-source, self-learning data agent](https://github.com/agno-agi/dash). You ask it questions in plain English, it writes SQL, runs it, and tells you what the numbers mean. Simple enough to clone and adapt. Real enough to demonstrate all five layers.

Here's how each layer works in dash:

### Agent Engineering

Dash is a team of agents. A Leader routes requests to two specialists: an Analyst that queries data (read-only) and an Engineer that builds computed assets like views and summary tables. Each specialist gets the same tool types wired to different capabilities. The Analyst's SQL tools connect to a read-only database engine. The Engineer's SQL tools connect to a writable engine scoped to a single schema. Same interface, different permissions, determined by configuration, not prompts. Instructions are assembled at runtime from table metadata and business rules.

### Data Engineering

Six layers of context, and tools for learning.

Raw LLMs writing SQL hit a wall fast: schemas lack meaning, types are misleading, tribal knowledge is missing, there's no way to learn from mistakes. Dash solves this with six layers of grounded context:

- Table metadata (schema, columns, relationships)
- Human annotations (metrics, definitions, business rules)
- Query patterns (SQL that is known to work)
- Institutional knowledge (docs, wikis)
- Learnings (error patterns and discovered fixes)
- Runtime context (live schema inspection)

These layers feed two systems:

- The first is curated knowledge: table schemas, validated queries, and business rules loaded into PostgreSQL.
- The second is discovered learnings: error patterns and fixes that the agent saves when it hits problems and recalls on future queries.

The learning loop is simple: the agent runs a query, gets a type error, diagnoses the fix, saves it. Next time it sees a similar column, it gets it right the first time. And when the Engineer creates a new view, it records the schema and example queries into the knowledge base. The Analyst discovers it on the next search and starts using it. Query 100 is better than query 1, not because the model improved, but because the data layer got better.

### Security Engineering

Enforced by the system, not the prompt.

Dash in production uses RBAC with JWT verification. Every query is scoped to `user_id`. An eval suite tests these boundaries directly: it prompts the agents to leak credentials, execute destructive SQL, and cross schema boundaries, then verifies they can't.

Security is a system property tested across layers. The Analyst's read-only access is a PostgreSQL connection parameter. The database itself rejects writes regardless of what the model generates. The Engineer can write, but only to a single schema: a query-level guard blocks any operation targeting the source data.

### Interface Engineering

Dash is available as a REST API, a Slack bot, a web UI, and a CLI. Each surface handles identity differently: Slack maps thread timestamps to sessions, the API uses JWT tokens in production. But all four hit the same agents, same tools, same knowledge.

Your auth and access controls need to hold across every surface, because the agent doesn't know which one it's being called from.

### Infrastructure Engineering

Minimal python container. Docker compose for local development. Deploy to your cloud of choice. Streaming via SSE through a standard ASGI server. The 95% that's identical to any other service is identical. The 5% that's different (longer timeouts, streaming, scheduled tasks) is handled with standard tools.

You can clone it, run `docker compose up`, and have the entire system.

One command, five layers, a working product thanks to systems engineering.

Here's the link for reference:

[https://github.com/agno-agi/dash](https://github.com/agno-agi/dash)
