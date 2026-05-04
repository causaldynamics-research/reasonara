# **Cielara Reasonara**

Graph-Structured Causal Memory for Agentic Systems

*Engineering an AI Memory System for the 500-Million Token Context Era*

Executive Summary — May 2026

*Bowen Zhu • Dr. Xuchao Zhang • Dr. Liang Zhao • Hasibul Haque — Causal Dynamics Lab*

## **The Problem: Context Rot Is Killing Enterprise AI**

95% of enterprise AI pilots are failing. According to MIT Media Lab’s [NANDA initiative](https://mlq.ai/media/quarterly_decks/v0.1_State_of_AI_in_Business_2025_Report.pdf) (The GenAI Divide: State of AI in Business 2025), enterprises consistently underestimate the challenge of making their own data ready for AI.1

[Harvard Business Review](https://hbr.org/2026/02/when-every-company-can-use-the-same-ai-models-context-becomes-a-competitive-advantage) put it plainly: “When every company can use the same AI models, context becomes a competitive advantage.” The question isn’t who has better models. The winner is who has the better memory.2

Anthropic’s March 2026 report, “[Labor market impacts of AI](https://www.anthropic.com/research/labor-market-impacts),” found that in Computer & Math roles, LLMs could theoretically automate 57% of tasks, but less than 2% of actual work was affected. The gap between 57% potential and 2% reality is largely a data infrastructure problem.3

Context Rot shows up in three recurring failure modes:

- **Cross-document confusion.** Single-vector retrieval conflates passages from different sources. When asked “What programming language does the billing service use?” a RAG system might retrieve documentation from three different services and blend the answers.
- **Shallow retrieval leading to hallucination.** When retrieved chunks are topically related but factually wrong, the LLM generates confident, sourced answers that are still incorrect. LlamaIndex’s own benchmarks show this pattern in 15–20% of complex enterprise queries.
- **Complete retrieval misses.** For some queries, the top-k chunks simply don’t contain the answer. LangChain responded with LangGraph, which helps with control flow but still depends on the same vector retrieval underneath.

The bottleneck isn’t intelligence, but memory infrastructure. [Gartner’s February 2026](https://www.gartner.com/doc/reprints?id=1-2JHTJ0FC) report confirms this: context graphs, structured memory layers that sit between raw data and AI models, are “the new essential infrastructure for agentic systems.”4

## **Results at a Glance**

Before diving into how Cielara’s Reasonara works, here is what it delivers:


| Metric                                      | Result                                                                  |
| ------------------------------------------- | ----------------------------------------------------------------------- |
| UltraDomain accuracy (125M+ tokens context) | **94%** (+20pp vs. frontier LLMs, 10pp vs. RAG) — **SOTA**              |
| LoCoMo (conversational memory)              | **93.3%** LLM-Judge (vs. 82.5% full context, 65.3% Mem0) — **New SOTA** |
| LoCoMo-Plus (cognitive memory)              | **72.82%** (vs. 26.06% Gemini-2.5-Pro, 21.05% GPT-4.1)                  |
| LongMemEval (115k-token contexts)           | **87.4%** (vs. 74.6% Nemori, 71.2% Zep GPT-4o) — **New SOTA**           |
| Speed vs. Codex High Reasoning              | **5–8× faster** (18.7s vs. 97–150s per question)                        |
| Token reduction vs. full context            | **98% fewer tokens** (1K–2.5K per query)                                |
| Scale tested                                | **125M+ tokens context today, 500M+ on roadmap**                        |


**Reasonara achieves the best accuracy-per-token ratio across the entire field of 20+ agent memory systems. This is the only system that explicitly measures and addresses cognitive memory, the class of retrieval problems where the right answer depends on latent constraints rather than direct factual recall.**

## **What Reasonara Actually Is**

Reasonara is a graph-structured causal memory system. It ingests heterogeneous enterprise data including code, documents, conversations, execution traces, and constructs a persistent, structured memory that AI agents query through a trained retrieval policy. Instead of treating enterprise knowledge as a bag of text chunks, Reasonara autonomously builds an organized memory where causal relationships, decision logic, and domain structure are explicitly represented and navigable.

The name combines “reason” with “ara,” reflecting the system’s core function: structured reasoning over enterprise knowledge.

### **Self-Describing Memory Representation**

The central breakthrough is representational. Every memory in Reasonara is a self-describing unit: a tuple of (abstraction, value). The abstraction is a named concept in natural language that serves as both the organizational identity and the retrieval entry point. The value preserves the full detailed content without lossy compression.

This is fundamentally different from every prior approach:


| System                      | Organization Model                     | Flexibility                   |
| --------------------------- | -------------------------------------- | ----------------------------- |
| Reasonara                   | Self-organizing per-entry abstractions | High — adapts to content      |
| RAG (LangChain, LlamaIndex) | Flat bag of chunks                     | None — fragments related info |
| Mem0, A-MEM                 | Flat key-value pairs                   | Low — no grouping mechanism   |
| Zep, MAGMA                  | Predefined graph schemas               | Low — rigid edge types        |
| Nemori                      | Fixed episodic / semantic split        | Low — can’t adjust boundary   |
| LiCoMemory, PISA            | Fixed-depth hierarchies                | Medium — predetermined levels |


Reasonara is the only system where memory organization emerges from the content itself rather than being imposed by design choices. The system identifies the natural abstraction level for each piece of content and uses it as the organizational unit. A meeting about budget constraints is abstracted as "Q3 budget planning"; a user preference becomes "dietary restrictions." In each case, the granularity is determined by the content, not by a predefined schema

### **Retrieval Anchors: Schema-Free Retrieval**

For each memory, Reasonara generates multiple retrieval anchors: short phrases that expose retrieval facets not captured by the abstraction alone. Retrieval anchors are schema-free. They require no predefined edge types, no graph schemas, no fixed taxonomies. They create a flexible associative fabric where connections emerge from content.

Seven anchor types, all stored as strings in a single unified index:


| Anchor Type | Purpose                                   | Example                                           |
| ----------- | ----------------------------------------- | ------------------------------------------------- |
| Entity      | Direct lookup by people / places / things | “Alice Chen”, “Stripe API”                        |
| Aspect      | Topical matching                          | “dietary restriction”, “budget planning”          |
| Cognitive   | Anticipate future query contexts          | “restaurant recommendation” for an allergy memory |
| Provenance  | Source filtering                          | “source:health-intake-form”                       |
| Temporal    | Time-based reasoning                      | “since childhood”, “diagnosed 2020”               |
| Relational  | Cross-references between memories         | “related:user-dietary-preferences”                |
| Cache       | High-frequency retrieval shortcuts        | “profile:dietary”                                 |


The most significant advancement is the cognitive anchor, which solves contextual isolation, the hardest retrieval problem in the field. When a memory carries a cognitive anchor for a query context that is semantically distant from the memory’s content, that memory still surfaces when needed.

**Consider a concrete enterprise scenario:** an internal AI agent is asked to recommend high-impact case studies for an upcoming industry conference. It identifies Project Titan as the strongest candidate. But Project Titan is governed by an NDA that restricts public disclosure of certain technical details. The query “which case studies should we feature at the conference?” has zero semantic overlap with “NDA constraint on Project Titan.” A traditional retrieval system never surfaces the constraint, the agent makes the recommendation, and a presenter unknowingly walks into a compliance violation.

Cognitive anchors prevent this. When the Project Titan memory is constructed, Reasonara runs Implication Chain Generation. It reasons forward from the memory to anticipate the future query contexts in which the memory should be retrieved from artifacts like public communications, conference materials, marketing copy, and sales decks. Then it writes those contexts as cognitive anchors. The constraint is now reachable from the query that needs it, even though the surface text doesn’t match.

Cognitive anchors are constructed in advance, not inferred at query time. This is what makes cognitive retrieval tractable in production: the reasoning cost is paid once at write time, and retrieval remains a fast lookup.

### **Cognitive Alignment at Retrieval Time**

When a query arrives, Reasonara doesn’t simply match anchors by surface similarity. The retrieval policy scans the cognitive anchor inventory along several alignment dimensions, surfacing memories whose connection to the query is logical rather than lexical:

- **Direct overlap:** the memory and the query refer to the same concept or entity.
- **Cause:** the memory describes a condition or decision that explains why the current situation exists.
- **Consequence:** the memory describes an outcome, obligation, or downstream effect that the current action would trigger or violate.
- **Contradiction:** the memory contains a constraint, prior commitment, or fact that is in tension with the apparent answer.

The NDA case is a contradiction signal. A user’s prior commitment to a deadline is a consequence signal. A regulatory rule that explains why a certain workflow exists is a cause signal. By making these alignment dimensions explicit, the retrieval policy can pull in latent constraints that a similarity-based retriever would miss entirely.

### **Self-Organizing Consolidation**

Reasonara’s create-or-update consolidation prevents both fragmentation (the curse of flat systems like RAG, Mem0, A-MEM, where related information scatters across disconnected entries) and drift (the risk of wrong merges corrupting memory). When new information matches an existing concept, that information merges under the abstraction; when genuinely new, a fresh entry is created. The result: 100 raw segments about “Alice’s career” consolidate into 1 rich, coherent entry, producing a 3–10× smaller search space.

### **Policy-Based Retrieval**

Reasonara models retrieval as a Markov Decision Process. The retriever maintains a state vector (current query embedding  retrieved context  frontier of unexplored neighbors) and selects actions such as REFINE the query, EXPAND to anchor-linked neighbors, or STOP and answer. The policy is trained via GRPO (Group Relative Policy Optimization) on downstream answer quality.

What this means practically: for simple queries, the policy stops early (1 step, low cost). For complex multi-hop queries, it expands through the implicit memory graph across multiple steps. The depth/breadth trade-off is learned, not hand-tuned.

## **Why Cognitive Memory Is the Hard Problem**

Most memory benchmarks measure recall: the user asks an explicit factual question, and the system either returns the right fragment or it doesn’t. “Which vendor did we approve for the Q3 rollout?” “What was the agreed SLA on the integration project?” These questions have a fact in the corpus, and the retrieval task is to find it.

Cognitive memory is a different class of problem. The right answer depends on a latent constraint like a goal, a state, a value, a prior commitment, or a piece of causal context which the current query never explicitly mentions. The Project Titan NDA case is the canonical example: the user asks about conference case studies, and the system has to recognize that an unstated constraint should shape the response. Cognitive memory is what separates an assistant that answers questions from an assistant that behaves consistently with what it knows about the user, the organization, and the situation.

This class of problem is invisible to standard memory benchmarks, which is why most existing systems quietly fail at it. To measure cognitive memory directly, Reasonara is evaluated on LoCoMo-Plus, a benchmark explicitly designed to test whether a system can recover and apply latent constraints rather than simply recall stated facts.

The results are decisive:


| System                                          | LoCoMo-Plus Score | Gap to Reasonara |
| ----------------------------------------------- | ----------------- | ---------------- |
| Reasonara (GPT-5.4)                             | 72.82%            | —                |
| Reasonara (GPT-5.4-mini)                        | 62.34%            | –10.48           |
| Gemini-2.5-Pro (frontier LLM, no memory system) | 26.06%            | –46.76           |
| GPT-4.1 (frontier LLM, no memory system)        | 21.05%            | –51.77           |
| Mem0                                            | 15.80%            | –57.02           |
| A-MEM                                           | 17.20%            | –55.62           |


The headline number is the 46.76-point absolute gap (about a 179% relative improvement) between Reasonara and the strongest non-Reasonara baseline. But the more important comparison is internal: Reasonara with GPT-5.4-mini, a smaller model, still beats Gemini-2.5-Pro by 36.28 points. The gain is coming from the memory architecture, not from the size of the underlying language model. A weaker model with the right memory layer outperforms a frontier model without one.

This is the practical case for cognitive memory infrastructure. Buyers comparing Reasonara to a frontier-LLM-only deployment are not comparing two systems on the same axis. They are comparing a system that can surface latent constraints to one that, by construction, cannot. In regulated industries where the cost of an unsurfaced NDA, a missed compliance rule, or an unknown prior commitment is measured in legal exposure. That difference is the entire point of deploying memory infrastructure in the first place.

## **Architecture: Four Layers**

![Reasonara architecture](assets/architecture.png)

### **Layer 1: Ingestion & Normalization**

Collects inputs from multiple source types (code repositories, documents, conversations, execution traces) and converts them to a unified record format with explicit provenance metadata.

### **Layer 2: Segmentation & Organization**

Decomposes inputs into atomic, addressable units using modality-specific rules: functions for code, sections for documents, turns for conversations. Each segment becomes the input for memory construction.

### **Layer 3: Structured Memory (the Core)**

This is where the self-describing memory representation lives. Reasonara automatically extracts domain ontologies—entities, relationships, decision logic, and procedural knowledge—and organizes them into a typed, directed acyclic graph. Nodes represent semantic units at multiple granularity levels; edges encode causal, temporal, and compositional relationships.

The data structure: each node is an (abstraction, value) pair with associated retrieval anchors. The abstraction provides the navigational handle; the value preserves full detail; the anchors create the associative retrieval surface. New information is consolidated via create-or-update: match against existing abstractions, merge if conceptually aligned, create new entry otherwise.

### **Layer 4: Policy-Driven Retrieval**

The retrieval engine is formulated as an MDP. The retrieval pipeline combines three complementary signals: semantic search (dense embeddings over abstractions), BM25 keyword search, and implicit memory graph traversal. The trained policy decides how to combine and extend these signals based on query complexity.

At query time, the policy first reformulates the search text, decomposes the query when needed, and optionally expands it toward new retrieval directions rather than simple paraphrases. It then scans the anchor inventory for cognitive alignment along the dimensions described earlier (direct overlap, cause, consequence, contradiction). The selected anchors resolve to memories whose values are assembled into a compact working context, which is what the downstream model actually sees.

## **Benchmark Results**

### **UltraDomain: Enterprise-Scale Retrieval**

Reasonara was benchmarked against UltraDomain, a dataset of high-density documentation spanning Technology, Computer Science, Legal, Financial, and Medical domains.

Reasonara achieved 94% accuracy across all five domains, outperforming OpenAI Codex with high reasoning (89.7%) while being 5–8× faster.

In the Technology domain, Reasonara achieved a 25:1 win ratio against LangChain on per-question disagreements: 25 questions where only Reasonara was correct vs. 1 where only LangChain was correct.

### **LoCoMo: Long-Context Conversational Memory**

On LoCoMo, Reasonara achieves a 0.933 LLM-judge score with GPT-5.4 and 0.898 with GPT-5.4-mini, establishing a new state of the art and improving on the strongest prior memory system (MEMORA, 0.863) by 7.0 absolute points.


| System                   | LoCoMo Score | Gap to Reasonara |
| ------------------------ | ------------ | ---------------- |
| Reasonara (GPT-5.4)      | 0.933        | —                |
| Reasonara (GPT-5.4-mini) | 0.898        | –0.035           |
| MEMORA                   | 0.863        | –0.070           |
| Full Context             | 0.825        | –0.108           |
| Nemori (gpt-4.1-mini)    | 0.794        | –0.139           |
| LangMem                  | 0.734        | –0.199           |
| Mem0                     | 0.653        | –0.280           |
| RAG                      | 0.633        | –0.300           |
| Zep                      | 0.616        | –0.317           |


The gains are concentrated in the harder reasoning categories—which is also where enterprise queries actually live. Single-hop factual recall is the easy case; the value of structured memory shows up in multi-hop reasoning, temporal reasoning, and open-domain synthesis.


| Question Type | Reasonara (GPT-5.4) | MEMORA | Absolute Gain |
| ------------- | ------------------- | ------ | ------------- |
| Multi-hop     | 0.967               | 0.787  | 0.180         |
| Open-domain   | 0.781               | 0.594  | 0.187         |
| Temporal      | 0.907               | 0.866  | 0.041         |
| Single-hop    | 0.908               | 0.918  | –0.010        |


Multi-hop and open-domain are the categories where evidence is dispersed across the corpus and the answer requires integrating multiple memories. The 18-point gains in those categories are the empirical signature of structured memory doing real work.

### **LongMemEval**

On LongMemEval, Reasonara scores 87.4% accuracy, outperforming Nemori (74.6%), LiCoMemory (73.8%), Zep  GPT-4o (71.2%), and Full Context (65.6%).

### **Token Efficiency**

Reasonara achieves 98% token reduction vs. full-context processing, delivering 1,000–2,500 tokens per query compared to 23,000–115,000 for full context. This is the empirical realization of the consolidation ratio—100 raw segments about one topic become 1 coherent entry.

### **Feature Comparison Matrix**

Across key capability dimensions evaluated against the field:


| Feature                               | Reasonara | Mem0 | Nemori | MAGMA | Zep | LangChain |
| ------------------------------------- | --------- | ---- | ------ | ----- | --- | --------- |
| Explicit per-entry abstraction        | ✓         | ✗    | ✓      | ✗     | ✗   | ✗         |
| Customizable granularity              | ✓         | ✗    | ✗      | ✗     | ✗   | ✗         |
| Schema-free organization              | ✓         | ✓    | ✓      | ✗     | ✗   | ✗         |
| Self-organizing consolidation         | ✓         | ✗    | ✓      | ◐     | ◐   | ✗         |
| Multi-hop retrieval                   | ✓         | ✗    | ✗      | ✓     | ✓   | ✗         |
| Trainable retrieval policy            | ✓         | ✗    | ✗      | ✗     | ✗   | ✗         |
| Cognitive memory (latent constraints) | ✓         | ✗    | ✗      | ✗     | ✗   | ✗         |
| Detail preservation                   | ✓         | ◐    | ◐      | ✓     | ✓   | ✓         |
| Generalizes RAG KG                    | ✓         | ✗    | ✗      | ✗     | ✗   | ✗         |


### **Ablation: Every Component Matters**

Systematic ablation studies confirm each retrieval component contributes independently:

- Retrieval anchor search is the single most impactful component. Removing it drops accuracy by up to 4 pp.
- Semantic search removal costs 2 pp.
- BM25 keyword search removal independently hurts accuracy.
- Agentic query tools (refinement and decomposition) each account for 1.2% accuracy.
- Even the worst ablation variant (89–92%) still beats LangChain (84–87%) by 5–8 points.

## **Competitive Landscape**

### **Microsoft GraphRAG**

Microsoft open-sourced GraphRAG in mid-2024 and has the distribution advantage of Azure integration.

- **Architecture:** GraphRAG builds a static community-based knowledge graph using LLM-generated summaries at indexing time. Reasonara builds a dynamic, query-responsive causal graph with trained retrieval policies. GraphRAG’s graph is fixed once built; Reasonara evolves with new data through create-or-update consolidation.
- **Scale:** GraphRAG’s indexing cost scales linearly with LLM calls (every chunk requires summarization). At 125M+ tokens, this becomes prohibitive. Reasonara’s segmentation is deterministic (no LLM needed), with LLM calls only for abstraction and retrieval anchor generation.
- **Retrieval:** GraphRAG uses community-level summaries for global queries and standard vector search for local queries. It doesn’t bridge between local and global. Reasonara’s policy retriever adaptively combines both through the MDP framework.

**Bottom line:** GraphRAG is a strong summarization layer. Reasonara is a reasoning layer. They solve different problems. For enterprises that need structured retrieval over complex, multi-source data—and that need cognitive memory to surface latent constraints—Reasonara provides what GraphRAG cannot.


| Dimension             | Microsoft GraphRAG                                            | Reasonara                                                                                |
| --------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Knowledge structure   | Static community-based KG; fixed once created                 | Dynamic causal DAG; evolves via create-or-update consolidation                           |
| Memory representation | LLM-generated summaries (lossy compression)                   | Self-describing (abstraction, value) pairs; full detail preserved                        |
| Graph maintenance     | Full rebuild when data changes; no incremental updates        | O(1) per write, zero maintenance overhead                                                |
| Retrieval method      | Community summaries (global) vector search (local); no bridge | MDP policy trained via GRPO; combines semantic, keyword, implicit memory graph traversal |
| Multi-hop reasoning   | Limited to community boundaries                               | Policy-driven retrieval across full memory graph                                         |
| Cognitive memory      | Not addressed                                                 | First-class via cognitive anchors and Implication Chain Generation                       |
| Indexing cost         | Linear in LLM calls; prohibitive at 125M+ tokens              | Deterministic segmentation targeted LLM; sub-linear scaling                              |
| Token efficiency      | Moderate reduction via summaries; loses detail                | 98% reduction; 1K–2.5K tokens/query with full detail                                     |
| Trainable retrieval   | No; fixed community-level or vector retrieval                 | Yes; GRPO learns retrieval depth from task quality                                       |
| Schema flexibility    | Fixed community detection algorithm                           | Schema-free retrieval anchors; extensible via prompt changes only                        |
| Benchmark (LoCoMo)    | Not benchmarked on LoCoMo                                     | 0.933 (SOTA)                                                                             |


### **Graph-Based Systems: Zep, MAGMA, SYNAPSE**

These systems build explicit knowledge graphs with predefined edge types. MAGMA maintains four typed graphs (temporal, causal, semantic, entity). Zep uses entity-relation-entity triplets in a temporal knowledge graph. SYNAPSE uses three edge types with spreading activation.

The fundamental limitation: rigid schemas. If information doesn’t fit the predefined edge types, it’s lost or misplaced. And graph maintenance costs grow with memory size—MAGMA manages millions of edges at 100K entries; Zep requires community detection and edge invalidation. Reasonara’s retrieval anchors achieve the same multi-hop connectivity with O(1) creation cost and zero maintenance overhead.

### **Mem0 and Nemori**

Mem0 (0.653 on LoCoMo) and Nemori (0.794) are purpose-built memory systems for conversational agents. Both store memory as flat key-value pairs or structured dual stores. Neither supports multi-hop retrieval, trainable policies, or schema-free organization that adapts to content. Nemori’s fixed episodic/semantic split can’t bridge distant memories; Mem0’s flat structure fragments related information. Both also score in the 15–26% range on LoCoMo-Plus, indicating that neither addresses cognitive memory in any structured way.

### **Standard RAG (LangChain, LlamaIndex)**

These are retrieval frameworks, not memory systems. They provide the plumbing for vector search but don’t build persistent knowledge structures. No consolidation, no causal reasoning, no learned retrieval. Reasonara outperforms LangChain by 5–8 points on UltraDomain with a 25:1 win ratio on disagreements.

### **Full-Context LLMs**

Stuffing everything into a large context window is the brute-force alternative. It works for small corpora but fails at enterprise scale: token cost scales linearly, latency increases, and accuracy drops due to attention dilution and the “lost-in-the-middle” effect. Reasonara achieves higher accuracy (0.933 vs. 0.825 on LoCoMo) at 98% fewer tokens. On LoCoMo-Plus, the gap widens further: a frontier LLM with the entire conversation in context still scores in the low 20s, because the bottleneck is no longer attention—it is the absence of structured cognitive retrieval.

## **Security & Compliance**

Reasonara is built for regulated industries. Finance, Retail, Healthcare, and Legal are our primary verticals, and we designed the system to meet their security requirements from day one.

- **Deployment model:** VPC deployment within the customer’s cloud environment. Customer data never leaves their infrastructure. Reasonara runs as a service inside the customer’s security boundary.
- **SOC 2 Type II:** Audit completed in April 2026, with full compliance achieved.
- **PII handling:** Provenance metadata includes access scope at the record level. The retrieval policy respects access controls inherited from source systems, ensuring that memory retrieval never surfaces data the querying agent isn’t authorized to see.
- **Data residency:** Because Reasonara deploys in the customer’s VPC, data residency is inherited from the customer’s cloud configuration. No data crosses regional boundaries.

## **Future Research**

We outline several promising directions to further improve the adaptability, efficiency, and trustworthiness of our memory solution.

- **Self-Evolving Memory:** A closed-loop memory improvement framework where the agent continuously monitors its own retrieval and response quality, identifies gaps or errors caused by missing or mis-retrieved memories, and automatically triggers targeted memory updates. Thus creating a self-correcting cycle in which the memory system improves over time without requiring explicit user feedback.
- **Deferred Memory Construction:** Rather than constructing rich memory representations at ingest time, facts are captured cheaply in a raw staging layer and full abstraction, indexing, and cue generation are deferred to an asynchronous background pipeline that processes facts in topic-coherent batches. This decoupling ingests latency from construction quality and enables cross-fact reasoning that is impossible when each fact is processed in isolation.
- **Provenance-Based Memory Access:** Each memory retrieval carries a structured provenance record such as capturing which index, anchor, or reasoning chain led to the memory’s inclusion. This is exposed to the downstream LLM as explicit evidence metadata, enabling more calibrated generation, transparent attribution of personalized responses, and auditable access logs that users can inspect, correct, or revoke.

## **Why This Matters Now**

MIT’s NANDA initiative found that 95% of AI pilots fail because enterprise data isn’t ready. Anthropic’s own data shows that LLMs could automate 57% of knowledge work tasks. However, less than 2% of actual work has been affected. That gap is an enterprise knowledge infrastructure problem.

Reasonara sits in the layer between the data and the model. Every enterprise deploying AI agents needs this layer. The question is whether they build it themselves, stitch together retrieval frameworks, or use purpose-built memory infrastructure that addresses both factual recall and cognitive memory.

**The AI models work. The data is there. What’s missing is the memory infrastructure between them. Infrastructure that can recall what was said and recognize what should still apply. That’s what Reasonara builds.**

## **References**

- 1 [MIT NANDA Initiative](https://mlq.ai/media/quarterly_decks/v0.1_State_of_AI_in_Business_2025_Report.pdf)*, “The GenAI Divide: State of AI in Business 2025”*
- 2 [Murty & Kumar, Harvard Business Review](https://hbr.org/2026/02/when-every-company-can-use-the-same-ai-models-context-becomes-a-competitive-advantage)*, “When Every Company Can Use the Same AI Models, Context Becomes a Competitive Advantage,” 2025*
- 3 [Massenkoff & McCrory](https://www.anthropic.com/research/labor-market-impacts)*, “Labor market impacts of AI: A new measure and early evidence,” Anthropic, March 2026*
- 5 [Miclaus, Coshow & Hare, Gartner](https://www.gartner.com/doc/reprints?id=1-2JHTJ0FC)*, “Emerging Tech: The New Essential Infrastructure for Agentic Systems,” February 2026*
- 6 [Foundation Capital](https://foundationcapital.com/context-graphs-ais-trillion-dollar-opportunity/)*, “Context Graphs: AI’s Trillion-Dollar Opportunity,” December 2025*

*Authors: Bowen Zhu • Dr. Xuchao Zhang • Dr. Liang Zhao • Hasibul Haque  — Causal Dynamics Lab*

*Causal Dynamics Lab | CausalDynamics.com | Confidential and Private*  