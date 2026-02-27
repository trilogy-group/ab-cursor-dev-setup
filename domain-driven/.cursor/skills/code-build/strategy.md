## Recommended target architecture strategy (high-level guidance)

### Design philosophy

This system is built for a **federated builder workforce**. Every architectural decision serves one overriding goal: **builders have free reign over their layer's implementation, and the blast radius of any one builder's errors in judgement is minimized.**

### Three layers

```
UI layer          → User-facing pages, components, guards, routes
                     Codes against the business API contract only.
                     Knows nothing about domains, DB, or edge functions.

Business layer    → Business logic, workflows, orchestration, API surface for the UI
                     Calls domain APIs. Owns cross-domain rules, state management
                     decisions, and the API contract the UI consumes.

Domain layer      → Backend capabilities (schema, commands, queries, events)
                     Enforces structural invariants. Knows nothing about UI or
                     business rules.
```

Each layer has strict boundaries enforced by folder structure:

```
supabase/
├── functions/
│   ├── _shared/                    # Shared infra (auto-generated DB types) — read-only for all
│   ├── _domains/                   # Domain layer source code — NOT a deployed function
│   │   └── <domain>/              # e.g. profile/ — owned exclusively by domain-builder
│   │       ├── handlers/          # Command and query handler implementations
│   │       ├── types.ts           # Domain types (internal to domain)
│   │       └── tests/             # Domain unit + integration tests
│   ├── business/                   # Business layer — deployed as an edge function
│   │   ├── index.ts               # Entry point: routing, auth, CORS
│   │   ├── routes/                # Route handlers that call into _domains/
│   │   └── tests/                 # Business layer tests (mocked domain calls)
│   └── deno.json                   # Shared Deno config
├── migrations/                     # SQL migrations

ui/src/app/
├── contracts/                      # Business API contract types — orchestrator produces, ui-builder reads
├── pages/                          # Page components
├── guards/                         # Route guards
├── services/                       # Angular services (API clients implementing contract interfaces)
├── auth/                           # Auth components and services
├── components/                     # Shared UI components
└── types/                          # UI-only types

e2e/                                # Playwright e2e tests — orchestrator owns
```

**Why `_domains/` has the underscore prefix:** Supabase treats folders starting with `_` as shared libraries, not deployable functions. Domain code is a library imported by the `business/` edge function — it's never deployed as a standalone function. This also makes it visually obvious: `_` prefix = library code, no prefix = deployed function.

**Layer boundary rules:**
- **Domain layer** code lives in `supabase/functions/_domains/<domain>/`. Each domain-builder owns its domain folder exclusively. Domain code is imported by the business layer at build time.
- **Business layer** code lives in `supabase/functions/business/`. The code-build orchestrator owns this folder. It is the single deployed edge function the UI calls. It imports from `_domains/` and `_shared/`.
- **UI layer** code lives in `ui/src/app/`. The UI builder owns this folder exclusively. It imports only from `ui/src/app/contracts/` for backend types.
- **Contract types** live in `ui/src/app/contracts/`. The orchestrator produces them; the UI builder consumes them read-only. No domain types, DB types, or edge function URLs leak into this folder.
- **E2e tests** live in `e2e/`. The orchestrator owns this folder.

---

### Core principles

- **Domains are capability providers, not business-rule gatekeepers.** A domain should offer robust, general-purpose capabilities and enforce only its own *structural invariants* (data integrity, ownership, uniqueness). It should not embed behavioral business rules that belong to a higher-level process.
- **The business layer owns cross-domain logic.** Eligibility checks, multi-step prerequisites, conditional logic that spans domains — all of this lives in the business layer, never scattered across domain handlers. This keeps domains reusable and independently evolvable.
- **The UI is domain-unaware.** UI components never import domain types, call domain APIs, or know about DB schema. They code against the business API contract — a set of TypeScript types and service interfaces.
- **Loose coupling and isolation are non-negotiable.** Domains communicate through APIs and events, never through shared tables and rarely have cross-schema joins.

