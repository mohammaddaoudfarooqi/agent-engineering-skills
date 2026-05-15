# Specification Templates

## Table of Contents
- [Workflow Selection Guide](#workflow-selection-guide)
- [Greenfield: Feature Spec](#greenfield-feature-spec)
- [Brownfield: Enhancement Spec](#brownfield-enhancement-spec)
- [Brownfield: Refactor Spec](#brownfield-refactor-spec)
- [Brownfield: Bugfix Spec](#brownfield-bugfix-spec)
- [Codebase Context Template](#codebase-context-template)
- [Shared: design.md (Greenfield)](#shared-designmd-greenfield)
- [Shared: design-delta.md (Brownfield)](#shared-design-deltamd-brownfield)
- [Shared: tasks.md](#shared-tasksmd)
- [EARS Notation Reference](#ears-notation-reference)
- [Requirement Quality Checklist](#requirement-quality-checklist)

---

## Workflow Selection Guide

| Scenario | Spec Files to Create | Template |
|----------|---------------------|----------|
| New project, no code exists | requirements.md + design.md + tasks.md | Feature |
| Add capability to existing project (tests exist) | requirements.md + design-delta.md + tasks.md | Enhancement |
| Add capability to existing project (no tests) | requirements.md + design-delta.md + tasks.md | Enhancement (with Test Infrastructure section) |
| Restructure without behavior change (tests exist) | refactor.md + tasks.md | Refactor |
| Restructure without behavior change (no tests) | refactor.md + tasks.md | Refactor (with Test Infrastructure section) |
| Fix a bug in existing code | bugfix.md + tasks.md | Bugfix |

---

## Greenfield: Feature Spec

### requirements.md

```markdown
# Feature: [Name]

## Overview
[1-2 sentence description of the feature and its purpose]

## User Stories

### US-1: [User Story Title]
**As a** [role]
**I want** [capability]
**So that** [benefit]

#### Acceptance Criteria (EARS Notation)
- REQ-001: WHEN [condition/event] THE SYSTEM SHALL [expected behavior]
- REQ-002: WHEN [condition/event] THE SYSTEM SHALL [expected behavior]

### US-2: [User Story Title]
**As a** [role]
**I want** [capability]
**So that** [benefit]

#### Acceptance Criteria (EARS Notation)
- REQ-003: WHEN [condition/event] THE SYSTEM SHALL [expected behavior]

## Non-Functional Requirements
- REQ-NF-001: THE SYSTEM SHALL [performance/security/accessibility constraint]
- REQ-NF-002: THE SYSTEM SHALL [constraint with measurable threshold]

## Edge Cases
- REQ-EC-001: WHEN [edge condition] THE SYSTEM SHALL [handling behavior]
- REQ-EC-002: IF [boundary condition] THEN THE SYSTEM SHALL [handling behavior]

## Out of Scope
- [Feature/behavior explicitly excluded]

## Open Questions
- [NEEDS CLARIFICATION]: [Unresolved question]
```

---

## Brownfield: Enhancement Spec

### requirements.md (enhancement)

```markdown
# Enhancement: [Name]

## Overview
[1-2 sentence description of what is being added/changed and why]

## Codebase Context
- **Affected components:** [existing files/modules this change touches]
- **Existing behavior:** [what the system currently does in this area]
- **Closest analog:** [existing feature most similar to this change]
- **Prior spec:** [link to existing requirements.md if any, or "None"]

## New/Changed Requirements

### [User Story or Change Description]

#### New Behavior (EARS Notation)
- REQ-E-001: WHEN [condition] THE SYSTEM SHALL [new behavior]
- REQ-E-002: WHEN [condition] THE SYSTEM SHALL [new behavior]

#### Modified Behavior (what changes from current)
- REQ-E-003: WHEN [condition] THE SYSTEM SHALL [changed behavior]
  - **Was:** [previous behavior]
  - **Now:** [new behavior]
  - **Reason:** [why the change]

## Non-Functional Requirements
- REQ-E-NF-001: THE SYSTEM SHALL [constraint]

## Edge Cases
- REQ-E-EC-001: IF [edge condition] THEN THE SYSTEM SHALL [handling]

## Unchanged Behavior (Invariants)
- INV-001: WHEN [condition] THE SYSTEM SHALL CONTINUE TO [existing behavior]
- INV-002: WHEN [condition] THE SYSTEM SHALL CONTINUE TO [existing behavior]
[List every existing behavior that could plausibly be affected by this change]

## Out of Scope
- [Explicitly excluded from this iteration]

## Open Questions
- [NEEDS CLARIFICATION]: [Unresolved question]
```

---

## Brownfield: Refactor Spec

### refactor.md

```markdown
# Refactor: [Name]

## Overview
[What is being restructured and why (performance, maintainability, tech debt)]

## Codebase Context
- **Affected components:** [files/modules being refactored]
- **Current structure:** [how it's organized now]
- **Target structure:** [how it should be organized after]

## Behavior Preservation Contract
THE SYSTEM SHALL MAINTAIN ALL EXISTING EXTERNAL BEHAVIOR UNCHANGED.
No user-visible behavior changes unless explicitly listed below.

## Structural Changes
- SC-001: [Component A] SHALL BE [restructured how]
  - **Current:** [current structure/pattern]
  - **Target:** [new structure/pattern]
  - **Verification:** [how to confirm behavior unchanged]

- SC-002: [Component B] SHALL BE [restructured how]
  - **Current:** [current structure/pattern]
  - **Target:** [new structure/pattern]
  - **Verification:** [how to confirm behavior unchanged]

## Intentional Behavior Changes (if any)
- REQ-R-001: WHEN [condition] THE SYSTEM SHALL [changed behavior]
  - **Was:** [previous behavior]
  - **Now:** [new behavior]
  - **Reason:** [why this behavior change is acceptable]

## Unchanged Behavior (Invariants)
- INV-001: WHEN [condition] THE SYSTEM SHALL CONTINUE TO [behavior]
- INV-002: WHEN [condition] THE SYSTEM SHALL CONTINUE TO [behavior]
[For refactors, this section should be extensive — list ALL key behaviors]

## Success Criteria
- [ ] All existing tests pass without modification
- [ ] No user-visible behavior changes (unless listed above)
- [ ] [Specific improvement metric, e.g., "Response time < 100ms"]

## Out of Scope
- [What is NOT being refactored in this iteration]

## Test Infrastructure (include ONLY if no tests exist)
- **Framework:** [chosen framework and version]
- **Rationale:** [why this framework — stack compatibility, team convention, etc.]
- **Runner command:** [e.g., `npm test`, `pytest`, `go test ./...`]
- **Config file:** [e.g., `vitest.config.ts`, `pytest.ini`]
- **Test location:** [co-located with source | separate `tests/` directory]
- **Naming convention:** [e.g., `*.test.ts`, `test_*.py`, `*_test.go`]
- **Characterization test scope:** [ALL externally observable behavior of refactored code — be thorough]
```

---

## Brownfield: Bugfix Spec

### bugfix.md

```markdown
# Bugfix: [Bug Title]

## Bug Analysis

### Current Behavior (Defect)
- WHEN [triggering condition] THEN the system [incorrect behavior observed]

### Expected Behavior (Correct)
- WHEN [triggering condition] THEN the system SHALL [correct behavior]

### Unchanged Behavior (Regression Prevention)
- WHEN [related condition A] THE SYSTEM SHALL CONTINUE TO [existing behavior A]
- WHEN [related condition B] THE SYSTEM SHALL CONTINUE TO [existing behavior B]

## Root Cause Analysis
[After investigation: what causes the bug and why]

## Affected Components
- [File/module where the bug lives]
- [Related files that might be affected by the fix]

## Proposed Fix
[Technical approach — minimal change to fix the root cause]

## Test Plan
| Test | Purpose | Status |
|------|---------|--------|
| TC-BUG-001 | Reproduce: confirm bug exists before fix | [ ] |
| TC-FIX-001 | Validate: confirm fix resolves the issue | [ ] |
| TC-REG-001 | Regression: unchanged behavior A preserved | [ ] |
| TC-REG-002 | Regression: unchanged behavior B preserved | [ ] |

## Tasks
1. Write reproduction test (TC-BUG-001) — confirm it FAILS showing the bug
2. Write fix validation test (TC-FIX-001) — confirm it FAILS before fix
3. Implement the fix (minimal change)
4. Confirm TC-FIX-001 passes, TC-BUG-001 now shows correct behavior
5. Write and run regression tests (TC-REG-*)
6. Confirm full test suite passes
```

---

## Codebase Context Template

Use this when the user requests a written context document for their project
(optional — Phase 0 can produce this mentally without writing a file).

```markdown
# Codebase Context: [Project Name]

## Tech Stack
- **Language:** [language and version]
- **Framework:** [framework and version]
- **Build:** [build tool and command]
- **Test:** [test framework, runner command]
- **Lint:** [linter and command]
- **Package manager:** [npm/pip/cargo/etc.]

## Architecture
- **Pattern:** [MVC, layered, microservices, etc.]
- **Key directories:**
  - `src/` — [description]
  - `tests/` — [description]
  - `config/` — [description]

## Conventions
- **File naming:** [pattern, e.g., kebab-case.ts]
- **Function naming:** [pattern, e.g., camelCase]
- **Test naming:** [pattern, e.g., test_feature_scenario]
- **Import style:** [pattern]

## Key Entities
- [Entity A]: [brief description and location]
- [Entity B]: [brief description and location]

## Existing Specs
- [Link to requirements.md if exists, or "None"]
- [Link to design.md if exists, or "None"]
```

---

## Shared: design.md (Greenfield)

```markdown
# Design: [Feature Name]

## Architecture Overview
[High-level description of the technical approach and component boundaries]

## Data Flow
[Description or diagram of how data moves through the system]

## Component Design

### [Component Name]
- **Purpose:** [What this component does]
- **Interfaces:** [APIs, events, or contracts it exposes]
- **Dependencies:** [What it depends on]

## Data Models

### [Entity Name]
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | string/uuid | PK, required | Unique identifier |
| [field] | [type] | [constraints] | [description] |
| createdAt | datetime | required | Creation timestamp |
| updatedAt | datetime | required | Last update timestamp |

## API Endpoints
| Method | Path | Description | Request Body | Response | Auth |
|--------|------|-------------|-------------|----------|------|
| GET | /api/[resource] | [description] | N/A | [type][] | [req?] |
| POST | /api/[resource] | [description] | [type] | [type] | [req?] |

## Error Handling Strategy
| Error Scenario | HTTP Status | Response | Recovery |
|---------------|-------------|----------|----------|
| [scenario] | [code] | [error body] | [what happens] |

## Testing Strategy
- **Properties to verify:** [invariants from requirements]
- **Integration points:** [external systems, databases]
- **Edge cases:** [from requirements edge cases section]

## Dependencies
- [Library/service]: [version] — [purpose]
```

---

## Shared: design-delta.md (Brownfield)

```markdown
# Design Delta: [Enhancement/Refactor Name]

## Current Architecture (Relevant Portion)
[Brief summary of existing architecture in the affected area.
Reference existing design.md if it exists rather than rewriting.]

## Changes

### Modified Components
| Component | Current Behavior | Change | Reason |
|-----------|-----------------|--------|--------|
| [component] | [what it does now] | [what changes] | [why] |

### New Components
| Component | Purpose | Interfaces | Integrates With |
|-----------|---------|-----------|-----------------|
| [component] | [purpose] | [APIs/events] | [existing components] |

### Files Impact
| File | Action | Description |
|------|--------|-------------|
| src/[existing].ts | MODIFY | [what changes] |
| src/[new].ts | CREATE | [what it does] |
| tests/[new].test.ts | CREATE | [what it tests] |

## Data Model Changes
| Entity | Change Type | Details |
|--------|-------------|---------|
| [entity] | ADD FIELD | [field]: [type] — [purpose] |
| [entity] | MODIFY FIELD | [field]: [old type] → [new type] |
| [new entity] | CREATE | [description] |

## API Changes (if applicable)
| Method | Path | Change | Details |
|--------|------|--------|---------|
| POST | /api/[resource] | NEW | [description] |
| GET | /api/[resource] | MODIFY | [what changes] |

## Change Impact Analysis
Before implementing, verify:
1. What files/functions does this change directly modify?
2. What code calls the modified functions?
3. What modules import the modified modules?
4. What existing tests cover the modified code paths?
5. Does this change any public API signatures or behavior?
6. Does this change any data schemas or serialization formats?
7. Does this change any configuration or environment expectations?

## Migration / Backwards Compatibility
[How to handle the transition. Any data migration needed?
Any API versioning needed? Any feature flags?]

## Testing Strategy Delta
- **New tests for:** [new behavior from requirements]
- **Regression tests for:** [invariants from requirements]
- **Existing tests to verify:** [list existing test files to run]

## Test Infrastructure (include ONLY if no tests exist)
- **Framework:** [chosen framework and version]
- **Rationale:** [why this framework — stack compatibility, team convention, etc.]
- **Runner command:** [e.g., `npm test`, `pytest`, `go test ./...`]
- **Config file:** [e.g., `vitest.config.ts`, `pytest.ini`]
- **Test location:** [co-located with source | separate `tests/` directory]
- **Naming convention:** [e.g., `*.test.ts`, `test_*.py`, `*_test.go`]
- **Characterization test scope:** [what existing behavior will be characterized]
```

---

## Shared: tasks.md

```markdown
# Tasks: [Feature/Enhancement/Refactor Name]

## Traceability Matrix
| Req ID | Requirement Summary | Test Case IDs | Task | Status |
|--------|-------------------|---------------|------|--------|
| REQ-E-001 | [summary] | TC-E-001, TC-E-002 | Task 1 | Not Started |
| REQ-E-002 | [summary] | TC-E-003 | Task 2 | Not Started |
| INV-001 | [unchanged behavior] | TC-REG-001 | Task 3 | Not Started |

## Task 1: [Description]
- **Status:** [ ] Not Started
- **Depends on:** None
- **Requirements:** REQ-E-001, REQ-E-002
- **Files:** [MODIFY: src/existing.ts] [CREATE: src/new.ts]
- **Subtasks:**
  1. Write failing tests (TC-E-001, TC-E-002)
  2. Implement [behavior]
  3. Run affected tests — confirm green
  4. Run full test suite — confirm no regressions
- **Acceptance:** [measurable criterion]

## Task 2: [Description]
- **Status:** [ ] Not Started
- **Depends on:** Task 1
- **Requirements:** REQ-E-003
- **Files:** [MODIFY: src/handler.ts]
- **Subtasks:**
  1. Write failing tests (TC-E-003)
  2. Implement [behavior]
  3. Run affected tests — confirm green
  4. Run full test suite — confirm no regressions
- **Acceptance:** [measurable criterion]

## Task 3: [Regression tests and verification]
- **Status:** [ ] Not Started
- **Depends on:** Task 1, Task 2
- **Requirements:** INV-001
- **Subtasks:**
  1. Write regression tests (TC-REG-001)
  2. Run full test suite
  3. Run linter and type checker
  4. Verify all acceptance criteria met
- **Acceptance:** All tests green, no regressions, static analysis clean
```

---

## EARS Notation Reference

| Pattern | Template | Example |
|---------|----------|---------|
| **Ubiquitous** | THE SYSTEM SHALL [behavior] | THE SYSTEM SHALL encrypt all data at rest |
| **Event-Driven** | WHEN [event] THE SYSTEM SHALL [behavior] | WHEN a user submits invalid data THE SYSTEM SHALL display validation errors |
| **State-Driven** | WHILE [state] THE SYSTEM SHALL [behavior] | WHILE in maintenance mode THE SYSTEM SHALL reject write operations |
| **Unwanted** | IF [condition] THEN THE SYSTEM SHALL [behavior] | IF the DB connection fails THEN THE SYSTEM SHALL retry 3 times |
| **Complex** | WHILE [state] WHEN [event] THE SYSTEM SHALL [behavior] | WHILE authenticated WHEN requesting protected resource THE SYSTEM SHALL return it |
| **Optional** | WHERE [feature] THE SYSTEM SHALL [behavior] | WHERE premium tier is enabled THE SYSTEM SHALL allow batch exports |

**Anti-patterns:**
- Vague: "handle errors gracefully" (no measurable criterion)
- Compound: "validate AND save AND notify" (split into 3)
- Implementation-specific: "use Redis to cache" (describe WHAT not HOW)
- Untestable: "be fast" (specify "respond within 200ms")

---

## Requirement Quality Checklist

**Clarity**
- [ ] Unambiguous: only one possible interpretation
- [ ] Atomic: describes exactly one behavior
- [ ] Active voice: "THE SYSTEM SHALL [verb]"

**Completeness**
- [ ] Preconditions stated (WHEN/WHILE)
- [ ] Postconditions stated (expected outcome)
- [ ] Error cases covered (IF [failure])
- [ ] No TBD/TODO placeholders remain

**Testability**
- [ ] Contains specific, measurable criteria
- [ ] Result can be verified externally
- [ ] Test can be written before implementation

**Traceability**
- [ ] Has unique ID (REQ-nnn or REQ-E-nnn)
- [ ] Maps to at least one test case
- [ ] Priority assigned (High/Medium/Low)

**Brownfield-specific**
- [ ] Does not duplicate an existing requirement
- [ ] Unchanged behaviors listed for every affected area
- [ ] ID namespace doesn't collide with prior specs
