# Building file system transactions for agents

![Hunter Leath](https://pbs.twimg.com/profile_images/1725510925610840064/z9JLeyDz_normal.jpg)

By [Hunter Leath](https://x.com/jhleath) [@jhleath](https://x.com/jhleath)

![Image](https://pbs.twimg.com/media/HHQDTEAa8AAbAk7?format=jpg&name=small)

There are a lot of reasons why object storage solutions like S3 became so popular for cloud application development in the past 20 years with "low-cost" and "ubiquity" being two. I think there's a bigger reason that we don't talk about quite enough: the unit of atomicity.

What do I mean?

There's a simple property about S3 that's totally elusive for file systems. When you call PutObject, you know that the object is stored (and readable, thanks to strong read-after-write consistency) if the call returns successfully. If it fails, then you know that it's not there.

This is absolutely not the case with file systems. File systems are designed for applications to send thousands of tiny operations -- writes, attribute modifications, and file creations. If each one of those operations were "atomic" and written to disk, the file system would crawl to a halt.

Instead, file systems are designed to implement a write barrier: fsync. Once your application calls "fsync", then you know that everything you executed before that operation has been durably committed to disk and is visible upon restart.

![Image](https://pbs.twimg.com/media/HHP4vbIakAAV4r6?format=jpg&name=small)

If you've been around systems long enough, this should beg an obvious question. If these operations are discrete, then what happens if a viewer observes these intermediate states? What happens if you crash in the middle of performing them? What happens if fsync *doesn't* return successfully? Do we get the same semantics of S3, where our file isn't visible to others?

Unfortunately, not quite. File systems make it possible that any of these intermediate states are viewable: empty files, partially written files, files not appearing in their directory at all.

In fact, if you want a taste of just *how* complex this can get, I would recommend watching [@danluu](https://x.com/@danluu)'s [talk from Deconstruct Conf 2020](https://www.deconstructconf.com/2019/dan-luu-files) (it's required annual viewing for engineers at Archil).

For example, TextEdit -- yes the simple text editor that comes with MacOS -- performs the following 5-step process each time you save edits to an already-existing file:

![Image](https://pbs.twimg.com/media/HHP6iWsbcAANnn1?format=png&name=small)

This complexity is why writing file based applications are so hard, and partially why in-process databases like SQLite appeared on the scene. It's easier to write to the file system in a more developer-friendly way if you outsource the actual writing to something that already comes with it's own write-ahead log.

This is insanity, and it's starting to break down in the age of agents.

What's the model that developers really need in order to build agents?

S3's unit of atomicity is larger than the file system's. Rather than little metadata commands and individual writes, it's at the level of the file. Surprisingly, this still isn't enough for what's coming next. The unit being a file made sense when we were doing map-reduce on big-data, and each unit of work was to transform exactly one file into one output.

Agents work in a workspace. They have context that spans multiple systems, and the work that they do -- writing code, calling APIs -- spans multiple files. Therefore, the unit of atomicity needs to be *bigger* than what you get with S3.

The unit of atomicity for agents is a task, a goal, or a turn. You want *all* of the work that the agent did in that turn to be either: completely committed to disk such that other agents can see the side-effects of that change, or not at all.

This isn't something that's currently supported in any of the major cloud platforms in a developer-friendly way, and it's one of the many reasons why we're starting to see git-based "file-systems" like Cloudflare Artifacts pop onto the scene. It's easy to see how Git can be used to get these kind of semantics. For example, an agent can roll a bunch of unrelated changes to the file system into a single commit, and then other users of that repository either get the entire commit (if it makes it to the main branch) or nothing.

This makes git work similar to how developers used to use SQLite.

![Image](https://pbs.twimg.com/media/HHP9MpNaAAAxdGy?format=png&name=small)

Unfortunately, like SQLite, it's not a scaleable solution to the generalized data problem. In both cases, we're trying to stuff, potentially terabytes of data, into a data structure that wasn't built for it.

This is one of the reasons why we're adding checkpoints and branching as a first-class citizen in Archil, starting next week ([beta docs are available here](https://docs.archil.com/concepts/branches-and-checkpoints)).

We think that checkpoints are the first solution to providing these semantics to agent builders -- that scales to the agents that need to work with terabytes and petabytes of data. By checkpointing the file system before and after a turn, it gives developers the ability to easily roll-back to before a turn if anything went wrong. With branching, agents get the opportunity to explore multiple different ways to accomplish a task, without worrying about overwriting the single copy of data that exists.

![Image](https://pbs.twimg.com/media/HHP_B65bsAAfLFy?format=jpg&name=small)

We think this solves for a tremendous amount of pain that agent builders have today, and because they're copy-on-write, they're nearly free to use.

Notably, this helps to solve the "atomicity issue" associated with file systems. If your operations, including fsync, completed successfully -- no matter how many files you touched, you have the ability to create a new checkpoint. If the checkpoint isn't created, you can resume your work from the previous checkpoint -- ignoring the potential partial writes that could normally occur on a file system. This requires an entirely new architecture for snapshots that's designed for speed and quantity, unlike existing storage providers.

However, we don't think that this is enough to fully solve for all of the problems that builders are thinking about today. What's left? Merging.

Checkpoints are a traditional storage concept used in file systems from NetApp on to help manage data, but the intention is that these branches always diverge and never come back together.

One of the other appeals of a git-based storage product is that you get this ability to "merge" your work back into a linear history.

We think this requires that the file system, as a primitive, continues to evolve to look a little bit like something else... a database. The good news is that we've already innovated on this front with Archil by starting to allow users to give information about how to use their data with Serverless execution -- [that turns bash into the SQL of file systems](https://x.com/compose/articles/edit/2044420745905000448).

If we continue to pull this thread more, we can actually continue to think of each call to Serverless execution as a different database transaction on top of the file system primitive, and use these "transactions" to provide a way to either accept their changes into the main file system or not depending on concurrent conflicts.

![Image](https://pbs.twimg.com/media/HHQB00XboAA6MEh?format=png&name=small)

This is only possible with Archil because we've increased the abstraction layer to include the compute that you are using to interact with the file system.

We think that starting to think about bash tool executions as database transactions that occur over your context will become the predominate way that agents think about interacting with file systems. It not only provides the semantics that developers are expecting after working with tools like S3, but it's also the only *scaleable* way to work with this data.

We expect that the data storage landscape is going to change tremendously over the next several years, and we're excited to be building the only primitive designed to scale for these next set of applications. You'll be able to try checkpoints and branching on Archil file systems starting next week.

If the problems associated with defining the storage infrastructure and interfaces for the next generation of applications is exciting, [we're always hiring more great engineers](https://jobs.ashbyhq.com/archil).

---

*1:33 PM · May 1, 2026 · 3,149 Views*