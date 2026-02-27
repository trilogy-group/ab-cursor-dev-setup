---
name: domain-designer
description: Use after the external spec of the domain is know and before writing domain code. Internal design process for domain-builders. Guides file structure, design guard creation, and internal decomposition decisions before writing implementation code.
disable-model-invocation: true
---

# Domain Designer (Internal Design Skill)

This skill guides the domain-builder's internal design process. It is invoked **after** receiving the external interface spec from the domain-interface-designer, and **before** writing any implementation code.

## When to Use

The domain-builder reads this skill when it needs to:
- Decide how to decompose the interface spec into files and modules
- Write design guards for each file
- Make internal implementation decisions (rare ADRs)

## Input Context

By the time you invoke this skill, you should already have:
1. The **interface spec** — command/query types, events, structural invariants (from domain-interface-designer)
2. The **domain subdirectory** — where to write code
3. The **build request** — what capability or fix to implement

## Procedure

### 1. Decide file structure

Given the interface spec, decide which files to create. Each file should have a single responsibility. Common patterns:

- **Command handler** — one file per command (validate, authorize, write, emit event)
- **Query handler** — one file per query (read from table/read model, return shaped response)
- **Types** — shared type definitions for the domain's API surface (input/output types, event payloads)
- **Validators** — input validation logic, separated from handler orchestration
- **Repository/data access** — database interaction, isolated behind an interface for testability

Do not add layers, factories, wrappers, or patterns that the current scope doesn't require. Build exactly what the interface spec asks for.

### 2. Write design guards

Before writing **any** implementation code in a file, write the `@design-guard` comment block at the top. Every TypeScript file must have one. The guard documents **invariants**, not implementation details. Required keys:

```
/**
 * @design-guard
 * role: <what this file is responsible for>
 * layer: <ui | service | domain | infrastructure>
 * non_goals:
 *   - <what this file must NOT do>
 * boundaries:
 *   depends_on_layers: [<layers this file may depend on>]
 *   exposes: [<public API surface>]
 * invariants:
 *   - <structural truths that must always hold>
 * authority:
 *   decides: [<what this file is the single authority on>]
 *   delegates: [<what it hands off and to whom>]
 * extension_policy:
 *   - <how this file should grow over time>
 * failure_contract:
 *   - <how errors are handled/surfaced>
 * testing_contract:
 *   - <how to test this file>
 */
```

**Design guard rules:**
- Invariants must be structural and stable — they should survive implementation refactors.
- No business rules that change over time (those belong in workflows or configuration).
- No implementation details (specific class names of dependencies, algorithm choices, etc.).
- The guard is a contract. Code beneath it must conform. If you find a conflict, update the guard deliberately, not silently.

### 3. Record implementation ADRs (rare)

If you face a genuinely important internal implementation decision (e.g., choosing between two data structures, or a non-obvious error handling strategy), write an ADR to `design/domains/<domain-name>/implementation/ADR-<name>.md`.

Use the standard ADR template:

```markdown
### Summary statement
### Context
### Problem to solve
### Options
#### Option 1
#### Option 2
### Reasoning
```

Most builds will not need implementation ADRs. Do not manufacture them. The design guards and code should speak for themselves.
