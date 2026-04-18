---
name: agent-dojo
description: >-
  Use for practitioner-grade software engineering knowledge from Agent Dojo (DevBot dojo):
  production failure modes, architecture trade-offs, anti-patterns, and review checklists.
  Ground requests via MCP (`ask_dojo`, `search_knowledge`, `list_dojos`) or HTTP (see this file).
  Pair with BRAINLIFT.md for compressed meta-rules (routing, timeouts). Do not use for trivial
  syntax lookups or bleeding-edge release notes — use base LLM + web search instead.
disable-model-invocation: false
---

# Agent Dojo — DevBot Dojo expert knowledge

Agent Dojo exposes **DevBot Dojo** (`dojo: "devbot"`) through MCP tools and a REST API. Answers are grounded in a **RAG + knowledge graph** (structured dimensions: gotchas, anti_patterns, procedures, etc.), not generic training data alone.

**Companion file:** **`BRAINLIFT.md`** in this directory — routing, timeouts, strategic use.

---

## When to use

| Pattern | Example |
|---------|---------|
| Trade-offs & failure modes | "What goes wrong with X in production?" |
| Architecture / "should I" | "DAX vs Redis for DynamoDB caching given …" |
| Anti-patterns & review | "What do teams get wrong about K8s RBAC?" |
| Security beyond docs | Misconfigurations that enable lateral movement |
| Systematic debugging | Triage workflows experts use for Y in Z |

## When **not** to use

- Simple syntax / API one-liners — base LLM is faster.
- **Latest** release notes (2025–2026) — **web search** is fresher.
- Questions about **your private repo only** — Dojo has no codebase access unless you paste context.

**Routing rule:** Prefer Dojo for "should I", "what goes wrong", "what teams get wrong", "how do I avoid". For bare "what is / how do I install" without production risk, skip Dojo unless the task is M/L/XL.

---

## MCP connection (recommended)

The MCP server connects your agent (Cursor, Claude Code, Codex, Gemini CLI) to DevBot Dojo. DevBot is a **public** dojo — no API key required.

### Option 1: MCP Hive (cloud — no local setup)

The server is hosted on MCP Hive. Add to your `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "agent-dojo": {
      "url": "https://mcp-server.ti.trilogy.com/098ab494/sse?x-api-key=sk-hive-api01-MDQ3NjI4ZDgtMjNmNi00MGNmLWI4NjYtYzBmOWNmMjgzOTYx-N2U4YmNjAAA"
    }
  }
}
```

The `x-api-key` query parameter is **required** for the SSE connection — without it the endpoint rejects the request. This is the MCP Hive connection key, not a dojo-level API key.

This is already configured in this repo's `.cursor/mcp.json`.

### Option 2: Local (uvx — if MCP Hive is unavailable)

Runs the MCP server locally via Python. Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "agent-dojo": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/kabir-ti/agent-dojo-mcp-server", "agent-dojo-mcp"],
      "env": {
        "AGENT_DOJO_API_URL": "https://zbjffcjzsnhqay2c5ckc7damky0tvotz.lambda-url.us-east-1.on.aws",
        "AGENT_DOJO_DEFAULT": "devbot"
      }
    }
  }
}
```

**Prerequisites:** Python 3.13+, `uvx` (from `uv`). Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`

### Environment variables (local mode only)

| Variable | Value | Purpose |
|----------|-------|---------|
| `AGENT_DOJO_API_URL` | `https://zbjffcjzsnhqay2c5ckc7damky0tvotz.lambda-url.us-east-1.on.aws` | REST API endpoint |
| `AGENT_DOJO_DEFAULT` | `devbot` | Default dojo for all queries |
| `AGENT_DOJO_API_KEY` | _(not needed)_ | DevBot is public — no key required |

> For **private** dojos, pass `api_key` as a tool parameter or set `AGENT_DOJO_API_KEY` env var.

---

## REST API (direct HTTP)

**Base URL:** `https://zbjffcjzsnhqay2c5ckc7damky0tvotz.lambda-url.us-east-1.on.aws`

### Endpoints

| Path | Method | Purpose |
|------|--------|---------|
| `/ask` | POST | Expert Q&A — JSON body, JSON response |
| `/query` | POST | Streaming (SSE) — playground-style |
| `/dojos` | GET | List dojos with stats |
| `/knowledge` | GET | Raw knowledge search/browse |

