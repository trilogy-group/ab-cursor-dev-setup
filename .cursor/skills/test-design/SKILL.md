---
name: test-design
description: Use when asked to design test cases and tests
disable-model-invocation: false
---

## Testing Design

This project uses a **pragmatic testing strategy** that emphasizes:

- ✅ **Unit tests** for pure business logic (fast, no dependencies)
- ✅ **Integration tests** for database and external services (real dependencies)
- ✅ **Dependency injection** for testability
- ✅ **Mock at boundaries** (external APIs, not internal services)
- ✅ **Test behavior, not implementation**

Think Blackbox testing and identify the different components that need to be tested. For each list the test cases, and whether each should be unit/integration/e2e tested.
Then list any important questions that arise to implement tests (including mocking strategy if necessary), and your recommended answers to those questions.
DO not generate any code yet, stop with just providing the above.

## Test Investment Priority

Prioritize test effort in this order:

1. **High-value features / happy paths** — the core flows users depend on
2. **Error and failure paths** — invalid input, permission failures, timeout handling
3. **Edge cases and boundary conditions** — empty states, max limits, concurrent access
4. **Integration boundaries** — real DB constraints, RLS policies, external API contracts
5. **Low-risk / cosmetic paths** — last priority, skip if time-constrained

Do not chase 100% code coverage as a goal. Use coverage as a diagnostic to find important untested paths, not as a metric to satisfy. High coverage with weak assertions is worse than moderate coverage with strong behavioral checks.

## Anti-Patterns to Avoid

- ❌ **Testing implementation details**: If a test breaks when you rename a function, refactor internal state, or reorder harmless internals — it's coupled to implementation, not behavior. Test rendered output, saved data, emitted side effects, and observable state changes.
- ❌ **Mocking everything**: Mock only at boundaries you own (external APIs, time, randomness). Do not mock internal services or child components just to isolate — you lose integration confidence.
- ❌ **Snapshot-testing complex output**: Snapshots are brittle for anything beyond trivial serialization. Prefer explicit assertions on the properties that matter.
- ❌ **Tests that pass but prove nothing**: Calling a function and asserting it returns "something truthy" or asserting a mock was called without checking the result is a coverage-padding exercise, not a test.
- ❌ **Time-dependent or order-dependent assertions**: These cause flaky tests. Inject time/clock dependencies. Don't rely on execution order between tests. Each test must set up its own state.

## Mock Boundary Rules

- Mock **external HTTP APIs** — you don't control their availability or response time
- Mock **time/clock** — use injectable clock for anything time-sensitive
- Mock **randomness** — seed or inject for deterministic tests
- Do NOT mock **your own database** in integration tests — use a real local DB (Supabase local)
- Do NOT mock **internal services** you own — test through the real call path
- Do NOT mock **the function under test** — if you need to mock internals to test it, the function has too many responsibilities
