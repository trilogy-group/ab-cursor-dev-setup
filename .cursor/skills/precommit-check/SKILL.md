---
name: precommit-check
description: precommit-check
disable-model-invocation: true
---

# precommit-check

Run all precommit checks in parallel and fix any errors.

## Strategy

Run the following checks in **parallel** using sub-agents (Task tool). Each sub-agent should run its assigned check and fix any errors it finds.

### Phase 1: Run checks in parallel (4 sub-agents max at a time)

**Batch 1** - Launch these 4 sub-agents simultaneously:

1. **TypeScript & Design Guards** - Run `npm run typecheck && npm run check:design-guards && npm run check:design-system` in `ui/` directory. Fix any type errors or design guard violations.
2. **Linting** - Run `npm run lint:all && npm run lint:styles` in `ui/` directory. Fix any lint errors (do not disable rules).
3. **Unit Tests** - Run `npm run test:unit` in `ui/` directory. Fix any failing tests.
4. **Edge Functions** - Run `npm run edge:fmt:fix && npm run edge:check` in `ui/` directory. Fix any edge function issues.

**Batch 2** - After batch 1 completes: 5. **Build & Format** - Run `npm run build:check && npm run format` in `ui/` directory. Fix any build errors.

### Phase 2: Verify

After all sub-agents complete, run the full `npm run precommit-check` once to verify everything passes.

## Important Guidelines

- **Respect type safety** - Do not use `any`, `@ts-ignore`, or disable type checking
- **Respect lint rules** - Do not disable ESLint rules; fix the underlying issue
- **Respect design guards** - Do not modify `@design-guard` invariants without careful consideration
- **Fix root causes** - Address the actual problem, not just the symptom

## Sub-agent Prompt Template

Each sub-agent should receive a prompt like:

```
Run the following commands in the ui/ directory:
[COMMANDS]

If any errors occur:
1. Read the error messages carefully
2. Fix the underlying issues in the code
3. Re-run the commands to verify the fix
4. Report back what was fixed or if issues remain

Do NOT disable type checking, lint rules, or use workarounds. Fix the actual problems.
```
