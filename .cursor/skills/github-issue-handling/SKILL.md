---
name: github-issue-handling
description: Use when you are responding to a GitHub issue or PR to get guidance on how to do it correctly
disable-model-invocation: false
---

## GitHub Issue Handling

**Core Principle:** Perform only what is explicitly requested. Do not make assumptions or overstep.

### 1. Process Input

- **Review History:** Pull the Issue/PR conversation history if details are missing.
- **Verify Intent:** Ensure 100% understanding. If any part is ambiguous, **reply with clarifying questions and exit.**

### 2. Execution Flow

- **Utilize Tools:** Use the appropriate skills or subagents for the task.
- **Design First:** Design is **mandatory** for all feature requests. If asked to build a feature, provide the design and **exit**.
- **Rework:** If asked to revise a design, perform only the requested changes, reply, and **exit**.
- **Coding:** Proceed to code **only** if:
  1. The user explicitly or implicitly approves the design in a new invocation.
  2. All workflow gates for coding are open.

## Responding to the user

Your last message is shown to the user when you push the branch. So if you need to share the design, then do so by pushing the barnch (even if there are no new files) and then printing the entire design (unchanged) as your last message. Do not summarize or modify the design. At the end of the design say "Let me know if I should proceed with this design, or if you need changes by adding a comment `@cursor <your message>`"
