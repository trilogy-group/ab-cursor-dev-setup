---
name: design
description: design
disable-model-invocation: true
---

# design

Present a design proposal in the form of:
- A list of questions that arise when deciding how to implement this. And your recommended answers with reasoning.
- A mermaid sequence diagram at the class level showing the flow of data and events along with their shapes.
- Design-guard text for any new classes proposed, or modifications if necessary to existing design-guards. Note design-guards should be invariants and not change with implementation, they should be an umbrella under which the implementation is free to change.
- Do not generate any code yet

### Failure Mode Analysis

For every critical flow, include a failure mode table:

| Component | Failure mode | Symptom | Detection | Mitigation | User impact |
|---|---|---|---|---|---|
| (fill in) | (fill in) | (fill in) | (fill in) | (fill in) | (fill in) |

Consider partial, gray, and asymmetric failures — not just "service up/down." Ask: "What if dependency X is slow but not fully down?" and "What if the deployment partially succeeds?"

### Rollback Strategy

For every change: state how to undo it. If irreversible, state that explicitly. Cover schema changes, feature flags, and data format compatibility.

### Capacity Check

For any new data path: what happens at 10x load? Where is the single point of failure? What resource bottlenecks first?
