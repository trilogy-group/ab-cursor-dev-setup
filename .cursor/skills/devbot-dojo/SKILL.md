---
name: devbot-dojo
description: >-
  Use for practitioner-grade software engineering knowledge from Agent Dojo (DevBot Dojo):
  production failure modes, architecture trade-offs, anti-patterns, and review checklists.
  Ground requests via MCP (`ask_dojo`, `search_knowledge`, `list_dojos`) or HTTP POST /ask.
  Pair with BRAINLIFT.md for meta-rules on when Dojo ROI is highest. Do not use for trivial
  syntax lookups or bleeding-edge release notes — use base LLM + web search instead.
disable-model-invocation: false
---

# DevBot Dojo — Agent Dojo expert knowledge

Agent Dojo exposes **DevBot Dojo** (`dojo: "devbot"`) and **OpenClaw Dojo** (`dojo: "openclaw"`) through MCP tools and a REST API. Answers are grounded in a **RAG + knowledge graph** (structured dimensions: gotchas, anti_patterns, procedures, etc.), not generic training data alone.

**Upstream reference:** LearnLens `experiment/demo/agent-dojo/devbot-dojo/` (kept in sync conceptually with this skill).

**Companion file:** Read **`BRAINLIFT.md`** in this directory for compressed DOK facts (routing, timeouts, strategic use).

---

## When to use

| Pattern | Example |
|---------|---------|
| Trade-offs & failure modes | “What goes wrong with X in production?” |
| Architecture / “should I” | “DAX vs Redis for DynamoDB caching given …” |
| Anti-patterns & review | “What do teams get wrong about K8s RBAC?” |
| Security beyond docs | Misconfigurations that enable lateral movement |
| Systematic debugging | Triage workflows experts use for Y in Z |

## When **not** to use

- Simple syntax / API one-liners — base LLM is faster.
- **Latest** release notes (2025–2026) — **web search** is fresher.
- Questions about **your private repo only** — Dojo has no codebase access unless you paste context.

**Routing rule:** Prefer Dojo for “should I”, “what goes wrong”, “what teams get wrong”, “how do I avoid”. For bare “what is / how do I install” without production risk, skip Dojo unless the task is M/L/XL.

---

## Configuration (no secrets in git)

| Env var | Purpose |
|---------|---------|
| `AGENT_DOJO_API_BASE` | HTTP API base, e.g. `https://dojo.learnlens.in/api` |
| `AGENT_DOJO_MCP_API_KEY` | MCP / Hive key — store in 1Password or CI secrets |
| `MCP_HIVE_SSE_URL` | Optional full SSE URL **without** embedding the key in repo |
| `MCPORTER_CALL_TIMEOUT` | Set `120000` (ms) for reliable `ask_dojo` / GPT-5.4 / Opus / Sonnet |

**Direct HTTP (no MCP):**

```bash
curl -sS -X POST "${AGENT_DOJO_API_BASE}/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"...","dojo":"devbot","model":"gpt-5.4"}'
```

**MCP Hive / Cursor `mcp.json`:** Use the **team-provided** SSE URL and key — copy from vault, not from this file. Example shape only:

```json
{
  "mcpServers": {
    "agent-dojo": {
      "url": "${MCP_HIVE_SSE_URL_WITH_SECRET_FROM_VAULT}"
    }
  }
}
```

**Local MCP (uvx):** optional — run from `git+https://github.com/trilogy-group/agent-dojo-mcp-server` per upstream docs.

Details: **`references/api-reference.md`**.

---

## Models (answer generation)

| Key | Use when |
|-----|----------|
| `gpt-5.4` | **Default** — best all-rounder for expert Q&A |
| `claude-opus-4.6` | Deepest multi-file / subtle issues |
| `claude-sonnet-4.6` | Balanced review / analysis |
| `gemini-3-pro` | Very long or exhaustive synthesis |
| `gpt-5.3` | Structured step-by-step |
| `gemini-3-flash` | **Trivial one-liners only** — not deep expert answers |

**Retrieval is model-agnostic**; only the **wording** of the answer changes.

---

## Client timeouts (critical)

Server-side Lambda allows **300s**. Many MCP/CLI clients default to **60s**. **`gpt-5.4`, `claude-opus-4.6`, `claude-sonnet-4.6` often exceed 60s** on hard questions.

- Set **`MCPORTER_CALL_TIMEOUT=120000`** (or per-call `--timeout 120000`).
- For **Cursor / Claude Code MCP**, set client timeout **≥ 120s** for `ask_dojo`.

If calls “hang” at ~60s, this is almost always the **client**, not Dojo.

---

## MCP tools (summary)

### `ask_dojo`

Full synthesized answer + sources. Params: `question` (required), `dojo` (`devbot`|`openclaw`), `model` (default `gpt-5.4`). Check `graph_context_found` for KG enrichment.

### `list_dojos`

Domains, stats, suggested questions — fast (~1–2s).

### `search_knowledge`

Raw items (faster, **better for PR review / checklists**). Params: `query`, `dojo`, `dimension` (optional), `limit`.

**Prefer `search_knowledge`** when you need citeable bullets against a diff; **`ask_dojo`** when you need narrative trade-off analysis.

### Dimensions (high signal)

| Dimension | Focus |
|-----------|--------|
| `gotchas` | Production failures — **query for any new stack** |
| `anti_patterns` | Named mistakes with consequences |
| `architecture_patterns` | Proven patterns + trade-offs |
| `security_practices` | Real misconfigurations |
| `tool_comparisons` | X vs Y with evidence |
| `debugging_techniques` | Triage workflows |
| `procedures` | Step-by-step patterns |
| `production_lessons` | Incident-learned lessons |

Full table: upstream SKILL + **`BRAINLIFT.md`**.

---

## Query craft (short)

**Weak:** “Tell me about Kubernetes security.”  
**Strong:** “What RBAC misconfigurations on EKS with Helm + shared namespaces lead to privilege escalation or lateral movement?”

Include: **stack**, **scale**, **constraints**, and **decision framing** (“should I X or Y”).

---

## DevBot / subagent mapping (efficient routing)

| Subagent | Primary dimensions |
|----------|---------------------|
| Architect | `architecture_patterns`, `gotchas`, `tool_comparisons` |
| Review | `anti_patterns`, `security_practices` (and `gotchas` for stack) |
| QA | `debugging_techniques`, `procedures` |
| Implementation | `procedures`, `gotchas` |

---

## Integration with `build-or-fix`

- **Design (step 2):** For non-trivial or production-impacting design, run Dojo **before** finalizing the design doc — trade-offs and failure modes.
- **Verify / review (step 4):** Use **`search_knowledge`** + `anti_patterns` / `gotchas` for technologies touched in the diff.
- Do **not** block the pipeline if MCP is unavailable — fall back to human review and note the gap.

---

## Coverage

**Strong:** DevOps/CI, architecture, APIs, PostgreSQL/Redis, React/Next, cloud, security, Docker/K8s, DynamoDB, testing.

**Weak / generic:** Mobile-native, .NET/C#, Rust, games, embedded — prefer base LLM + docs.

---

## References in this folder

| File | Content |
|------|---------|
| `BRAINLIFT.md` | DOK meta-knowledge, timeouts, strategy |
| `references/api-reference.md` | HTTP endpoints, env-based base URL |
| `references/experiment-results.md` | Summary metrics from the 15-question study |
