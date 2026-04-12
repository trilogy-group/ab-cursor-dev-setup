---
name: build-or-fix
description: Use when you are asked to build or fix something. This handles the end to end stages of building from design, to test to review. Takes as input the DoD (Definition of Done) ie the edsired behaviour from the system.
disable-model-invocation: false
---

# build-or-fix

This skill is an orchestrator. It owns process, delegation, approval gates, and quality bars. It does not own discovery, design, or implementation details when a specialized subagent should do that work.

## Delegation Principle

- Delegate first. Do not pre-solve the problem.
- The `designer` owns repo discovery needed for design.
- The `coder` owns implementation details needed to realize the approved design.
- The orchestrator may do direct investigation only when one of these is true:
  - The user explicitly asks the orchestrator to investigate.
  - There is a blocking business-rule ambiguity that must be clarified before delegation.
  - A subagent failed and needs a narrower retry or a specific missing input.
- Do not narrow the solution space beyond the user's stated requirements and constraints.

## 1. Clarify

- Identify the desired system behavior, constraints, approval requirements, and any non-negotiable rules.
- If the request is attached to a GitHub issue or PR, follow the `github-issue-handling` skill.
- Ask clarifying questions only if the business behavior is materially ambiguous.
- Do not perform broad repo discovery in this step. If the request is clear enough to delegate, delegate immediately.

## 2. Design

- Use the `designer` subagent to create the design proposal.
- Tell it what behavior is required, what constraints apply, and what must be explicitly covered.
- It can and will gather whatever context it needs.
- Do not ask it for code snippets, it has been instructed to work at the design level and knows what to return and in what format. It will put all of that in a .md file
- For production-impacting or architecture-heavy work, follow the **`agent-dojo`** skill to ground the design in practitioner knowledge (failure modes, trade-offs, `gotchas` / `architecture_patterns`). Use MCP `ask_dojo` or `search_knowledge` when available; if Dojo is unavailable, proceed but note the gap for the reviewer.

### Approval Gate

- Share the design with the user.
- If this work is associated with a GitHub issue or PR, present the design using the `github-issue-handling` skill.
- Do not proceed to implementation until the user approves the design.

## 3. Implement

- After approval, use the `coder` subagent to implement the design.
- Pass the approved design, repository constraints.
- The `coder` may inspect the repository as needed while implementing.
- If implementation exposes a material gap in the approved design, stop and return to design instead of silently improvising a new plan.

## 4. Verify

- Use the `precommit-check` subagent to run the precommit checks and fix any errors.
- Use the `code-reviewer` subagent to review the code. Share the code with it and ask it to review the code.
- Optionally augment review with **`agent-dojo`**: `search_knowledge` on dimensions `anti_patterns`, `gotchas`, or `security_practices` for technologies changed in the diff (faster than a full `ask_dojo` synthesis).
- Address any issues raised by the reviewer before proceeding.

## 5. Evidence

A PR without evidence is incomplete. A reviewer should not need to run the code to understand why the change is correct.

- Before considering the work done, follow the instructions in `evidence-gatherer.md` in this directory.
- Capture evidence that demonstrates the approved behavior, not just that commands passed.

## Escalation Rules

- If the `designer` says more context is needed, prefer one of these:
  - ask the user a focused clarifying question
  - run a narrow follow-up exploration requested by the `designer`
- If the `coder` identifies a design flaw, return to the user with the design gap rather than silently compensating.
- If verification or review reveals that the approved design was wrong, stop and surface that explicitly.
