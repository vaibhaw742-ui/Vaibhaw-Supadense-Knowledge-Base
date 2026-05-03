# Coding Agents are Effective Long-Context Processors

Weili Cao    Xunjian Yin    Bhuwan Dhingra    Shuyan Zhou

## Abstract

Large Language Models (LLMs) have demonstrated remarkable progress in scaling to _access_ massive contexts. However, the access is via the latent and uninterpretable attention mechanisms, and LLMs fail to effective _process_ long context, exhibiting significant performance degradation as context length increases. In this work, we study whether long-context processing can be externalized from latent attention into explicit, executable interactions, by allowing coding agents to organize text in file systems and manipulate it using its native tools. We evaluate off-the-shelf frontier coding agents as the general interface for tasks that require processing long contexts, including long-context reasoning, retrieval-augmented generation, and open-domain question answering with large-scale corpus contains up to _three trillion_ tokens. Across multiple benchmarks, these agents outperform published state-of-the-art by 17.3% on average. We attribute this efficacy to two key factors: _native tool proficiency_, which enables agents to leverage executable code and terminal commands rather than passive semantic queries, and _file system familiarity_, which allows them to navigate massive text corpora as directory structures. These findings suggest that delegating long-context processing to coding agents offers an effective alternative to semantic search or context window scaling, opening new directions for long-context processing in LLMs. Our code is available at [this repository.](https://github.com/weilicao/Coding_Agents_are_Effective_Long_Context_Processors)

LLM Agent, Long Context, Coding Agent

---

## 1 Introduction

![Figure 1: Coding agents significantly outperform best published results across five long-context benchmarks spanning from 188K to three trillion tokens. Green percentages indicate relative improvement over prior state-of-the-art.](https://arxiv.org/html/2603.20432v1/x1.png)

![Figure 2: Iterative refinement example on Oolong-Real. When asked to identify the last spell cast by Vax’ildan in each episode of a 385K-token transcript, the coding agent wrote a Python script, discovered domain-specific spell references through failure analysis, and iteratively refined its logic.](https://arxiv.org/html/2603.20432v1/figures/coding_agent_example.png)

Modern applications increasingly require models to reason over massive corpora, such as scientific archives, or web-scale text collections. Recent advances in LLMs have significantly scaled supported context windows, with frontier systems now handling millions of tokens (Comanici et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib6); Anthropic, [2025](https://arxiv.org/html/2603.20432v1#bib.bib7)). Recent results show that drop-in long-context models can outperform retrieval-augmented generation (RAG) systems (Cao et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib3); Xu et al., [2023](https://arxiv.org/html/2603.20432v1#bib.bib1); Li et al., [2024](https://arxiv.org/html/2603.20432v1#bib.bib2); Jiang et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib4)), despite the extensive optimization of modern retrieval pipelines.

Despite these gains, long-context scaling primarily improves input access rather than effective processing. As context length grows, models suffer from context rot (Hong et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib9)), with performance degrading as context length increases (Bertsch et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib12); Li et al., [2025a](https://arxiv.org/html/2603.20432v1#bib.bib21); Hadeliya et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib22); He et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib23)). Moreover, reasoning remains latent and uninterpretable, as models provide little transparency into which parts of the context inform a given generation. While recent work has advanced interpretability of model internals (Gao et al., [2024](https://arxiv.org/html/2603.20432v1#bib.bib50); Zhou et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib51); Nanda et al., [2023](https://arxiv.org/html/2603.20432v1#bib.bib53)), these methods remain difficult to apply at scale (Sharkey et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib52)).

RAG addresses some of these challenges by externalizing long-context access through retrieval and reasoning stages. However, standard RAG pipelines rely on fixed, shallow retrieval mechanisms, which limit their ability to support iterative, multi-hop reasoning where intermediate findings must guide subsequent queries (Trivedi et al., [2023](https://arxiv.org/html/2603.20432v1#bib.bib54); Tang and Yang, [2024](https://arxiv.org/html/2603.20432v1#bib.bib55)). As a result, RAG systems offer limited flexibility for complex long-context processing tasks such as multi-hop question answering.

In this work, we propose a different approach based on a simple observation: coding agents, trained on large code repositories with long files and hierarchical structure, can transfer these skills to long-context text processing tasks. Rather than relying on latent attention or fixed retrieval, such agents can explicitly organize, filter, and transform text using executable programs.

As illustrated in [Figure 2](https://arxiv.org/html/2603.20432v1#S1.F2), when asked to identify the last spell cast by a specific character in each episode of a long transcript, a coding agent wrote a script that segmented the document by episode, filtered relevant mentions, and extracted spell names using pattern matching. When the initial script failed to capture many cases, the agent inspected intermediate outputs, discovered domain-specific spell references, and iteratively refined its logic. This iterative, programmatic interaction is difficult to realize with fixed retrieval pipelines or passive long-context attention, but arises naturally from the agent’s software engineering training.

Building on this intuition, we frame long-context processing as a file system navigation and manipulation problem with coding agents. We place massive text corpora into directory structures and delegate processing to off-the-shelf coding agents (OpenAI, [2025](https://arxiv.org/html/2603.20432v1#bib.bib48)), which can explore and manipulate these structures using familiar tools such as terminal commands, programmatic search, file manipulation, and iterative execution.

We evaluate coding agents on long-context QA benchmarks spanning two settings: BrowseComp-Plus (Chen et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib10)) and Natural Questions (Kwiatkowski et al., [2019](https://arxiv.org/html/2603.20432v1#bib.bib14)), which require synthesizing answers from information distributed across massive corpora; and LongBench (Bai et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib11)) and the Oolong benchmarks (Bertsch et al., [2025](https://arxiv.org/html/2603.20432v1#bib.bib12)), which require reasoning over individual long documents.

As shown in [Figure 1](https://arxiv.org/html/2603.20432v1#S1.F1), coding agents consistently outperform strong baselines across all settings, establishing new state-of-the-art results on _four out of five_ benchmarks and remaining competitive on the remaining one. These gains persist across context scales ranging from hundreds of thousands to trillions of tokens, and the gain holds across different LLM backbones.

Our analysis attributes this effectiveness to two core capabilities: _native tool proficiency_, which enables precise, executable interactions beyond natural-language queries, and _file system familiarity_, which provides strong inductive priors for navigating large text collections. These capabilities also help explain a surprising negative result: equipping coding agents with standard retrieval tools does not consistently improve performance. More interestingly, we observe _emergent, task-specific processing strategies_: agents autonomously develop iterative query refinement for multi-hop retrieval, programmatic aggregation for analytical tasks, and hybrid strategies for reading comprehension, all arising without explicit instruction or specialized training.

We hope the strong performance demonstrated in our work encourages a rethinking of simple, versatile approaches as backbone LLMs grow increasingly capable.

![Figure 3: Text Processing as File System Navigation. We organize text corpus into a Navigable File System of documents and folders. The Coding Agent explores this hierarchy using native tools (e.g., ripgrep, terminal commands), writes Python scripts for Programmatic Aggregation, and saves intermediate results. The agent Iteratively Refines its queries based on discovered information, enabling multi-hop reasoning without fixed retrieval pipelines.](https://arxiv.org/html/2603.20432v1/figures/main_figure.png)

---

## 2 Text Processing as File System Operation

Our approach reformulates long-context processing as a file system operation task, as illustrated in [Figure 3](https://arxiv.org/html/2603.20432v1#S1.F3). Rather than feeding massive text directly into a model’s context window or relying on semantic retrieval, we structure textual content as files within a directory hierarchy and delegate processing to off-the-shelf coding agents.

**Problem Setup.** Given a query $q$ and either a large corpus $\mathcal{C}=\{d_1, d_2, \ldots, d_n\}$ where $d_i$ is a piece of document, or a single long document $D$, the task is to produce an answer $a$ by reasoning over the provided corpus.

**Corpus Formatting.** For large corpus settings (corpus size $\gg$ 100M tokens), we format each document as an individual txt file and organize these files within a corpus directory. For NQ, since the corpus is prohibitively large, we store all documents in a single JSONL file. For single long-document settings in long-context QA tasks, we place the entire context in one txt file.

**Agent Interface.** The coding agent receives only the file or directory path along with the query. The agent then freely employs its native capabilities: executing terminal commands (e.g., grep and head), writing and running Python scripts for programmatic search and text processing, creating intermediate files to store partial results, and iteratively refining its exploration based on discovered information.

Crucially, we impose no constraints on how the agent processes the content. The agent autonomously decides whether to scan files sequentially, construct keyword searches, write custom parsing scripts, or combine multiple strategies. In some configurations, we additionally provide agents with access to a retrieval tool (BM25 or dense embeddings); however, even in these settings, the agent retains full autonomy over whether and how to use these tools. This stands in contrast to RAG pipelines with fixed retrieval stages or ReAct agents limited to predefined tool APIs. The complete prompts for all methods are provided in [Appendix A](https://arxiv.org/html/2603.20432v1#A1).

**Table 1: Main results across five benchmarks.** Best results are in bold. * indicates results evaluated on the full set, used here only for reference. Reported metrics are Accuracy for BrowseComp-Plus and LongBench, Exact Match (EM) for NQ, and Score for Oolong.

| Method | BrowseComp-Plus | Oolong-Syn | Oolong-Real | LongBench | NQ |
|---|---|---|---|---|---|
| Context Length | 750M | 536K | 385K | 188K | 3T |
| GPT-5 Full Context | 20.00 | 59.22 | 22.45 | 61.00 | 27.00 |
| RAG | 65.00 | 45.53 | 13.38 | 50.50 | 47.00 |
| ReAct Agent | 72.50 | 31.39 | 19.06 | 59.00 | 49.00 |
| RLM | – | 64.38 | 23.07 | 54.00 | 55.33 |
| Best Published | 80.00* | 64.38 | 24.09 | **63.30*** | 50.90* |
| **Coding Agents (Ours)** | | | | | |
| Codex (No Retriever) | **88.50** | **71.75** | 33.73 | 61.50 | **56.00** |
| Codex + Gemini Emb. | 84.00 | 68.03 | 32.40 | 61.50 | – |
| Codex + BM25 | 78.50 | 71.07 | 30.86 | 60.80 | 53.00 |
| Claude Code + BM25 | – | – | **37.46** | **62.50** | – |

---

## 3 Experiments

### 3.1 Benchmarks

- **BrowseComp-Plus** is a web browsing benchmark for evaluating Deep-Research agents on complex, multi-hop question answering. Built upon BrowseComp, BrowseComp-Plus provides a fixed corpus of 100K web documents and guarantees to contain the gold documents. The task requires agents to iteratively search and reason across multiple documents to locate hard-to-find, entangled information. For the evaluation, we employ an LLM-as-a-judge approach using GPT-5 to assess whether predicted answers match the ground truth, and report accuracy.

- **LongBench-v2** is a long-context benchmark designed to evaluate the ability of LLMs to perform deep understanding and complex reasoning across diverse real-world tasks. The benchmark adopts a multiple-choice question answering format and encompasses six task categories: single-document QA, multi-document QA, long in-context learning, long-dialogue history understanding, code repository understanding, and long structured data understanding. We report accuracy for these MCQs.

- **Oolong-Real and Oolong-Synthetic** are two variants from the Oolong benchmark designed for long-context reasoning. Oolong tasks require analyzing, synthesizing, and aggregating information distributed across entire documents to answer questions about patterns and distributions. Both variants test models’ ability to reason over large quantities of examples, perform in-context classification and counting, and handle temporal and user relations. We follow the scoring protocol described in the original paper for evaluation: questions requiring a label, date, user ID, or comparison are scored using exact match, and questions requiring a numerical answer are scored using $\texttt{score}(\hat{y})=0.75^{|y-\hat{y}|}$.

- **Natural Questions (NQ)** is a widely-used open-domain question answering benchmark. The task requires retrieving relevant passages from a large-scale Wikipedia corpus and extracting short factoid answers. We report exact match (EM) after normalization.

Due to computational cost, we randomly sample 200 examples from each benchmark, and we rerun all baselines with the same subset for fair comparisons.

### 3.2 Baselines

- **LLM full-context:** We evaluate the ability of GPT-5 to directly answer questions given full context. For BrowseComp-Plus and NQ, since the corpus is too large for the LLM to handle, we randomly sample documents from the corpus to form a 100k-token context. For LongBench and Oolong, we apply a sliding window strategy following prior work. For datapoints with context lengths greater than 200k tokens, we use a window size of 200k tokens with 50k overlaps. Answers and reasoning are produced from each window and then aggregated by the same LLM, which produces a final answer.

- **RAG:** We follow a standard RAG pipeline: retrieve the top 10 documents (for corpus-level tasks) or 300-word chunks (for long-document tasks), then generate the answer using GPT-5. We use Gemini embeddings for retrieval. We use BM25 for NQ due to its large corpus size.

- **ReAct-Style Search Agents:** We perform agentic search by placing the LLM in a ReAct loop. We provide GPT-5 with a Gemini embedding model as a retrieval tool. The LLM is shown the question and given access to “retrieve” and “get document” tools.

- **Recursive Language Model (RLM):** Recursive Language Models treat long input text as part of an external environment where LLMs can programmatically examine and recursively call themselves over text snippets using a Python REPL. We evaluate RLM using the exact setting described in the original paper. We exclude RLM from our BrowseComp-Plus evaluation because running it on the full 100k-document corpus is prohibitively time-consuming.

### 3.3 Coding Agent

We evaluate Codex v0.46.0 with GPT-5 as the base model under three configurations: (1) Native codex without any retriever, (2) Codex with BM25 as the retriever, and (3) Codex with dense retriever using Gemini embeddings as the encoder. We use the default system prompt in the first setting. We include instructions explaining how to use the retriever, along with the retriever’s Python implementation in the second and the third setting. For retrieval of Longbench and Oolong, we split documents into chunks of 300 words following prior work.

We additionally evaluate Claude Code with Sonnet 4.5 as the base model. The purpose of this experiment is to demonstrate that our findings are not specific to a single coding agent implementation. Claude Code represents an alternative frontier coding agent with distinct training and architecture from Codex. Due to budget constraints, we limit our Claude Code evaluation to two benchmarks: Oolong-Real and LongBench.

**Table 2: Ablation study on file system structure.** We test Codex with GPT-5 on BrowseComp-Plus.

| Retriever | Folder Structure | Single File |
|---|---|---|
| No Retriever | 89.0 | 83.0 |
| Gemini Emb. | 90.0 | 86.0 |
| BM25 | 82.0 | 82.0 |

---

## 4 Main Results

Coding Agents Establish New State-of-the-Art

Off-the-shelf coding agents significantly outperform all baselines across diverse benchmarks. Notably, these gains hold across vastly different context scales, from average 188K tokens (LongBench) to over three trillion tokens (NQ), demonstrating that coding agents provide a robust, general-purpose solution for long-context processing without task-specific training or architectural modifications.

Although GPT-5 full-context sees only a very small fraction of the corpus in its context window on large-corpus tasks, its non-trivial accuracy (20.0% on BrowseComp-Plus, 27.0% on NQ) is likely due to data contamination. On Oolong, GPT-5’s scores are substantially lower than reported in the original paper, which evaluated only on datapoints with context lengths under 200K tokens; while our sample includes much longer contexts where performance degrades significantly. We also note that the original RLM paper evaluates on the trec_coarse subset of Oolong-Synthetic rather than the full dataset.

![Figure 4: Two emergent processing strategies. Left: BrowseComp-Plus—iterative query refinement with entity chaining across searches. Right: Oolong-Synthetic—programmatic aggregation via Python scripts with regex patterns.](https://arxiv.org/html/2603.20432v1/figures/strategy_characterization_single.png)

---

## 5 Ablations and Analysis

In this section, we conduct detailed ablation studies to identify the key factors contributing to the effectiveness of coding agents in long-context processing, along with an in-depth analysis of their emergent behaviors.

### 5.1 File System Structure Matters

We hypothesize that coding agents benefit from file system familiarity, the ability to leverage directory structures acquired through training on code repositories. To test this hypothesis, we conduct an ablation study comparing two corpus organization strategies on a 100-example subset of BrowseComp-Plus.

- **Folder Structure:** Documents are organized as individual files within a directory hierarchy, mirroring the structure of typical code repositories.
- **Single File:** The directory structure is eliminated, and the corpus is stored as a single JSON dictionary where document ids serve as keys. This allows the retriever to directly output the relevant document id.

As shown in Table 2, the folder structure outperforms the single file configuration across retriever settings.

**Table 3: Analysis of average command usage counts on the BrowseComp-Plus dataset (No Retriever).**

| Structure | Search (rg) | Extract (sed) | Index (nl) |
|---|---|---|---|
| Single File | 31.05 | 0.61 | 0.69 |
| Folder | 18.46 | 4.48 | 1.22 |
| Diff (%) | -40.5% | +634% | +76.8% |

With folder structure, agents employ coordinate-based reading, using `nl` (number lines) to index content and `sed` to extract specific line ranges. The usage of `sed` increases by over seven times in the folder setting, indicating that agents selectively read relevant context rather than consuming entire files. This "index and slice" strategy effectively builds a coordinate system (file + line number) that enables more accurate data extraction. In contrast, without navigable structure, agents fall into repeated discovery loops. The higher usage of `rg` suggests that agents struggle to isolate information and must rely on expensive corpus-wide scans.

### 5.2 Retrieval Tools Do Not Uniformly Improve Performance

Our main results reveal a counterintuitive finding: equipping coding agents with retrieval tools does not consistently improve performance and can even degrade it. To better understand this phenomenon, we analyze agent behavior across retriever configurations on BrowseComp-Plus. For each trajectory, we count native search commands measured by the number of shell commands invoking search utilities (grep, ripgrep, find, etc.) that do not involve the provided retriever.

**Table 4: Agent exploration patterns across retriever configurations on BrowseComp-Plus.**

| Configuration | Native Search |
|---|---|
| No Retriever | 14.92 |
| BM25 | 9.84 |
| Gemini Emb. | 8.33 |

Agents without retrieval tools issue substantially more native search commands compared to retriever-augmented variants. This difference reveals a behavioral shift: when provided with a retriever, agents reduce their use of native exploration tools such as grep.

Counterintuitively, equipping agents with IR tools does not guarantee improved performance. We hypothesize that standard retrievers, when available, become the agent’s default discovery mechanism and displace the broader file-system exploration strategies that agents otherwise employ autonomously. Since retrieval ranking is imperfect, this substitution can cause agents to miss relevant context. The precise mechanism remains an open question we leave to future work.

### 5.3 Emergent Task-Specific Processing Strategies

![Figure 5: Quantitative characterization of agent strategies per query. The y-axis represents the normalized proportion of each metric, where the values for a given model sum to 1 across all datasets.](https://arxiv.org/html/2603.20432v1/figures/strategy_characterization_single.png)

A key advantage of coding agents over fixed-pipeline approaches is their ability to adapt processing strategies to task requirements. We analyze agent trajectories across benchmarks and identify distinct behavioral patterns that emerge in response to different tasks. To validate that coding agents dynamically adapt their strategies to different task types, we track Search Intensity (using search commands such as grep or find), Read Volume (number of tokens the agent reads from documents), and Code Volume (number of Python functions generated). We conduct this ablation study on codex without a retriever to eliminate confounding factors. For RLM, we count search intensity by identifying code blocks that scan files with regex patterns, excluding regex used for computation such as data processing and aggregation.

- **Iterative Query Refinement for Multi-Hop Retrieval:** On BrowseComp-Plus, which requires multi-hop reasoning across a large corpus, agents exhibit an iterative search-and-refine pattern. The agent usually begins with an initial search based on entities or concepts in the question, examines the retrieved documents, extracts new entities or relationships, and formulates refined queries targeting the next reasoning hop. Critically, this behavior emerges without explicit instruction.

  [Figure 4](https://arxiv.org/html/2603.20432v1#S4.F4) (left) illustrates this pattern on a representative example. The task requires finding a professional gamer satisfying multiple constraints linked through a chain of entities. The agent begins by searching for game developers founded in the specified time range, discovering Brandon Beck as a Riot Games co-founder. It then refines its query to search for Beck’s spouse, discovering Natasha Beck. Subsequent searches verify her credentials and trace the chain back to Valorant professional players, ultimately identifying Max Mazanov as the answer. This six-hop reasoning chain—Riot Games → Brandon Beck → Natasha Beck → Pepperdine → Valorant → Demon1 → Max Mazanov—emerges entirely from the agent’s autonomous query refinement, with each search informed by entities discovered in previous steps.

  This example reflects a broader pattern we observe quantitatively across the benchmark. As shown in [Figure 5](https://arxiv.org/html/2603.20432v1#S5.F5), the agent prioritizes discovery over generation: BrowseComp-Plus elicits the highest Search Intensity. The agent relies primarily on an iterative loop of native search commands to locate and read relevant files.

- **Programmatic Aggregation for Analytical Tasks:** Oolong tasks require analyzing, synthesizing, and aggregating information distributed across entire documents. As shown in [Figure 5](https://arxiv.org/html/2603.20432v1#S5.F5), on analytical tasks requiring aggregation (e.g., counting, sorting), the agent abandons search in favor of code generation. Both Oolong tasks show a dramatic drop in reading but a substantial spike in Code Volume.

  [Figure 4](https://arxiv.org/html/2603.20432v1#S4.F4) (right) demonstrates this strategy on a task requiring the agent to identify which user has the most "contradiction" labels across 1,772 sentence pairs, without labels being provided. The agent writes a Python script that: (1) parses the document structure to extract user IDs and sentence pairs, (2) implements a rule-based NLI classifier using regex patterns to detect negation (no, not, never) and quantity mismatches (only one vs. series of), (3) executes the classifier over all pairs, and (4) aggregates results by user. When initial patterns miss edge cases, the agent examines intermediate outputs, expands its pattern set, and re-executes. This is an iterative refinement loop applied to code rather than queries. This approach leverages the agent’s native proficiency with text processing tools. We provide concrete examples and analysis of agent-generated scripts in [Appendix B](https://arxiv.org/html/2603.20432v1#A2).

- **Direct Inference for Diverse Long-Context Tasks:** LongBench presents a diverse mixture of long-context challenges that resist any single processing paradigm. It contains single-document and multi-document question answering, summarization, few-shot learning, synthetic retrieval, and code completion. As shown in [Figure 5](https://arxiv.org/html/2603.20432v1#S5.F5), the coding agent has relatively low overall tool usage: modest Search Intensity, very low Read Volume, and near-zero Code Volume. This differs from the heavily read-dominated pattern on BrowseComp, the search-dominated pattern on NQ, and the code-dominated pattern on Oolong. Specifically, the near-zero Code Volume indicate that programmatic data processing is largely unnecessary for LongBench. Instead, the most effective strategy is to rely directly on the LLM’s inherent long-context reasoning abilities. Consistent with this behavioral profile, our results in Table 1 demonstrate that the agent’s performance is nearly identical to the baseline performance of the LLM provided with the full context.

These emergent patterns demonstrate that coding agents function as generalizable long-context processors that dynamically adjust their approach based on task demands. In contrast, ReAct agents are limited to a fixed action space defined by their tool APIs, and RLMs impose a uniform recursive decomposition strategy regardless of task structure. Coding agents face no such constraints. As shown in [Figure 5](https://arxiv.org/html/2603.20432v1#S5.F5), agents employ markedly different tools and strategies across tasks: leveraging search utilities for retrieval-heavy benchmarks, custom scripts for aggregation tasks, and hybrid approaches for reading comprehension.

**Table 5: Average cost per query across benchmarks.**

| Method | BrowseComp-Plus | Oolong-Syn | Oolong-Real | LongBench | NQ |
|---|---|---|---|---|---|
| GPT-5 Full Context | $0.275 | $1.421 | $0.770 | $0.432 | $0.129 |
| RAG | $0.111 | $0.045 | $0.026 | $0.024 | $0.006 |
| ReAct Agent | $0.237 | $0.092 | $0.168 | $0.056 | $0.027 |
| RLM | – | $0.920 | $0.094 | $0.360 | $0.630 |
| Codex (No Retriever) | $0.703 | $0.194 | $0.419 | $0.128 | $0.111 |
| Codex + Gemini Emb. | $0.628 | $0.149 | $0.371 | $0.129 | – |
| Codex + BM25 | $0.828 | $0.161 | $0.368 | $0.124 | $0.094 |
| Claude Code + BM25 | – | – | $0.380 | $0.319 | – |

### 5.4 Cost Analysis

Coding agents incur higher costs than lightweight baselines such as RAG, but remain competitive with or cheaper than other strong methods while delivering substantially superior performance.

---

## 6 Related Work

- **Long-Context Language Models (LCLM):** Recent advances have dramatically expanded the context windows of frontier models. This scaling has enabled direct processing of long documents. However, prior work has shown substantial performance degradation as context length increases, with models often losing much of their short-context capability well before reaching advertised limits. Additionally, inference cost of LCLMs scales linearly with context length, making very long contexts computationally expensive. These findings motivate our exploration of alternatives to context window scaling.

- **Agentic RAG:** Traditional RAG methods retrieve relevant passages using a fixed pipeline, typically dense retrieval followed by answer generation, which limits their ability to handle queries requiring iterative refinement or multi-hop reasoning. Agentic RAG approaches address this limitation by allowing models to dynamically reformulate queries and iteratively search based on intermediate findings. However, existing agentic RAG systems are predominantly trained for specialized web search or open-domain QA tasks, requiring task-specific fine-tuning or reinforcement learning to learn effective search strategies. Our work demonstrates that off-the-shelf coding agents without any task-specific training are already capable agentic searchers.

- **Agent with Long-Term Memory:** A growing body of work has focused on building and optimizing memory-centric agentic architectures through various memory manipulation strategies. This line of work is orthogonal to ours: rather than storing context in the agent’s memory, we place it in the environment as files that the agent can interact with.

- **Recursive Language Models (RLM):** Closest and concurrent to our work, RLMs propose treating long input text as part of an external environment where LLMs can programmatically examine, decompose, and recursively call themselves over snippets of the text using a Python REPL. The key distinction lies in how agents interact with this environment: RLMs employ a specialized system prompt that instructs models to decompose problems through recursive LLM sub-calls over text segments, whereas we use off-the-shelf coding agents with no task-specific prompting. Our agents instead leverage native file system tools (e.g., grep, sed) and custom scripts for exploration and aggregation.

- **Coding Agents:** Prior work has demonstrated that incorporating coding data during fine-tuning improves LLM reasoning capabilities. Some work equips LLMs with code execution to solve complex reasoning tasks. However, these agents perform poorly on long-context processing tasks. Another line of work trains or builds coding agents for software engineering tasks involving large codebases. These agents are designed and evaluated for long-horizon coding tasks rather than general text processing.

---

## 7 Conclusion and Future Work

We have demonstrated that off-the-shelf coding agents provide an effective paradigm for long-context processing, achieving state-of-the-art results on four out of five benchmarks spanning context lengths from 188K to three trillion tokens. By reformulating long-context tasks as file system navigation problems, coding agents can leverage their native capabilities, terminal commands, programmatic search, and iterative script refinement, to process massive text corpora without task-specific training or architectural modifications.

Our analysis reveals two key factors underlying this effectiveness: _native tool proficiency_, which enables precise, executable interactions that go beyond natural language retrieval queries, and _file system familiarity_, which provides strong inductive priors for navigating hierarchically organized text. We further observe that coding agents autonomously develop task-appropriate strategies, including iterative query refinement for multi-hop reasoning, programmatic aggregation for analytical tasks, and hybrid approaches for reading comprehension.

These findings suggest that increasingly capable foundation models for software engineering reduce the distinction between coding and general text processing tasks. Rather than relying on specialized architectures for long-context understanding, our results show that structuring text in formats aligned with code can be sufficient for effective reasoning over extended contexts.

Our approach has several limitations that suggest directions for future work. First, our analysis reveals that naively providing retrieval tools may degrade performance; future work should investigate how to better integrate retrieval capabilities without suppressing agents’ native exploration. Second, while off-the-shelf coding agents transfer surprisingly well to text processing tasks, they are primarily aligned and optimized for coding rather than long-context reasoning.

An important direction for future work is developing frameworks that specialize these agents for navigating and reasoning over massive text corpora.

---

## Impact Statement

This paper presents work whose goal is to advance the field of Machine Learning. There are many potential societal consequences of our work, none which we feel must be specifically highlighted here.

---

## Acknowledgments

This work is partially supported by the Learning Engineering Virtual Institute, funded by leading education philanthropists and organizations through Grant G-23-2137070 to the University of Florida and its partner institutions. This work is also supported by Google.org, the Google Cloud Research Credits program for the Gemini Academic Program, and Amazon AGI Labs SF.

---

## References

(References omitted for brevity; see original for full list.)

---

## Appendix A Prompts

This section provides the complete prompts used for all methods evaluated in our experiments. We organize prompts by method type and benchmark. Variables in curly braces (e.g., {question}, {context_location}) are replaced with actual values at runtime.

### A.1 Coding Agent Prompts

We present prompts for our coding agent approach under two configurations: (1) without retriever access, where agents rely entirely on native file system exploration, and (2) with retriever access, where agents can optionally use a retrieval tool alongside their native capabilities.

#### A.1.1 Without Retriever

#### A.1.2 With Retriever

When equipped with a retriever, the coding agent receives additional instructions explaining how to invoke the retrieval tool. The {embedding_model} parameter is set to either BM25 or Gemini Emb. depending on the retriever configuration.

### A.2 ReAct-Style Search Agent Prompts

The ReAct agent is provided with two tools: retriever for searching the corpus using semantic embeddings, and get_document for retrieving the full content of a specific document. The agent performs step-by-step reasoning interleaved with tool calls.

### A.3 Full-Context LLM Prompts

For the full-context baseline, we provide the entire context (or a sampled/windowed portion for very large corpora) directly in the prompt. The model must answer based solely on the provided context without any tool access.

### A.4 Prompt Design Rationale

Our prompt design reflects several key principles:

- Minimal instruction for coding agents. We deliberately keep coding agent prompts simple, providing only the task description and file location. This allows agents to leverage their native capabilities for file system navigation and text processing without constraining their approach. The contrast with retriever-augmented prompts (which include explicit tool instructions) enables us to study how tool availability affects agent behavior.
- Task-specific output formatting. Each prompt includes output format instructions appropriate to the benchmark’s evaluation protocol. LongBench uses multiple-choice format, Oolong requires exact numerical or categorical answers, and open-domain QA benchmarks expect short factoid responses.
- Consistent structure across methods. While the available tools differ across methods (file system access for coding agents, retrieval tools for ReAct agents, none for full-context), we maintain consistent task descriptions to enable fair comparison of the underlying approaches rather than prompt engineering differences.

---

## Appendix B Case Studies: Agent-Generated Scripts

We present example Python scripts autonomously written by Claude Code when solving Oolong benchmark tasks. These examples illustrate the _programmatic aggregation_ strategy discussed in [subsection 5.3](https://arxiv.org/html/2603.20432v1#S5.SS3), where agents write custom code to analyze, count, and aggregate information distributed across long documents.

### B.1 Example 1: Counting Dice Rolls

**Task:** Given a transcript of a tabletop role-playing game (Critical Role), count the number of dice rolls with a specific value and compute the percentage.

**Analysis:** The agent identifies that this task requires aggregating information scattered throughout a long transcript. Rather than attempting retrieval (which would miss many instances), the agent writes a Python script that: (1) locates episode boundaries using marker tags, (2) identifies player dialogue lines by speaker prefixes, (3) applies multiple regex patterns to capture various roll announcement formats (e.g., “rolled a 15”, “Natural 20”, or standalone numbers), and (4) computes statistics over all extracted values.

### B.2 Example 2: Tracking Character Actions Across Episodes

**Task:** Identify the last spell cast by a specific character (Vax’ildan) in each episode of a multi-episode transcript.

**Analysis:** This task requires tracking character-specific actions across multiple episodes within a single long document. The agent constructs a structured approach: (1) parse the document into separate episodes using boundary markers, (2) filter lines to those involving the target character (by speaker name or character mentions), (3) identify spell-related content using keyword matching, (4) extract spell names using regex patterns and a predefined spell list, and (5) report the last occurrence per episode.

### B.3 Key Observations

These examples demonstrate several characteristics of the coding agent’s approach:

1. **Structured parsing:** The agent recognizes and leverages document structure (episode markers, speaker prefixes) rather than treating the text as unstructured.
2. **Robust pattern matching:** Multiple regex patterns handle variations in how information is expressed (e.g., “rolled 15” vs. “rolled a fifteen” vs. “Natural 20”).
3. **Programmatic aggregation:** Instead of retrieving a few relevant passages, the agent processes the entire document systematically to ensure complete coverage.
4. **Domain adaptation:** The agent incorporates domain knowledge (player names, spell lists, D&D conventions) into its parsing logic.

These behaviors emerge without explicit instruction, demonstrating how coding agents transfer software 