---

### Domain layer principles

- **Domains are bounded contexts**: each domain owns its **schema**, **command API**, **query API**, **structural invariants**, and **events**.
- **Database access is API-only**: UI does not read/write tables directly. Domain APIs run with **domain DB roles** limited to their schemas.
- **CQRS by default**:
  - **Commands**: validate + authorize + write domain tables; emit domain events.
  - **Queries**: read from **read models** shaped for consumers.

---

### Business layer principles

The business layer is the bridge between UI needs and domain capabilities. Not everything in this layer is a workflow. The key decisions for each capability are:

- **Passthrough** — the UI need maps 1:1 to a single domain command/query. The business layer endpoint just calls the domain API and returns the result. This is the default and most common case.
- **Composition** — the UI need requires data from multiple domains. The business layer calls multiple domain query APIs and composes the response.
- **Short-lived workflow** — the operation requires multiple domain commands in sequence within a single request (e.g., create profile then send welcome email).
- **Long-lived workflow** — the process spans time and external/human latency. Backed by a durable state machine.

#### Durable workflows

- A **process state machine** represents durable progress (days/weeks).
- Each state transition is executed by an **idempotent transition workflow** that calls domain commands and waits on events/timeouts.
- Stored state is **process state**, not duplicated domain truth.

#### Storage contracts (when workflows need persistence)
- `process_instances`: process id, current state, version, correlation ids.
- `process_events` (optional): append-only event log for replay/debug.
- `process_timers`: scheduled wakeups.
- `domain_outbox`: domain-emitted events used to update read models and signal processes.

#### Idempotency contract
- Every transition workflow is safe to retry using idempotency keys, correlation ids, and conditional updates on process version.

---

### Cross-domain read models: ownership

Cross-domain projections span domain boundaries by design. Ownership rules:

- The **consuming query API owns the read-model schema.**
- **Projectors** that populate these tables subscribe to domain outbox events and write into the read-model schema. The projector is part of the consuming query API's codebase, not the source domain's.
- Source domains are **unaware** of downstream projections. They emit events to their own outbox; they never write to read-model schemas they don't own.

---

### Folder ownership

| Folder | Owner | May not touch |
|---|---|---|
| `supabase/functions/_domains/<domain>/` | domain-builder (one per domain) | business layer, UI, other domains |
| `supabase/functions/_shared/` | auto-generated / read-only | — |
| `supabase/functions/business/` | code-build orchestrator | domain internals, UI |
| `supabase/migrations/` | db-resetter (applies); domain-interface-designer (designs DDL) | — |
| `ui/src/app/contracts/` | code-build orchestrator (produces); ui-builder (consumes read-only) | domain code |
| `ui/src/app/` (everything else) | ui-builder | domain code, business layer internals |
| `e2e/` | code-build orchestrator (Playwright tests) | — |

---

### Summary rules of thumb
- **Default**: CQRS with API-only DB access. UI calls business layer; business layer calls domains.
- **Introduce projections** only for hot reads and lists; keep projection scope tight and page-shaped.
- **Use durable workflows** for processes with human/external latency and cross-domain coordination.
- **Model durable workflows as state machines**; implement transitions as idempotent workflows calling domain commands.

---

## ADR Template

Use this template for Architecture Decision Records in the `design/` folder. One file per ADR. Only record important decisions.

```markdown
### Summary statement
<!-- "Use approach ABC to solve problem XYZ." Summarizes the decision; doesn't need to cover reasoning. -->

### Context
<!-- What does the reader need to know to understand why this decision is necessary? -->

### Problem to solve
<!-- Why is this decision necessary? -->

### Options
#### Option 1
#### **Option 2 (selected)**
#### Option 3

### Reasoning
<!-- Avoid "simple/complex/easy/difficult" — explain WHY it is simpler/easier. -->
<!-- Don't list pros and cons. Highlight one or two deciding factors. -->
```
