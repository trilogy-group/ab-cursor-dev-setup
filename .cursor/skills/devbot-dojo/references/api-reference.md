# Agent Dojo API Reference

HTTP endpoints for direct integration (no MCP). **Do not commit API keys** — use team secrets or env vars.

## Endpoints

| Endpoint | Method | Purpose | Response Type |
|----------|--------|---------|---------------|
| `/ask` | POST | Expert Q&A (MCP-ready, non-streaming) | JSON |
| `/query` | POST | Playground (SSE streaming, 3 columns) | SSE |
| `/dojos` | GET | List available dojos with stats | JSON |
| `/knowledge` | GET | Browse/search raw knowledge items | JSON |

## POST /ask

**Request:**
```json
{
  "question": "string (required)",
  "dojo": "devbot | openclaw (default: devbot)",
  "model": "gpt-5.4 | gpt-5.3 | claude-sonnet-4.6 | claude-opus-4.6 | gemini-3-pro | gemini-3-flash (default: gpt-5.4)"
}
```

**Response (200):**
```json
{
  "answer": "string — full expert answer",
  "sources": [
    {"type": "dimension_name", "title": "item title", "video": "video_id"}
  ],
  "dojo": {"id": "devbot", "name": "DevBot Dojo"},
  "model": "openai/gpt-5.4",
  "knowledge_items_used": 10,
  "graph_context_found": true
}
```

## Base URLs (configure via env)

| Variable | Example | Notes |
|----------|-----------|--------|
| `AGENT_DOJO_API_BASE` | `https://dojo.learnlens.in/api` | Production API prefix; paths are `/ask`, `/dojos`, etc. |

**Example:**
```bash
curl -sS -X POST "${AGENT_DOJO_API_BASE}/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"What goes wrong with PostgreSQL connection pooling at scale?","dojo":"devbot","model":"gpt-5.4"}'
```

If your deployment uses an API key, pass it the way your platform documents (header or query) — **never hardcode in this repo.**

## MCP Hive (optional)

For SSE MCP, use the URL and API key from **team secrets** (e.g. 1Password). Placeholder:

```
${MCP_HIVE_SSE_URL}?x-api-key=${AGENT_DOJO_MCP_API_KEY}
```

Set `MCPORTER_CALL_TIMEOUT=120000` (or equivalent) for `ask_dojo`-style calls — see main `SKILL.md`.

## Model & timeout summary

See `SKILL.md` — default `gpt-5.4`; use **120s client timeout** for GPT-5.4, Opus, and Sonnet on complex questions.
