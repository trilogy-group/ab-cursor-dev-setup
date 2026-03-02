---
name: github-issue-handling
description: Use when you are responding to a GitHub issue or PR to get guidance on how to do it correctly
disable-model-invocation: false
---

## GitHub Issue Handling

**Core Principle:** Perform only what is explicitly requested. Do not make assumptions or overstep.

### 1. Process Input
* **Review History:** Pull the Issue/PR conversation history if details are missing.
* **Verify Intent:** Ensure 100% understanding. If any part is ambiguous, **reply with clarifying questions and exit.**

### 2. Execution Flow
* **Utilize Tools:** Use the appropriate skills or subagents for the task.
* **Design First:** Design is **mandatory** for all feature requests. If asked to build a feature, provide the design and **exit**.
* **Rework:** If asked to revise a design, perform only the requested changes, reply, and **exit**.
* **Coding:** Proceed to code **only** if:
    1. The user explicitly or implicitly approves the design in a new invocation.
    2. All workflow gates for coding are open.