---
name: build-or-fix
description: Use when you are asked to build or fix something. This handles the end to end stages of building from design, to test to review. Takes as input the DoD (Definition of Done) ie the edsired behaviour from the system.
disable-model-invocation: false
---

# build-or-fix

Your strategy for building should be:

## 1. Clarify

- Check if you have clarity on what the desired system behaviour is. If not, ask the user for clarification. DO NOT PROCEED UNTIL YOU HAVE CLARITY ON THE DESIRED SYSTEM BEHAVIOUR. If you are working on a Github issue or PR then post the questions to the issue or PR comments.

## 2. Design

- Use the `designer` subagent to put together a design proposal. Tell it "what" behaviour is required, not "how" to implement/design it.
- **Approval gate:** If this work is associated with a GH issue or PR, post the design summary as a comment using `gh`. Then STOP and tell the user you are waiting for approval. Do not proceed to implementation until the user approves. The motivation: building the wrong thing wastes everyone's time.

## 3. Implement

- Use the `coder` subagent to implement the approved design. Share the proposed design with it and ask it to implement the design.

## 4. Verify

- Use the `precommit-check` subagent to run the precommit checks and fix any errors.
- Use the `code-reviewer` subagent to review the code. Share the code with it and ask it to review the code.
- Address any issues raised by the reviewer before proceeding.

## 5. Evidence

A PR without evidence is an incomplete PR. A reviewer shouldn't have to run the code to know it works. Before considering the work done, follow the instructions in `evidence-gatherer.md` (in this same directory) to capture and post evidence to the PR.
