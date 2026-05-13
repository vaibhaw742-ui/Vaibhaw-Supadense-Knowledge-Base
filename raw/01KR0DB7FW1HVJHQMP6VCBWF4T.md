# How to Build a Self-Improving AI Lead Gen Agent on Hermes

[Michel Lieben](https://x.com/MichLieben)  
[@MichLieben](https://x.com/MichLieben)

[![Image 2: Image](https://pbs.twimg.com/media/HHkeG9HbIAAWNPw?format=jpg&name=small)](https://x.com/MichLieben/article/2051707320699396454/media/2051704210077720576)

By the end of this article you'll know how to build an AI lead gen agent that writes your content, runs your prospecting, books your meetings, and updates its own skills based on what works.

It runs from one terminal. It remembers everything across sessions. It gets smarter the more you use it.

Max Mitcham ([@max_trigify](https://x.com/@max_trigify)), founder of [Trigify.io](http://trigify.io/), just deployed this system. In two weeks his last YouTube video did 26x his usual reach. His Substack subscribers started growing consistently for the first time, with roughly 80 net adds in the same window. The AI SDR he's building runs on the same architecture.

Max and I just recorded a 56-minute breakdown of his setup, layer by layer. This article is the written playbook from that recording.

Here's exactly how to build it.

---

## The brain: memory as a Wikipedia graph

[![Image 3: Image](https://pbs.twimg.com/media/HHkAGmVaoAAKZdb?format=jpg&name=small)](https://x.com/MichLieben/article/2051707320699396454/media/2051671218613559296)

Most teams structure agent memory as a folder tree. One folder for sales, one for marketing, one for product. The folder structure breaks the moment a concept needs to live in two places at once.

Andrej Karpathy posted recently about a different model:

Markdown files inside Obsidian, linked to each other like a Wikipedia graph.

- Each concept becomes a file.
- Each link between concepts becomes an edge.
- The agent walks the graph before answering anything, picking up related context that a folder structure would have hidden underneath itself.

Max's agent memory looks like this in practice:

- "Founder-led content" links to Kieran Flanagan from HubSpot as a reference, signal-based selling as a content lane, Trigify-the-product, brand intelligence systems, and the Claude Code skills that drive his marketing.
- "Brand intelligence systems" links to competitor landscape mapping, the agent skills that perform that mapping, and the products being analyzed.
- Click "founder content" and the agent surfaces Kieran's frameworks. Click "signal-based selling" and you see how it shows up across product copy and content simultaneously.

The graph makes the agent's reasoning more like yours. Concepts have multiple anchors. Context pulls from every direction at once.

Hermes adds an active layer on top of the static graph. As you work with it, it auto-creates new skill files for recurring tasks, auto-creates new memory files for things you keep referencing, and auto-updates both based on how you actually use them. After two weeks of working with you, the graph reflects how you run your business, not the generic version some YouTube tutorial seeded.

This is the layer everything else compounds on top of. Build it first. Spend an afternoon seeding the vault with your ICP, your content pillars, your product features, your sales motions, the patterns you see in customer success, and the actual jargon your top customers use. Start linking concepts the moment a relationship exists between them.

---

## The content engine

[![Image 4: Image](https://pbs.twimg.com/media/HHkAOo1awAAe9Wa?format=png&name=small)](https://x.com/MichLieben/article/2051707320699396454/media/2051671356723609600)

The content engine has three input layers and one output layer.

- **Signal layer.** Trigify monitors X, LinkedIn, YouTube, and Substack for two things: trends gaining steam (potential content topics) and patterns in what's working at the format level (hooks, length, post structure, thumbnail patterns, opening lines). Both feed back into the brain.
- **Understanding layer.** Once a signal lands, the agent moves to longer-form content to build depth. Substack articles, podcast transcripts, YouTube videos. Trends are surface. Long-form is where the substance is. Without this layer the agent writes shallow because the source material is shallow.
- **Internal context layer.** Hermes is connected to Fireflies for sales calls, Granola for internal team calls, Slack channels, the email system, and Stripe. When it drafts a post for you, it pulls from your actual conversations, customer base, and product behavior, not just the trend feed.

The output layer assembles all three:

1. A draft Substack post that quotes a customer call from three weeks ago you'd forgotten about.
2. A LinkedIn post built on a trend that landed yesterday plus an internal data point you mentioned offhand in Slack.
3. A YouTube script structured around the patterns the agent saw working in the last 50 videos in your category.

Max wired this specifically into his YouTube content. The agent monitors top creators in his category (Alex Finn, Nate Herk, and others in the AI space), analyzes their thumbnails, hooks, descriptions, and chapter structure, and stores the patterns in the brain. It doesn't act on a single data point. It needs the same pattern to repeat enough times to become a belief.

Once a pattern crosses the threshold, the agent sends a Slack message: "I think it's time to update these titles to these. Try these new descriptions. Update the thumbnails on these three videos."

Max replies "approved."

The harness makes the changes via the YouTube API.

His most recent video did 26x his usual reach. The harness wrote the script, generated the thumbnail (in Gemini's banana 2 model), and chose the description and chapter structure. Max just filmed.

---

## The self-learning loop

[![Image 5: Image](https://pbs.twimg.com/media/HHkAY8YaIAAV3v0?format=png&name=small)](https://x.com/MichLieben/article/2051707320699396454/media/2051671533769334784)

The harness runs a cron over your performance data: reply rates, content engagement, meeting bookings, whatever you instrument. It generates hypotheses based on what it sees ("hooks containing a specific number outperform hooks without one"). It runs experiments against those hypotheses. It logs the results. It updates its own beliefs based on what worked.

Two design rules matter here.

1. **Patterns, not points.** A single post going viral doesn't trigger a belief update. The same pattern has to repeat enough times to clear a threshold. Otherwise you're chasing flukes.
2. **Beliefs are version-controlled.** When a new experiment contradicts an old belief, the belief updates. The system isn't locked into one playbook. It rewrites its own playbook as it learns.

The output is a Slack message asking for approval before applying any change. You're not handing over the controls. You're approving the changes the system has already validated against your own data.

The compounding effect is what most operators miss. After two months of running this loop, the agent has dozens of validated beliefs about your audience, your formats, your market, and your customer base. Those beliefs inform every new piece of content, every new outbound campaign, every new product decision the agent assists on. The longer it runs, the harder the system is for anyone else to copy.

---

## The AI SDR

[![Image 6: Image](https://pbs.twimg.com/media/HHkAjRIbIAAWr0i?format=png&name=small)](https://x.com/MichLieben/article/2051707320699396454/media/2051671711138127872)

The same architecture runs a sales motion. Max is building this live right now, and the structure is what makes it different from any off-the-shelf AI SDR.

Two signal sources feed it.

- **External signals** through Trigify, monitoring Substack, X, Hacker News, and LinkedIn for engagement on relevant topics, competitor mentions, funding rounds, role changes, company changes, headcount increases, hiring spikes, and new initiatives that match what your product helps with. Funding rounds in particular leak on X earlier than anywhere else, often before they hit Crunchbase.
- **Internal signals** built directly into your own systems: onboarding drop-off when someone signed up but never activated, spend thresholds when an account crosses a usage limit, upsell candidates, churn risks, and hiring signals from inside your existing customer base.

Hermes treats both feeds the same way. It runs a cron over each signal source, validates whether the signal is relevant against the ICP definition stored in the brain, and either routes the lead or drops it before any further work happens.

The ICP definition itself was built by Hermes, not by Max. The agent listened to every sales call (Fireflies), every internal team call (Granola), every customer conversation in Slack, and every Stripe transaction, and produced an ICP with three core segments and sub-personas under each. The kind of ICP definition that takes a sales team six months to write down formally. The agent surfaced it the first time Max asked.

Once a lead clears qualification, the workflow runs:

1. Query the API. Whichever data source the signal came from gets called for additional context on the company and the person.
2. Enrich. Lead Magic, Surfe, or whichever provider you've wired in. Two providers in the email layer matters because each has different strengths at scale.
3. Compose. This is where the brain layer pays off. Hermes pulls customer pain points from past calls, product context from internal documents, and copywriting patterns from a skill it has accumulated by studying examples Max fed it. The output is a message specific to that lead's signal trigger.
4. Route or send. Hot leads (enterprise-scale spend signal, returning website visitor, big-account engagement) route to Max for human approval. Everything else goes through Instantly programmatically.
5. Feed results back. Reply, no reply, meeting booked, deal closed. Each outcome updates the agent's beliefs about which signals, which copy, and which sequences are actually producing pipeline.

Most AI SDRs stop at step three. The compounding starts at step five.

---

## The stack Max runs end-to-end

[![Image 7: Image](https://pbs.twimg.com/media/HHkAupLbkAExiv3?format=jpg&name=small)](https://x.com/MichLieben/article/2051707320699396454/media/2051671906571751425)

- **Harness:** Hermes (orchestrator, ~90% of the work). OpenClaw kept around for the specific tasks that need heavy tool calling (~10%).
- **Models:** Codex 5.4 on the $200/month plan for heavy reasoning and coding work. Claude Code for front-end and lighter tasks.
- **Memory:** Obsidian for the Wikipedia-graph brain. Free.
- **Automation runtime:** Hermes for crons and sequences. [Trigger.dev](http://trigger.dev/) as an alternative if you need a sequential workflow engine outside the harness.
- **Signal data:** Trigify for social and intent monitoring across X, LinkedIn, Substack, Hacker News, and YouTube.
- **Email enrichment:** Lead Magic and Surfe (run two providers in parallel).
- **Outreach:** Instantly for cold email at scale.
- **Database and analytics:** Supabase for product data, PostHog for product analytics.
- **CRM:** Any CRM with full API access to objects, lists, and fields. We run Attio at ColdIQ for exactly this reason.
- **Call recordings:** Fireflies for sales calls, Granola for internal team calls. Granola is where the gold lives, because the internal conversations are what teach the agent how your team actually thinks about the business.
- **Interface:** Warp browser/terminal for working with the agent. Slack and Discord wired in for chat-based interaction.

Note: You don't have to read this stack as a buyer's guide. Read it as proof that the architecture works with substitutions too.

Swap CompanyEnrich for one of the enrichment tools. Swap Lemlist for Instantly. Swap Codex for whichever model is currently winning. The pattern remains the same because substitutions don't really break the workflow.

Full breakdown of the build, including the live AI SDR walkthrough, is in the video Quote Retweeted to this article.
