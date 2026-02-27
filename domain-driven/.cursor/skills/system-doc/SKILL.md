---
name: system-doc
description: System Documentation Generator (Collaborative)
disable-model-invocation: true
---

---
description: Create a system design document for a feature using collaborative multi-agent discussion
---

# System Documentation Generator (Collaborative)

Create comprehensive system documentation through a structured multi-agent discussion process that involves the user at key decision points.

## Process Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  Phase 1: Discovery                                                  │
│  - Gather initial requirements from user                            │
│  - Ask clarifying questions                                         │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│  Phase 2: Parallel Analysis (4 Sub-agents)                          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌─────────────┐│
│  │ Requirements │ │  Technical   │ │   Security   │ │  UX/Product ││
│  │   Analyst    │ │  Architect   │ │   Reviewer   │ │   Reviewer  ││
│  └──────────────┘ └──────────────┘ └──────────────┘ └─────────────┘│
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│  Phase 3: Decision Review                                           │
│  - Present key decisions with options and rationale                 │
│  - Ask user for feedback/approval on each major decision            │
│  - Iterate if user disagrees or has concerns                        │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│  Phase 4: Document Generation                                       │
│  - Generate final system document incorporating all decisions       │
│  - Save to docs/system/[feature-name].md                           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Discovery

Start by understanding what the user wants to build. Ask these questions:

1. **What feature are you designing?** (brief description)
2. **Who are the users/actors involved?**
3. **What problem does this solve?**
4. **Are there any hard constraints?** (tech stack, timeline, budget)
5. **Any existing patterns in the codebase to follow?**

Gather enough context before proceeding to analysis.

---

## Phase 2: Parallel Analysis

Launch **4 sub-agents simultaneously** to analyze different aspects. Each agent should explore the codebase if needed and provide structured output.

### Sub-agent 1: Requirements Analyst
```
Prompt:
You are a Requirements Analyst. Based on the feature description below, analyze and produce:

Feature: [FEATURE_DESCRIPTION]
Context: [USER_CONTEXT]

Tasks:
1. List all Functional Requirements (FR1, FR2, etc.)
2. List all Non-Functional Requirements (NFR1, NFR2, etc.)
3. Identify any ambiguities or gaps - list as questions to ask user
4. Identify edge cases that need handling
5. Search the codebase for similar features to understand patterns

Output format:
## Functional Requirements
- FR1: [requirement]
- FR2: [requirement]
...

## Non-Functional Requirements
- NFR1: [requirement]
...

## Questions for User
- Q1: [question]
...

## Edge Cases
- EC1: [edge case]
...

## Similar Patterns Found
- [file/feature]: [how it's relevant]
```

### Sub-agent 2: Technical Architect
```
Prompt:
You are a Technical Architect. Based on the feature description, design the technical approach:

Feature: [FEATURE_DESCRIPTION]
Tech Stack: Angular 20+, Supabase, PostgreSQL, Deno Edge Functions

Tasks:
1. Propose data model (tables, relationships, types)
2. Identify key technical decisions needed (ITDs)
3. For each ITD, provide 2-3 options with pros/cons
4. Recommend an option with clear rationale
5. Explore the existing codebase to find reusable patterns
6. Identify any new dependencies needed

Output format:
## Proposed Data Model
[Schema description with types]

## Technical Decisions

### ITD 1: [Title]
**Context**: [Why this decision matters]
**Options**:
1. [Option A] - Pros: ... Cons: ...
2. [Option B] - Pros: ... Cons: ...
3. [Option C] - Pros: ... Cons: ...
**Recommendation**: [Option X] because [rationale]

## Reusable Patterns Found
- [pattern]: [where found, how to reuse]

## New Dependencies
- [dependency]: [why needed]
```

### Sub-agent 3: Security Reviewer
```
Prompt:
You are a Security Reviewer. Analyze the feature for security considerations:

Feature: [FEATURE_DESCRIPTION]

Tasks:
1. Identify authentication/authorization requirements
2. Identify data that needs protection (PII, sensitive data)
3. Analyze potential attack vectors
4. Review RLS (Row Level Security) policies needed
5. Check existing security patterns in the codebase

Output format:
## Auth Requirements
- [requirement]

## Data Protection
- [data type]: [protection needed]

## Potential Risks
- Risk: [description] → Mitigation: [approach]

## RLS Policies Needed
- [policy description]

## Security Patterns to Follow
- [existing pattern]: [where found]
```

