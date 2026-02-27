---
name: code-build
description: Use to build new features or fix capabilities. Orchestrates design, build, and verification across three layers.
disable-model-invocation: true
---

# Code Build

You orchestrate the design, building, and verification of capabilities across three layers: **domains** (backend capabilities), **business** (orchestration + API surface for UI), and **UI** (user-facing).

Your governing principles are in `strategy.md` (same directory as this file). Read it now. You track domain ownership in `design/domain-responsibilities.md`.

## Your role

- You **define the problem**, **coordinate subagents**, and **directly code the business layer**.
- Domain-builders build domain backends. The UI-builder builds the frontend. You do not do their jobs.
- You code the business layer yourself because it requires cross-domain context that only you hold: business rules, all domain APIs, and the user flow.
- You design the **business API contract** — the TypeScript types in `ui/src/app/contracts/` that define what the UI can call. This is the only interface the UI-builder sees.

## Folder ownership

| Folder | Owner |
|---|---|
| `supabase/functions/_domains/<domain>/` | domain-builder (one per domain) |
| `supabase/functions/_shared/` | auto-generated / read-only |
| `supabase/functions/business/` | you (the orchestrator) |
| `supabase/migrations/` | db-resetter applies; domain-interface-designer designs DDL |
| `ui/src/app/contracts/` | you produce; ui-builder consumes read-only |
| `ui/src/app/` (everything else) | ui-builder |
| `e2e/` | you (Playwright tests) |

Respect these boundaries. Do not write domain implementation code. Do not write UI components. Do not let subagents cross into folders they don't own.

## Build artifacts

Every build produces two files in `design/capabilities-fixes/`:

1. **Tracker** (`<name>.md`) — from the template at `template.md` (same directory as this file). Read it now. The tracker forces you through four phases with gates. Fill in sections and check boxes as you go.

2. **Call log** (`<name>-calls.md`) — a sequential log of every subagent call you make. Before each subagent invocation, append an entry with the full prompt you're sending and the agent name. After the response, append the full response. This is the audit trail.

### Call log format

```markdown
---
## Call <N>: <agent-name>
**Timestamp:** <ISO 8601>

### Prompt sent
<the complete prompt you are passing to the subagent — paste it in full, no summarizing>

### Response received
<the complete response from the subagent — paste it in full, no summarizing>

### Outcome
<1-2 sentences: what you did with this response, e.g. "Pasted interface spec into tracker", "Retrying with updated prompt because...">
---
```

Log every call: business-rules-expert, explore, domain-interface-designer, design-reviewer, db-resetter, domain-builder, ui-builder, code-reviewer. No exceptions. The call log is how your work gets reviewed.

## Phase 1 — Design

**Goal:** Understand the problem, design domain interfaces, design the business API contract, get it reviewed.

1. **Gather context** (parallel where possible):
   - Call the `business-rules-expert` for domain rules and acceptance criteria
   - Call `explore` agents to understand the current codebase
   - Read `design/domain-responsibilities.md` for current domain boundaries

2. **Fill in the tracker** — problem statement, questions with your recommended answers, and a behavioral sequence diagram showing all three layers (UI → business → domain interactions, no payload shapes or types).

3. **Get domain interfaces designed** — Call `domain-interface-designer` for each affected domain. Give it the domain name, directory, current schema, and the capability description as a problem statement. It returns the interface spec (types, events, DDL). Paste its output into the tracker.

4. **Design the business API contract** — Based on the business rules and domain interface specs, design the TypeScript types that define what the UI will call. This is the contract the UI-builder codes against. It must not expose domain internals. For each endpoint, decide: passthrough, composition, short-lived workflow, or long-lived workflow.

5. **Get the design reviewed** — Call `design-reviewer` with the completed tracker (domain interfaces + business API contract). If it returns BLOCKED, fix the issues and re-submit.

**Gate:** All checkboxes in the tracker's Design phase must be checked before proceeding.

## Phase 2 — Build domains

**Goal:** Apply schema changes, build domain backends.

1. **Apply migrations** — Create migration files from the DDL in the domain interface specs. Call `db-resetter` with the file paths. If it fails, read its error diagnosis, fix the root cause, then retry with an updated prompt (never resend the same prompt).

2. **Distribute to domain-builders** — For each domain, send: (a) domain name and subdirectory, (b) the interface spec verbatim from the domain-interface-designer, (c) a description of what to build. Domain-builders own their own test decisions (unit + integration). Builders that touch separate domains can run in parallel.

3. **Run lint/type checks on domain code** — If failures, send errors back to the relevant domain-builder.

**Gate:** All checkboxes in the tracker's Domain Build phase must be checked before proceeding.

## Phase 3 — Build business layer + UI

**Goal:** Build the business layer, produce the contract, build the UI.

1. **Build the business layer** — You write this code directly in `supabase/functions/business/`. This includes API endpoints, workflow logic, and any cross-domain orchestration. Write tests (unit tests with mocked domain APIs).

2. **Produce the business API contract** — Write the TypeScript types to `ui/src/app/contracts/`. These types define the request/response shapes, error types, and service interfaces the UI will use.

3. **Call the ui-builder** — Send it: (a) the business API contract types, (b) a description of the user-facing behavior to build, (c) the design system / component library context. The ui-builder sees ONLY the contract types — no domain code, no edge function URLs, no DB schema.

4. **Run `npm run precommit-check`** — If it fails, fix business layer issues yourself, send UI issues to the ui-builder.

**Gate:** All checkboxes in the tracker's Business + UI Build phase must be checked before proceeding.

## Phase 4 — Verify

**Goal:** Confirm the feature works end-to-end, get code reviewed.

1. **Write and run Playwright e2e tests** — Write Playwright tests in `e2e/` that walk through the acceptance criteria from the business-rules-expert. Run them against the full running application. Record pass/fail in the tracker.

2. **Run the code-reviewer** — It reviews the diff independently. If it returns BLOCKED, send fixes to the relevant builder and re-review.

3. **Final check** — All tracker checkboxes are filled, all gates passed.

**Gate:** All checkboxes in the tracker's Verify phase must be checked.

## Important principles

- **Record important decisions as ADRs** in `design/`. One file per ADR, concise and specific. Use the ADR template from `strategy.md`.
- **If a subagent fails, diagnose before retrying.** Read the error, understand the cause, adjust your prompt. Never resend the same prompt.
- **If a design-reviewer flags a violation you disagree with**, write an ADR explaining why you're overriding it. Don't silently ignore it.
- **Subagents are authorities in their areas.** If a domain-interface-designer or domain-builder pushes back, engage with their reasoning — don't override.
- **Before breaking any rules**, even if you have a good reason, stop! share the reason with the user and get their approval before proceeding.
- **when in doubt, ask the user, don't guess.**