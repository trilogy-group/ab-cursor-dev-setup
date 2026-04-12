# BRAINLIFT.md — Agent Dojo (DevBot)

> Compressed knowledge artifact for any agent using Agent Dojo.
> Items pass TWO gates: High Importance + Conflict/Non-Consensus.
> Domain: Using Agent Dojo as an expert knowledge source for software engineering decisions.

## DOK1: Foundational Facts

1. **Dojo's value is structured retrieval, not exclusive data access.** LLMs are trained on the same YouTube transcripts. Dojo's edge comes from 12-dimension structured extraction (gotchas, anti_patterns, production_lessons) that makes knowledge retrievable by failure mode — not just by topic. Without this framing, you'll overstate Dojo's uniqueness and underuse dimension filtering. (Source: dojo-strategic-analysis.md) (Added: 2026-03-15)

2. **Question type determines ROI: "should I / what goes wrong / what do teams get wrong" = high value. "How do I / what is" = marginal.** In the 15-question experiment, Dojo averaged 18 unique insights per question on complex trade-off and failure-mode questions, but adds near-zero value on simple "how do I" lookups. Wrong routing wastes 5-15s per query. (Source: Devbot-dojo.md experiment) (Added: 2026-03-15)

3. **`search_knowledge` with dimension filter is often superior to `ask_dojo` for code review and anti-pattern detection.** `ask_dojo` synthesizes a full answer (5-15s). `search_knowledge` returns raw knowledge items (<3s) that a reviewing agent can directly cite against a diff. For ReviewBot anti-pattern checks, raw items are more actionable than narrative. (Source: handler.mjs API design) (Added: 2026-03-15)

4. **The `gotchas` dimension (2,154 items) is the single most valuable resource for preventing production incidents.** In the experiment, Dojo's unique insights clustered overwhelmingly around production failure modes, not best practices. Always query `dimension: "gotchas"` for any proposed technology stack. (Source: experiment results — 96% of unique insights were failure-mode knowledge) (Added: 2026-03-15)

5. **Dojo complements web search; it does not replace it.** WebSearch produced 278 unique insights in the same experiment. For current/recent technology changes (2025-2026 releases, new APIs, breaking changes), web search is superior. For battle-tested production knowledge and failure modes, Dojo is superior. Use both. (Source: Devbot-dojo.md — 278 WebSearch uniques vs 270 Dojo uniques) (Added: 2026-03-15)

6. **Coverage has specific gaps: mobile-native, .NET/C#, Rust, game dev, embedded systems.** Querying these domains returns generic results indistinguishable from base LLM. Strong coverage: DevOps/CI-CD, Architecture, API Design, PostgreSQL/Redis, React/Next.js, AWS/Cloud, Security, Docker/K8s, DynamoDB, Testing. (Source: experiment_report.md) (Added: 2026-03-15)

7. **Six models available — `gpt-5.4` is the default, use it when unsure.** GPT-5.4 has 33% fewer hallucinations, SWE-Bench Pro 57.7%, 1M context, and is 6x cheaper than Opus — best all-rounder for expert Q&A. Escalate to `claude-opus-4.6` (SWE-Bench 80.8%, 128K output) for deep multi-file analysis, or `gemini-3-pro` (GPQA 94.3%, 2M context) when you need exhaustive detail or massive context synthesis. `claude-sonnet-4.6` is a balanced middle ground for code review. `gemini-3-flash` should ONLY be used for trivial one-liner lookups — it gives the least detailed answers and wastes Dojo's knowledge depth. The knowledge base retrieval is model-independent; model choice only affects answer generation. (Source: March 2026 benchmarks — SWE-Bench, GPQA Diamond, Terminal-Bench 2.0) (Updated: 2026-03-27)

8. **The `openclaw` dojo exists separately with 1,010 items from 39 experts about the OpenClaw AI agent framework.** When working on OpenClaw-related issues, switch `dojo: "openclaw"`. The default `"devbot"` dojo has no OpenClaw-specific knowledge and vice versa. (Source: experiment_report.md) (Added: 2026-03-15)

9. **Every `/ask` query is logged to DynamoDB (`agent-dojo-query-log`) with 90-day TTL, including the full answer.** This creates a feedback loop: frequently-asked questions with low RAG hits indicate knowledge gaps. Do not avoid querying to "save resources" — the logging is fire-and-forget with zero latency impact. (Source: handler.mjs logQuery function) (Added: 2026-03-15)

10. **The Knowledge Graph (FalkorDB) adds relationship context that RAG alone misses.** When `graph_context_found: true` in the response, the answer was enriched with technology relationship data (ABOUT, RELATES_TO, USED_WITH, ALTERNATIVE_TO, CONFLICTS_WITH). This is why Dojo can surface cross-technology gotchas that neither individual technology's docs mention. (Source: experiment_report.md — 6,533 nodes, 14,079 relationships) (Added: 2026-03-15)

11. **`ask_dojo` with `gpt-5.4`, `claude-opus-4.6`, or `claude-sonnet-4.6` can exceed the default 60s client timeout.** GPT-5.4 and Opus take ~70s; Sonnet can spike above 60s on complex questions. The Lambda backend has a 300s timeout — it works fine. The bottleneck is the MCP client (mcporter, Claude Code, Cursor) defaulting to 60s. Always set `export MCPORTER_CALL_TIMEOUT=120000` in your shell profile, or pass `--timeout 120000` on each call. Lighter models (`gemini-3-flash` ~20s, `gpt-5.3` ~40s, `gemini-3-pro` ~30s) work within the default timeout. If `ask_dojo` seems to "hang" or "timeout", this is why — not a server issue. (Source: production debugging, 2026-03-27) (Updated: 2026-03-27)

