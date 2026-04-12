# ab-cursor-dev-setup

Shared **Cursor** conventions for this org: agents under `.cursor/agents/`, skills under `.cursor/skills/`, and optional **`domain-driven/`** presets for Supabase-style domain work.

## DevBot Dojo (Agent Dojo)

The **`agent-dojo`** skill adds practitioner-grade engineering knowledge (RAG + knowledge graph) for design and review. See:

- `.cursor/skills/agent-dojo/SKILL.md` — when to use, MCP/HTTP setup, models, timeouts
- `.cursor/skills/agent-dojo/BRAINLIFT.md` — compressed meta-rules (DOK)
- `.cursor/mcp.json` — pre-configured MCP server for DevBot Dojo (auto-detected by Cursor)

**API endpoint:** `https://api.dojo.ti.trilogy.com` — DevBot is a **public** dojo, no API key required.

**Prerequisites:** Python 3.13+, `uvx` (from `uv`). Install: `curl -LsSf https://astral.sh/uv/install.sh | sh`

The orchestrator skill **`build-or-fix`** references `agent-dojo` in the design and verify steps.
