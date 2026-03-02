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
Then list any important questions that arise implment tests (including mocking strategy if necessary), and your recommended answers to those questions.
DO not generate any code yet, stop with just providing the above.