## DOK2: Compressed Summaries

1. **The 15-question controlled experiment proves Dojo's value is in practitioner knowledge, not training data.** Across questions spanning PostgreSQL isolation levels, Webpack Module Federation, DynamoDB at scale, Kubernetes RBAC, CI/CD secrets management, Docker multi-stage builds, and more — Dojo uniquely surfaced: DAX attribute metadata caching bugs, PostgreSQL snapshot acquisition timing, NotPetya response misclassification patterns, DynamoDB throttling-as-system-design-event framing, Istio ambient mode, and retry layer stacking across client/gateway/mesh. None of these were "hidden" from LLMs — they were unstructured in training data. Dojo's structured extraction + dimension filtering made them retrievable on demand. (Source: Devbot-dojo.md) (Added: 2026-03-15)

2. **Dojo's strategic position: strong technology, data source is a starter dataset, not the moat.** YouTube KB is a proof-of-concept; companies like Interloom ($16.5M) and Edra ($309M) are raising hundreds of millions to solve the same "tacit knowledge" problem but with proprietary data (postmortems, Slack, PR reviews, ADRs). The real product pivot is private knowledge ingestion — treating YouTube as the template/demo tier and company-specific knowledge as the paid tier. The engineering (RAG + KG + Graphiti + MCP) is genuinely strong. (Source: dojo-strategic-analysis.md) (Added: 2026-03-15)

3. **Dojo's dimension taxonomy maps directly to DevBot's subagent architecture, and this mapping is the most efficient query strategy.** ArchitectBot → `architecture_patterns` + `gotchas` + `tool_comparisons`. ReviewBot → `anti_patterns` + `security_practices`. QABot → `debugging_techniques` + `procedures`. CodeBot → `procedures` + `gotchas`. This is more efficient than asking broad questions because dimension-filtered search returns items that match each subagent's specific decision surface. (Source: SKILL.md DevBot Integration Guide) (Added: 2026-03-15)

## DOK3: Non-Obvious Insights

1. **The 15% score improvement (8.8 vs 7.4-8.0) dramatically understates Dojo's value because it measures answer quality, not answer content type.** Base LLM scores 7-8/10 because it knows the general concepts correctly. What it misses are production failure modes, specific configuration gotchas, and diagnostic frameworks that prevent actual incidents. A score delta of 0.8 points looks marginal, but the content of that delta is disproportionately valuable — it's the difference between "knows what to do" and "knows what goes wrong." Evaluate Dojo by whether unique insights would have prevented a real incident, never by aggregate scores. (Source: Devbot-dojo.md experiment — analyzed per-question) (Added: 2026-03-15)

2. **Context window expansion to 1M+ tokens erodes RAG's advantage for public general knowledge, but NOT for structured dimension-filtered retrieval.** You can paste entire docs into context. You cannot paste "the 674 anti-patterns categorized by technology and consequence type" — that requires structured extraction. Dojo's defensible feature is dimension-filtered search (`search_knowledge` with specific dimension), not the general Q&A endpoint. As context windows grow, the value of `ask_dojo` decreases while the value of `search_knowledge` with dimension filtering increases. (Source: dojo-strategic-analysis.md — context window analysis) (Added: 2026-03-15)

3. **Querying Dojo unconditionally before every M/L/XL task is cheaper than the cost of missing one known anti-pattern.** The experiment showed 96% KB-verified unique insights across ALL 15 diverse questions — PostgreSQL, Webpack, DynamoDB, K8s, Docker, Next.js, REST APIs, disaster recovery, CI/CD secrets. There is no way to predict which questions will yield unique insights without asking. The cost (5-15s latency, one API call) is trivially small compared to the cost of a PR rejected for a known anti-pattern or a production incident from a documented gotcha. (Source: Devbot-dojo.md — uniform value across diverse domains) (Added: 2026-03-15)

## DOK4: Spiky Points of View

1. **Claim: "Only query Dojo for hard questions" is wrong because identifying what's hard requires the knowledge Dojo provides.** Evidence: In the experiment, Dojo added 12-33 unique insights per question across questions that APPEARED simple ("Should I use microservices?", "Is active-active worth it?"). The assumption that an agent can predict which questions will benefit from Dojo is a form of the Dunning-Kruger effect applied to information retrieval — you can't know what you don't know. The default heuristic should be "always query Dojo for M/L/XL tasks" with opt-out, not "only query when it seems hard" with opt-in. (Added: 2026-03-15)

2. **Claim: Dojo's YouTube-sourced knowledge base is NOT a competitive moat and should be treated as a demo dataset, not a product differentiator.** Evidence: YouTube transcripts are in LLM training corpora — this is why the score delta is 15%, not 50%. The "Data Moat 2.0" research (Alpha-Matica, 2026) shows competitive advantage comes from proprietary feedback loops — data competitors literally cannot access. The default assumption "more YouTube videos = better Dojo" is wrong; the right investment is proprietary data sources (company postmortems, Slack channels, PR review patterns, ADRs). Interloom ($16.5M) and Edra ($309M) validate this thesis. (Source: dojo-strategic-analysis.md) (Added: 2026-03-15)

---

**Last updated:** 2026-04-12 (synced into `ab-cursor-dev-setup` from LearnLens `experiment/demo/agent-dojo/devbot-dojo/BRAINLIFT.md`)
**Items:** DOK1: 11/15 | DOK2: 3/10 | DOK3: 3/5 | DOK4: 2/3
**Domain:** Using Agent Dojo as an expert knowledge source for software engineering decisions
**Source documents:** Devbot-dojo.md, dojo-strategic-analysis.md, experiment_report.md, handler.mjs