### Sub-agent 4: UX/Product Reviewer
```
Prompt:
You are a UX/Product Reviewer. Analyze the feature from user perspective:

Feature: [FEATURE_DESCRIPTION]

Tasks:
1. Identify key user flows
2. Identify product decisions needed (IPDs)
3. For each IPD, provide options with user impact analysis
4. Consider accessibility requirements
5. Review existing UI patterns in the codebase

Output format:
## Key User Flows
1. [Flow name]: [step by step]
...

## Product Decisions

### IPD 1: [Title]
**User Problem**: [What user pain point this addresses]
**Options**:
1. [Option A] - User impact: ...
2. [Option B] - User impact: ...
**Recommendation**: [Option X] because [user benefit]

## Accessibility Requirements
- [requirement]

## UI Patterns to Reuse
- [component]: [where found]
```

---

## Phase 3: Decision Review

After sub-agents complete, present findings to user in a structured way:

### Format for User Review

```markdown
## Summary of Analysis

### Key Decisions Requiring Your Input

#### Decision 1: [Title]
**Type**: Technical / Product
**Context**: [Brief context]

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A | ... | ... | ... |
| B | ... | ... | ... |
| **C (Recommended)** | ... | ... | ... |

**Recommendation**: Option C
**Rationale**: [Why this is recommended]

👉 **Your input needed**: Do you agree with Option C? Any concerns?

---

[Repeat for each major decision]

### Questions from Analysis
1. [Question from requirements analyst]
2. [Question from architect]
...

### Identified Risks
1. [Risk] - Proposed mitigation: [approach]
...
```

**Wait for user feedback before proceeding.** If user disagrees with a recommendation:
1. Understand their concern
2. Re-analyze that specific decision
3. Present updated options if needed

---

## Phase 4: Document Generation

Once all decisions are approved, generate the final document:

### Document Template

```markdown
# [Feature Name] System Document

> Generated: [Date]
> Status: Draft | Review | Approved

## Overview
[Brief description of the feature and its purpose]

## Requirements

### Functional Requirements
[From Requirements Analyst]

### Non-Functional Requirements
[From Requirements Analyst]

## Design Decisions

### Product Decisions (IPDs)
[From UX/Product Reviewer - user-approved decisions]

### Technical Decisions (ITDs)
[From Technical Architect - user-approved decisions]

## Data Model
[From Technical Architect]

```typescript
// Type definitions
interface [EntityName] {
  // ...
}
```

```sql
-- Database schema
CREATE TABLE [table_name] (
  -- ...
);
```

## API Design
[Endpoints, request/response formats]

## Sequence Diagrams
[Mermaid diagrams for key flows]

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Database
    ...
```

## Security Considerations
[From Security Reviewer]

### RLS Policies
[Policy definitions]

## Error Handling
[Error scenarios and handling approach]

## Testing Strategy
- Unit tests: [approach]
- Integration tests: [approach]
- E2E tests: [approach]

## Edge Cases
[From Requirements Analyst]

## Future Considerations
[Potential enhancements, known limitations]

---
## Appendix: Decision Log
| ID | Decision | Options Considered | Selected | Rationale |
|----|----------|-------------------|----------|-----------|
| IPD-1 | ... | A, B, C | C | ... |
| ITD-1 | ... | A, B | B | ... |
```

### Save Location
Save the document to: `docs/system/[feature-name].md`

---

## Guidelines

- **Be collaborative**: Always explain reasoning, don't just dictate decisions
- **Ask, don't assume**: When uncertain, ask the user rather than guessing
- **Show your work**: Present options with clear pros/cons so user can make informed decisions
- **Reference the codebase**: Find and cite existing patterns to maintain consistency
- **Iterate**: If user pushes back on a decision, re-analyze and present alternatives
- **Keep it practical**: Focus on decisions that actually matter for implementation
