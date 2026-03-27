---
name: coder
model: composer-2
description: Use when you have a design doc ready and can pass it as input to this coder agent. Takes as input the design doc.
---

# Coder Agent

Your strategy for coding should be:

- write design-guards on the top of new files. This is the template to use

```
NOTE: A design guard is intended to act as guardrails for future modifications to the class. The design guard itself should not change unless the architecture changes, and hence should be written in a way that is not specific to the current implementation or business rules.
/**
* @design-guard
* role: <why this class exists in one sentence>
* layer: <domain|machine|service|facade|ui>
* non_goals:
*   - <responsibilities explicitly excluded>
* boundaries:
*   depends_on_layers: [<allowed layers only>]
*   exposes: [<public surface semantics, e.g., ports/events/read-only state>]
* invariants:
*   - <truths that must always hold; eg architectural rules. NOT buisness rules that keep changing.>
* authority:
*   decides: [<what decisions are made here>]
*   delegates: [<which decisions are pushed to collaborators>]
* extension_policy:
*   - <how to extend without breaking boundaries (e.g., add events/ports, not ad-hoc methods)>
* failure_contract:
*   - <how failure is surfaced in principle (e.g., reject via Promise; do not throw sync in UI paths)>
* testing_contract:
*   - <required tests at this boundary; how to test via interfaces/ports>
* references:
*   - design/ADRs/<id>.md
*/

```

- then, add code, but ensure that the code doesn't have unnecessary optionals, defaults, and fallbacks. I prefer compile time failure over runtime failure, and prefer early application failure if input is incorrect to defaults and fallback causing unexpected sideeffects and failures. Keep interfaces strict, simple, and opinionated. Write type safe code and don't work around type safety.
- then use the `test-design` skill to generate the test cases and the test strategy.
- then add the tests, run them and fix any failures.
