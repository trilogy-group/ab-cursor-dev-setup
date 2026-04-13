# BRAINLIFT.md — Agent Dojo (DevBot)

> Compressed knowledge for agents using DevBot Dojo as an expert source.
> Domain: software engineering decisions grounded in structured retrieval (dimensions + knowledge graph).

## DOK1: Foundational Facts

1. **Dojo's value is structured retrieval, not exclusive data access.** The edge is 12-dimension extraction (gotchas, anti_patterns, production_lessons, etc.) so knowledge is retrievable by failure mode — not only by topic. Without this framing, you'll underuse dimension filtering. (Source: product strategy docs)

2. **Question type determines ROI:** “Should I / what goes wrong / what do teams get wrong” → high value. Bare “how do I / what is” lookups → often better served by the base LLM for speed. Wrong routing wastes time per query.

3. **`search_knowledge` with a dimension filter is often better than `ask_dojo` for code review and anti-pattern checks.** `ask_dojo` synthesizes a full answer (longer). `search_knowledge` returns raw items faster; reviewers can cite them against a diff.

4. **The `gotchas` dimension is the highest-signal resource for production risk.** Prefer `dimension: "gotchas"` when evaluating a proposed stack or change.

5. **Dojo complements web search; it does not replace it.** For very new releases, breaking API changes, or “what shipped last month,” web search is fresher. For battle-tested production failure modes and practitioner trade-offs, Dojo is stronger. Use both when needed.

6. **Coverage has gaps:** mobile-native, .NET/C#, Rust, games, embedded — answers may feel generic. Strong areas include DevOps/CI, architecture, APIs, PostgreSQL/Redis, React/Next, AWS/cloud, security, Docker/K8s, DynamoDB, testing.

7. **Default model: `gpt-5.4`.** Escalate to `claude-opus-4.6` for deep multi-file work, `claude-sonnet-4.6` for balanced review, `gemini-3-pro` for very large context synthesis. Use `gemini-3-flash` only for trivial one-liners — not for deep expert answers. Retrieval is model-independent; only the answer wording changes.

8. **`/ask` requests may be logged server-side (e.g. DynamoDB with TTL) for product analytics.** Do not skip queries to “save resources” unless your org policy says otherwise.

9. **Knowledge graph (e.g. FalkorDB) adds relationship context RAG alone can miss.** When `graph_context_found: true`, the answer used technology relationship context (e.g. ABOUT, RELATES_TO, USED_WITH).

10. **`ask_dojo` with `gpt-5.4`, `claude-opus-4.6`, or `claude-sonnet-4.6` can exceed a 60s client timeout.** The backend allows up to 300s; clients default to 60s. Set tool timeout to ≥120s (mcporter: `MCPORTER_CALL_TIMEOUT=120000`). The MCP server has a 180s internal HTTP timeout. Lighter models (`gemini-3-flash`) often finish within 60s.

## DOK2: Compressed Summaries

1. **Structured dimensions + graph context make “failure mode” knowledge usable on demand** — not because raw facts are hidden from LLMs, but because they are organized for retrieval by consequence and stack.

2. **Long-term product direction:** proprietary org knowledge (postmortems, ADRs, internal reviews) is the durable tier; public curated corpora are the template layer. Engineering (RAG + KG + MCP) stays valuable across both.

3. **Dimension → role mapping (efficient routing):** Architect → `architecture_patterns`, `gotchas`, `tool_comparisons`. Review → `anti_patterns`, `security_practices`. QA → `debugging_techniques`, `procedures`. Implementation → `procedures`, `gotchas`.

## DOK3: Non-Obvious Insights

1. **Aggregate “quality scores” understate operational value** — the important delta is often specific failure modes and diagnostics, not a single number.

2. **As context windows grow, broad Q&A competes with pasted docs — dimension-filtered `search_knowledge` stays distinctive** for catalogued anti-patterns and gotchas at scale.

3. **For M/L/XL tasks, querying Dojo before implementation is cheap** relative to missing a documented gotcha in review or production.

## DOK4: Spiky Points of View

1. **“Only use Dojo for hard questions” is a weak default** — what looks easy may still benefit from failure-mode retrieval. Prefer “use Dojo for M/L/XL unless you opt out.”

2. **Public curated corpora are not a standalone moat** — durable advantage pairs retrieval quality with proprietary, org-specific knowledge loops.

---

**Last updated:** 2026-04-12  
**Domain:** DevBot Dojo as expert knowledge for software engineering
