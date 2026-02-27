# Capability: <name>

**Call log:** [<name>-calls.md](./<name>-calls.md)

## Phase 1 — Design

### Problem Statement
<!-- What capability or fix is needed? What user-facing behavior should change? Keep this to 2-3 sentences. -->


### Business Rules
<!-- Paste the acceptance criteria from the business-rules-expert here. These become your e2e test cases. -->


### Questions
<!-- Questions that arise when deciding how to implement this. Answer each with reasoning. -->

1. **<question>**
   → <answer>


### Sequence Diagram
<!-- Behavioral flow across all three layers: UI → business → domain. Show WHICH layers/domains interact and WHAT happens, not payload shapes or types. -->

```mermaid
sequenceDiagram
```

### Domain Interface Specs
<!-- BLANK until domain-interface-designers fill them in. One subsection per domain. -->

#### <Domain Name>
**Designed by:** domain-interface-designer
**Status:** [ ] Received

<!-- Paste the domain-interface-designer's full response here: types, events, DDL, boundary evaluation. -->


### Business API Contract
<!-- Designed by the orchestrator. TypeScript types defining what the UI will call. For each endpoint, state the pattern: passthrough | composition | short-lived workflow | long-lived workflow. -->

| Endpoint | Pattern | Domain APIs called |
|---|---|---|
| <e.g. getMyProfile> | passthrough | profile.getProfile |
| <e.g. completeOnboarding> | passthrough | profile.completeOnboarding |

```typescript
// Contract types to be written to ui/src/app/contracts/
```

**Status:** [ ] Designed


### Design Review
**Reviewer:** design-reviewer
**Status:** [ ] Approved (zero Blockers, zero unresolved Majors)

<!-- Paste the design-reviewer's verdict and violations here. If violations were overridden, link the ADR. -->


### Design Phase Gate
- [ ] Business rules and acceptance criteria captured
- [ ] Questions answered
- [ ] Sequence diagram shows UI → business → domain interactions
- [ ] Domain interface specs received from domain-interface-designer(s) (not self-authored)
- [ ] Business API contract designed (endpoint patterns decided)
- [ ] Design-reviewer verdict: APPROVED

---

## Phase 2 — Build Domains

### Migrations
**Status:** [ ] Applied and types regenerated

<!-- List migration files created and db-resetter result. If it failed, record the diagnosis. -->


### Domain Builds
<!-- One subsection per domain-builder invocation. -->

#### <Domain Name>
**Status:** [ ] Complete
**Subdirectory:** `supabase/functions/_domains/<domain>/`

<!-- Record what was sent to the builder and its result summary. -->


### Domain Build Phase Gate
- [ ] Migrations applied, types regenerated
- [ ] All domain-builders completed
- [ ] Domain code precommit-check pass

---

## Phase 3 — Build Business Layer + UI

### Business Layer
**Status:** [ ] Complete
**Subdirectory:** `supabase/functions/business/`

<!-- Record what was built: endpoints, workflow logic, tests. -->


### Business API Contract Types
**Status:** [ ] Written to `ui/src/app/contracts/`

<!-- List the contract type files produced. -->


### UI Build
**Builder:** ui-builder
**Status:** [ ] Complete
**Subdirectory:** `ui/src/app/`

<!-- Record what was sent to the ui-builder and its result summary. The ui-builder receives ONLY the contract types and a behavior description. -->


### Business + UI Build Phase Gate
- [ ] Business layer built and tested (unit tests with mocked domain APIs)
- [ ] Contract types written to `ui/src/app/contracts/`
- [ ] UI-builder completed
- [ ] `npm run precommit-check` passes (lint, types, build, format)

---

## Phase 4 — Verify

### Playwright E2E Tests
**Location:** `e2e/`

<!-- For each acceptance criterion, record the test and result. -->

| Acceptance criterion | Test file | Result | Notes |
|---|---|---|---|
| <Given X, when Y, then Z> | `e2e/<test>.spec.ts` | [ ] Pass | |

### Code Review
**Reviewer:** code-reviewer
**Status:** [ ] Approved (zero Blockers)

<!-- Paste the code-reviewer's verdict and violations here. -->


### Verify Phase Gate
- [ ] Playwright e2e tests pass for all acceptance criteria
- [ ] Code-reviewer verdict: APPROVED
- [ ] All sections of this tracker are filled in
