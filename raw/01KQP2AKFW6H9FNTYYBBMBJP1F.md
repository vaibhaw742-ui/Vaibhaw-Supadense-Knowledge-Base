# The Database Is No Longer Storage - It Is Becoming the Runtime for AI

![Image 1](https://pbs.twimg.com/profile_images/497634078980468737/SxGpbB3x_normal.png)

By [siddontang](https://x.com/siddontang) [@siddontang](https://x.com/siddontang)

![Image 2: Image](https://pbs.twimg.com/media/HGcDBYuagAAa3qP?format=jpg&name=small)

This is Part 5 of a series, and also the final part.

**Part 1:**  [SaaS is Dead - The Rise of Result as a Service](https://x.com/siddontang/status/2032696118988189742)

**Part 2:**  [When Databases Start Eating Software - The Paradigm Shift in the AI Era](https://x.com/siddontang/status/2033391893883818470)

**Part 3:**  [Database as the Runtime Layer for AI Agents - Why Storage Is No Longer Enough](https://x.com/siddontang/status/2039890576607429117)

**Part 4:**  [The End of SaaS - Except for a Few Types](https://x.com/siddontang/status/2046497828277772605)

---

## Ten Years Ago, the Job Was Simpler

When we started building TiDB, the problem was pretty clear.

Applications were scaling faster than databases.

People were dealing with sharding, painful operations, weird consistency issues, and all the usual distributed-systems fun.

So the mission was simple:

> Build a database that scales like the internet.

That became TiDB.

A distributed SQL database that scales horizontally without turning transactions into a philosophical debate.

Back then, the architecture was also easier to explain.

Applications had the logic. Databases stored the data.

Everyone more or less agreed on the job description.

---

## Then Cloud Happened

As cloud became the default, the next problem was not only scale.

It was usability.

Distributed databases are great until you actually have to run them.

So the next step was obvious:

> Make distributed databases easy to use.

That led to TiDB Cloud.

The goal was not mysterious. Give developers the power of distributed SQL without also giving them a new part-time job in infrastructure therapy.

That was an important change.

But it still did not change the role of the database.

The application was still the center. The database was still the storage layer.

Then AI showed up and started breaking the mental model.

---

## AI Is Not Just Another Workload

At first, AI looked like a familiar pattern.

Store embeddings. Add vector search. Bolt on retrieval. Call it innovation.

That was the easy interpretation.

I think it was also incomplete.

The bigger change is not that AI adds new data types.

The bigger change is that agents use data differently from traditional software.

Traditional applications are mostly deterministic.

They take a request, run a workflow, write a result, and go home.

Agents do not go home.

They read context, reason about it, take action, write new state, read that state again, and keep looping.

That is a very different shape of system.

The software is no longer just executing a fixed program.

It is continuously operating through data.

---

## The Center of the Stack Moves

The old architecture looked like this:

> Human → Application → Database

Simple. Clean. Familiar.

The new one increasingly looks like this:

> Human → Agent → Database → Result

And yes, the model matters.

But the more interesting shift is not “the model is the new app”.

It is that the database becomes much more central, because that is where context, memory, coordination, and durable state actually live.

Agents are not useful because they generate tokens.

They are useful because they can operate against state.

Without state, an agent is just a very enthusiastic intern with short-term memory loss.

---

## So the Database Stops Being Just Storage

Once you build real agent systems, the old description of the database starts sounding incomplete.

The database is no longer just where you put records after the real work is done.

It starts doing much more of the real work.

It needs to provide:

- memory for agents
- coordination across processes
- consistency under concurrency
- history you can query later
- isolation across users, tasks, and contexts

At some point, calling this “storage” feels like calling Kubernetes “a place to keep YAML”.

Technically not wrong. But also not the point.

The database is becoming part of the runtime.

---

## This Changes the Engineering Requirements

And once you see the database as part of the runtime, a lot of infrastructure assumptions break.

Agent workloads are weird.

They are bursty, concurrent, unpredictable, and often mix transactional and analytical patterns in the same system.

They do not behave like normal application traffic, because the thing generating the workload is no longer just a human clicking buttons.

It is software generating more software activity.

Which means infrastructure has to deal with:

- elastic compute
- large-scale durable state
- many isolated contexts
- mixed OLTP and OLAP patterns
- economics that still make sense when machines are the main users

Traditional database systems were mostly designed for applications serving humans.

We are moving toward systems increasingly serving agents.

That is not a small parameter change. That is a different design point.

---

## Why This Leads to TiDB X

This is the thinking behind TiDB X.

TiDB X starts from a different assumption:

most future database traffic will not come directly from humans.

It will come from machines acting on behalf of humans.

That sounds small, but it changes a lot.

If workloads are dynamic, infrastructure has to scale differently. If compute demand is spiky, economics have to look different. If state is the operating surface for intelligence, storage and compute need a tighter relationship.

That is why ideas like these start to matter more:

- object-storage-native persistence
- elastic compute
- usage-based pricing
- unified transactional and analytical workloads

This is not just “let’s make databases cheaper”.

It is a shift in what the database is for.

---

## Looking Back, the Path Makes More Sense Now

If I look back at the past decade, the progression feels surprisingly consistent.

First, the problem was scalability. Then it was usability. Now it is the role of the database itself.

We started by building a database for modern applications.

Now we are exploring what it means to build infrastructure for intelligent systems that operate through data.

Same direction. Bigger implication.

---

## The Next Ten Years

I do not think the next decade will be defined mainly by software categories.

Not CRM. Not ERP. Not even SaaS.

It will be defined by a much simpler question:

> How fast can intent become result?

Agents will generate workflows dynamically. Applications will get thinner. And the systems that manage memory, state, coordination, and history will become more important than ever.

Which is why I think databases are moving back to the center of software architecture.

Not as storage.

As runtime.

---

## Final Thought

Ten years ago, we thought we were building a scalable database.

Now it feels more like we were building part of the execution layer for a new kind of software.

Turns out “store the data” was only the beginning.
