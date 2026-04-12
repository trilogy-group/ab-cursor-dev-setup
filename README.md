# ab-cursor-dev-setup

Shared **Cursor** conventions for this org: agents under `.cursor/agents/`, skills under `.cursor/skills/`, and optional **`domain-driven/`** presets for Supabase-style domain work.

## DevBot Dojo (Agent Dojo)

The **`devbot-dojo`** skill adds practitioner-grade engineering knowledge (RAG + knowledge graph) for design and review. See:

- `.cursor/skills/devbot-dojo/SKILL.md` — when to use, MCP/HTTP setup (secrets via env), models, timeouts
- `.cursor/skills/devbot-dojo/BRAINLIFT.md` — compressed meta-rules (DOK)
- `.cursor/skills/devbot-dojo/references/` — API summary and experiment highlights

Configure **`AGENT_DOJO_API_BASE`** and MCP credentials from team secrets — do not commit keys.

The orchestrator skill **`build-or-fix`** references `devbot-dojo` in the design and verify steps.