### POST `/ask`

**Headers:** `Content-Type: application/json`
**Body (JSON):**

```json
{
  "question": "string (required)",
  "dojo": "devbot",
  "model": "gpt-5.4"
}
```

**curl example:**

```bash
curl -sS -X POST "https://zbjffcjzsnhqay2c5ckc7damky0tvotz.lambda-url.us-east-1.on.aws/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"What goes wrong with PostgreSQL connection pooling at scale?","dojo":"devbot","model":"gpt-5.4"}'
```

**Response fields:** `answer`, `sources`, `dojo`, `model`, `knowledge_items_used`, `graph_context_found`.

### GET `/knowledge`

**Query params:** `dojo` (default `devbot`), `dimension`, `q`, `from`, `size`.

```bash
curl "https://zbjffcjzsnhqay2c5ckc7damky0tvotz.lambda-url.us-east-1.on.aws/knowledge?dojo=devbot&dimension=gotchas&q=postgres&from=0&size=20"
```

### GET `/dojos`

```bash
curl "https://zbjffcjzsnhqay2c5ckc7damky0tvotz.lambda-url.us-east-1.on.aws/dojos"
```

Optionally pass `?email=you@company.com` to include your private and shared dojos alongside public ones.

> DevBot is **public** — no `x-api-key` header required for any of the above calls.

---

## MCP tools

### `ask_dojo`

Full synthesized answer + sources. Params: `question` (required), `dojo` (default `"devbot"`), `model` (default `"gpt-5.4"`), `api_key` (optional, for private dojos). Check `graph_context_found` for KG enrichment.

### `list_dojos`

Domains, stats, suggested questions — fast (~1–2s). Optional param: `email` — include your email to see private/shared dojos alongside public ones.

### `search_knowledge`

Raw items (faster, **better for PR review / checklists**). Params: `query` (required), `dojo` (default `"devbot"`), `dimension` (optional), `limit` (default 20, max 50), `api_key` (optional, for private dojos).

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
| `performance_insights` | Performance tuning |
| `expert_opinions` | Practitioner consensus |

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

Server-side allows **300s**. Many MCP/CLI clients default to **60s**. **`gpt-5.4`, `claude-opus-4.6`, `claude-sonnet-4.6` often exceed 60s** on hard questions.

- Set your agent's tool timeout to **≥ 120s** for `ask_dojo`.
- For mcporter: `MCPORTER_CALL_TIMEOUT=120000`.
- The MCP server has a **180-second** HTTP timeout internally.

If calls "hang" at ~60s, this is almost always the **client**, not Dojo. Retry with a higher timeout.

---

## Query craft

**Weak:** "Tell me about Kubernetes security."
**Strong:** "What RBAC misconfigurations on EKS with Helm + shared namespaces lead to privilege escalation or lateral movement?"

Include: **stack**, **scale**, **constraints**, and **decision framing** ("should I X or Y").

---

## DevBot / subagent mapping (efficient routing)

| Subagent | Primary dimensions |
|----------|---------------------|
| Architect | `architecture_patterns`, `gotchas`, `tool_comparisons` |
| Review | `anti_patterns`, `security_practices` (and `gotchas` for stack) |
| QA | `debugging_techniques`, `procedures` |
| Implementation | `procedures`, `gotchas` |

---

## Integration with `build-or-fix` (this skill: `agent-dojo`)

- **Design (step 2):** For non-trivial or production-impacting design, run Dojo **before** finalizing the design doc — trade-offs and failure modes.
- **Verify / review (step 4):** Use **`search_knowledge`** + `anti_patterns` / `gotchas` for technologies touched in the diff.
- Do **not** block the pipeline if MCP is unavailable — fall back to human review and note the gap.

---

## Coverage

**Strong:** DevOps/CI, architecture, APIs, PostgreSQL/Redis, React/Next, cloud, security, Docker/K8s, DynamoDB, testing.

**Weak / generic:** Mobile-native, .NET/C#, Rust, games, embedded — prefer base LLM + docs.

---

## Files in this skill

| File | Content |
|------|---------|
| `BRAINLIFT.md` | Compressed meta-rules (routing, timeouts, strategy) |
