---
name: spec-driven-tdd
description: >
  Disciplined spec-driven test-driven development workflow for building software
  with AI coding agents. Transforms ambiguous requests into verified implementations
  through structured specification, test derivation, and strict TDD.
  Handles greenfield projects, brownfield enhancements (with or without existing
  tests), refactors, and complex bug fixes with workflow-specific guidance for each.
  Use when the user requests a new feature, module, enhancement, refactor, API,
  data pipeline, CLI tool, or system with multiple requirements, edge cases, or
  unclear specifications. Also use for complex bug fixes requiring root cause analysis.
  Triggers on phrases like "add a feature", "implement", "build a new module",
  "build an API", "build a CLI", "build a data pipeline", "refactor", "fix this
  bug", "write tests for", "TDD", "test-first", "the requirements are unclear",
  "characterization tests", or "spec this out". Triggers when modifying code
  with adjacent test files (`tests/`, `*_test.py`, `*.test.ts`, `*.spec.ts`,
  `spec/`, `__tests__/`) or test framework config (pytest.ini, jest.config.*,
  go.mod with testing imports, Cargo.toml with [dev-dependencies], package.json
  with a test script). Triggers when the user mentions edge cases, invariants,
  acceptance criteria, EARS notation, red-green-refactor, smoke tests,
  functional tests, mock parity, or boundary inventory.
  Do NOT use for simple one-line fixes, cosmetic changes, formatting, renames,
  dependency bumps, or tasks where requirements are already fully specified
  with tests provided.
---

# Spec-Driven Test-Driven Development

Transform requests into verified implementations through a structured pipeline
that adapts to the project context.

## Workflow Router

Determine the workflow type before starting. This drives which phases apply and
which templates to use.

```
Does a codebase already exist?
  |
  NO --> GREENFIELD (new project from scratch)
  |       Phases: Clarify -> Specify -> Test Plan -> TDD -> Verify
  |
  YES --> What type of change?
           |
           +-> Bug report / error / regression
           |     --> BUGFIX workflow
           |     Phases: Explore -> Bugfix Spec -> TDD -> Verify
           |
           +-> Add new capability not in codebase
           |     --> Does test infrastructure exist?
           |          |
           |          YES --> ENHANCEMENT workflow
           |          |     Phases: Explore -> Clarify -> Specify (delta) -> Test Plan -> TDD -> Verify
           |          |
           |          NO  --> ENHANCEMENT (NO TESTS) workflow
           |                Phases: Explore -> Clarify -> Specify (delta) -> Bootstrap Tests -> Test Plan -> TDD -> Verify
           |
           +-> Change behavior without adding capability
           |     --> Does test infrastructure exist?
           |          |
           |          YES --> REFACTOR workflow
           |          |     Phases: Explore -> Clarify -> Specify (delta) -> Test Plan -> TDD -> Verify
           |          |
           |          NO  --> REFACTOR (NO TESTS) workflow
           |                Phases: Explore -> Clarify -> Specify (delta) -> Bootstrap Tests -> Test Plan -> TDD -> Verify
           |
           +-> Simple, well-understood change (one-sentence diff)
                 --> DIRECT (skip this skill)
```

**Signal detection:**

| Signal in request | Likely type |
|-------------------|-------------|
| Error message, stack trace, "doesn't work", "broken" | Bugfix |
| "Add", "create", "new", "build", "implement" | Enhancement |
| "Improve", "enhance", "extend", "also support" | Enhancement |
| "Clean up", "modernize", "consolidate", "simplify" | Refactor |
| "Slow", "optimize", "performance" | Refactor (optimization) |
| No existing code, "start fresh", "new project" | Greenfield |
| "Add tests", "no tests", "untested" | Enhancement/Refactor (no tests) |

**Test infrastructure detection:**

Existing test infrastructure = ANY of these found:
- Test files (`*test*`, `*spec*`, `__tests__/`)
- Test runner config (jest.config.*, vitest.config.*, pytest.ini, conftest.py, .mocharc.*)
- Test script in manifest (`"test"` script in package.json, test target in Makefile)
- CI config that runs tests

No test infrastructure = NONE of the above found.

## Two Axes of Verification

Verification has **two orthogonal dimensions**, not one. A spec that scores
high on one axis but low on the other ships systems that pass every gate
and still fail on first real traffic.

| Axis | Question | Failure mode if missing |
| --- | --- | --- |
| **Coverage** | How many requirements / behaviors / lines / branches are exercised? | Untested requirements, silent regressions |
| **Realism** | Does any test cross a real boundary — real HTTP, real DB, real LLM, real browser, real process? | Mock/prod divergence, unawaited coroutines, FE↔BE drift, 0% adapters |

Most TDD discipline optimises the coverage axis. This skill **also**
requires the realism axis: a smoke / functional tier, mock-parity
contracts, boundary inventories, phase-end demos, and outcome-vs-side-
effect assertions. They are not optional add-ons; they are how this
skill avoids producing systems that "pass every test and still don't
work."

The phases below enforce both axes. When a phase says *"required"* for
a realism artifact, refusing to produce it means refusing to mark the
phase complete.

## Phase 0: Explore Existing Codebase (Brownfield Only)

**Skip this phase for greenfield projects.**

Before asking the user anything or writing any spec, understand what already
exists. Explore in read-only mode.

### What to discover

1. **Project structure**: Directory layout, where source and tests live
2. **Tech stack**: Framework, language, versions, build system, package manager
3. **Architectural patterns**: MVC, layered, event-driven, microservices, etc.
4. **Naming conventions**: File naming, function naming, variable casing
5. **Existing patterns**: Find the closest analog to the requested change — how
   is similar functionality already implemented?
6. **Data models**: Key entities, relationships, schemas
7. **Test infrastructure**: Test framework, test runner command, fixture patterns,
   approximate coverage, where test files live
8. **Existing spec artifacts**: Check for prior requirements.md, design.md,
   tasks.md, CLAUDE.md, or similar documentation
9. **Affected area**: Which files and modules will this change touch?

### How to explore

- Read the project's CLAUDE.md or README first (if they exist)
- Use glob/grep to find relevant source files and test files
- Read 2-3 existing files in the affected area to absorb patterns
- If prior spec files exist, read them to understand accumulated requirements

### Test infrastructure assessment

Determine the test state by scanning for:

| Indicator | What it tells you |
|-----------|------------------|
| Test files exist in affected area | Tests cover the code you'll change |
| Test files exist elsewhere but not here | Project has tests, but not for this area |
| Test framework in dependencies | Framework chosen but maybe unused |
| Test runner config exists | Infrastructure is set up |
| `"test"` script in manifest | Runner command is established |
| CI config runs tests | Tests are part of the workflow |
| No test files, no config, no scripts | No test infrastructure at all |

Classify the result:

| Test State | Definition | Workflow Impact |
|-----------|-----------|----------------|
| **Tests exist** | Test files + runner + config all present | Standard brownfield |
| **Partial infra** | Framework in deps or config exists, but no/few test files | Bootstrap: write tests, skip framework selection |
| **No tests** | No test files, no config, no runner | Bootstrap: full test infrastructure setup |

### Output: Codebase Context Summary

Produce a brief mental model (do not write a file unless the user requests it):

```
Stack: [framework, language, test runner]
Patterns: [architecture, naming, file organization]
Affected area: [files/modules this change touches]
Existing tests: [relevant test files, approximate count, runner command]
  OR: No test infrastructure found
Test state: [tests exist | partial infra | no tests]
Prior specs: [any existing requirements.md/design.md or none]
Closest analog: [existing feature most similar to requested change]
```

This context informs every subsequent phase.

## Phase 1: Clarify Requirements

Evaluate the request's clarity before starting any work.

### Greenfield

Ask about the blank slate: inputs/outputs, data formats, error handling,
edge cases, performance constraints, tech stack preferences.

### Brownfield (tests exist)

**Use codebase context from Phase 0.** Do NOT ask questions the codebase
already answers. Focus questions on:
- What should the NEW behavior be? (the codebase shows current behavior)
- Where does new behavior differ from existing patterns?
- What existing behavior must NOT change?
- Are there constraints the existing architecture imposes?

### Brownfield (no tests)

In addition to the standard brownfield questions, clarify:
- **Test framework preference**: Does the user have a preferred test framework?
  If not, recommend one based on the stack (see Bootstrap Tests phase).
- **Test scope**: Should tests cover ONLY the new/changed code, or also
  establish baseline coverage for existing code in the affected area?
- **Test location**: Co-located with source (`src/foo.test.ts`) or separate
  directory (`tests/`)? Infer from project conventions if possible.

**Ambiguity signals (if ANY present, ask before proceeding):**
- Multiple valid interpretations producing different implementations
- Missing I/O specifications, pre/post conditions, or data formats
- Unspecified error handling, edge cases, or boundary conditions
- Unclear performance, security, or environmental constraints
- Change touches shared/critical code paths

**When to proceed without asking:**
- Request is well-constrained with one obvious implementation
- Ambiguity is purely cosmetic (naming, formatting)
- Clarification loop has reached 3 rounds (proceed with stated assumptions)

**Interview approach:**
- Ask up to 3 focused questions per round
- Each question should target a decision that changes the implementation
- State the default assumption alongside each question
- After each round, summarize resolved items and remaining unknowns

## Phase 2: Write Specification

Use the templates in [references/spec-template.md](references/spec-template.md).
Which template depends on the workflow type:

| Workflow | Spec Template | Key Difference |
|----------|--------------|----------------|
| Greenfield | Feature (requirements.md + design.md) | Written from scratch |
| Enhancement | Enhancement (requirements.md + design-delta.md) | Extends existing specs, documents only what changes |
| Enhancement (no tests) | Enhancement (requirements.md + design-delta.md) | Same as above, plus test infrastructure decisions |
| Refactor | Refactor (refactor.md) | Documents behavior preservation + structural changes |
| Refactor (no tests) | Refactor (refactor.md) | Same as above, plus test infrastructure decisions |
| Bugfix | Bugfix (bugfix.md) | Current/Expected/Unchanged behavior |

### EARS notation (all workflows)

| Pattern | Syntax | Use Case |
|---------|--------|----------|
| Ubiquitous | THE SYSTEM SHALL [behavior] | Always-on requirements |
| Event-Driven | WHEN [event] THE SYSTEM SHALL [behavior] | Triggered by events |
| State-Driven | WHILE [state] THE SYSTEM SHALL [behavior] | During a state |
| Unwanted | IF [condition] THEN THE SYSTEM SHALL [behavior] | Exception handling |
| Complex | WHILE [state] WHEN [event] THE SYSTEM SHALL [behavior] | Combined |

Every requirement must be **atomic**, **testable**, and have a **unique ID**.

### Requirement ID namespacing (brownfield)

If prior specs exist with REQ-001 through REQ-N, continue numbering from REQ-(N+1).
If no prior specs exist, start fresh. Use prefixes to distinguish:
- `REQ-E-nnn` for enhancement requirements
- `REQ-R-nnn` for refactor requirements
- `REQ-BUG-nnn` for bugfix requirements

### Unchanged Behavior section (critical for brownfield)

Every brownfield spec MUST include an Unchanged Behavior section:
```
- INV-001: WHEN [condition] THE SYSTEM SHALL CONTINUE TO [existing behavior]
```
This is the primary regression prevention mechanism.

**When no tests exist, invariants are your ONLY regression protection.** Be
more thorough than usual: list every behavior in the affected area that must
survive the change. Read the source code and its callers to identify these.

### Greenfield: requirements.md + design.md

Write both from scratch per the Feature template. Full architecture, data models,
API contracts, error handling, testing strategy.

### Enhancement: requirements.md + design-delta.md

**requirements.md**: Add only NEW requirements. Reference existing spec if present.
Include Unchanged Behavior invariants for every existing behavior the change
could affect.

**design-delta.md**: Document only what changes:
- Modified components (what exists → what changes)
- New components (what's added)
- Integration points (how new connects to existing)
- Files to modify vs. files to create

Do NOT rewrite the entire architecture. Reference existing design.

**No-tests addition to design-delta.md**: Include a Test Infrastructure section:
```markdown
## Test Infrastructure (New)
- **Framework:** [chosen framework and rationale]
- **Runner command:** [command to run tests]
- **Config file:** [path to test config]
- **Test location:** [co-located | separate directory]
- **Naming convention:** [pattern, e.g., *.test.ts, test_*.py]
```

### Premortem section (required in requirements.md, all workflows except Bugfix)

Before finalising requirements, list **the top 5 ways this will fail in
production** and an EARS-form mitigation for each. This is required —
not advisory. It forces the spec author to think one level past *"what
is the requirement"* into *"what will go wrong at the seams."*

```
## Premortem

| # | Failure mode | Mitigation (EARS) |
| - | --- | --- |
| 1 | Mongomock returns sync values; production AsyncMongoClient returns coroutines. Tests pass; routes fail. | THE SYSTEM SHALL include a mock-parity contract test asserting the fake returns awaitable values for every async DB method used. |
| 2 | LLM provider returns a 503 mid-stream; partial output is persisted as final. | WHEN an LLM stream errors mid-response THE SYSTEM SHALL discard the partial buffer and surface a retry-able error. |
| ... | ... | ... |
```

Each mitigation must produce at least one acceptance test. Every entry
in the premortem should have a corresponding requirement (existing or
new) that drives a test.

### Boundary Inventory (required in design.md, all multi-component workflows)

A "boundary" is any place where the runtime model changes:

- sync → async
- mock → real (test fake → production dependency)
- frontend → backend
- in-process → out-of-process
- JSON → Python / TypeScript / Go object (de/serialisation)
- single thread → multi-process / multi-machine
- HTTP → SSE / WebSocket / streaming

Every boundary in the system must appear in `design.md` under a
**Boundary Inventory** section, with at least one acceptance test
referenced by ID.

```
## Boundary Inventory

| # | Boundary | From | To | Acceptance test |
| - | --- | --- | --- | --- |
| 1 | HTTP entry | external client | FastAPI route | TC-SMOKE-001 (curl over uvicorn) |
| 2 | Sync→Async DB | route handler | AsyncMongoClient | TC-PARITY-001 (mongomock vs Atlas) |
| 3 | FE→BE | React fetch | /api/sessions | TC-SMOKE-002 (Playwright + live BE) |
| 4 | Persisted state | Repository | MongoDB Atlas | TC-INTEG-LIVE-001 (testcontainers) |
| 5 | LLM stream | Backend SSE | Browser EventSource | TC-SMOKE-003 (websocat or playwright-cli) |
| ... | ... | ... | ... | ... |
```

If a boundary has **no** acceptance test, the design is not done. This
is the structural fix for the entire class of "passed every test, broke
on first real request" failures.

For single-component pure-function libraries with no I/O, document this
explicitly: *"No service-to-service boundaries; all behaviour is
in-process pure functions."* Then this section is satisfied.

See [references/test-tiers.md](references/test-tiers.md) for which test
tiers cover which boundaries.

### Bugfix: bugfix.md

Three-section format:
1. **Current Behavior** (defect): WHEN [x] THEN the system [incorrect behavior]
2. **Expected Behavior** (correct): WHEN [x] THEN the system SHALL [correct behavior]
3. **Unchanged Behavior** (regression): WHEN [y] THE SYSTEM SHALL CONTINUE TO [existing behavior]

Derive three tests: reproduce bug, validate fix, confirm no regressions.

## Phase 3: Bootstrap Test Infrastructure (No-Tests Workflows Only)

**Skip this phase if test infrastructure already exists.**

Before deriving a test plan, establish the test infrastructure. You cannot
write tests without a framework, runner, and conventions.

### Step 1: Select test framework

If the user specified a preference in Phase 1, use it. Otherwise, select
based on the stack:

| Stack | Recommended Framework | Rationale |
|-------|----------------------|-----------|
| TypeScript / ESM | vitest | Native ESM, fast, compatible with Jest API |
| TypeScript / CJS | jest + ts-jest | Mature ecosystem, wide adoption |
| JavaScript / Node | jest or vitest | Either works; match project's module system |
| JavaScript / Browser | vitest or jest + jsdom | DOM testing support |
| Python 3 | pytest | De facto standard, fixtures, parametrize |
| Python (Django) | pytest-django | Django integration with pytest |
| Go | testing (stdlib) | Built-in, no external dependency needed |
| Rust | cargo test (built-in) | Built-in test framework |
| Java / Spring | JUnit 5 + Mockito | Industry standard |
| Ruby / Rails | RSpec or Minitest | RSpec for BDD style; Minitest for minimal |
| C# / .NET | xUnit or NUnit | xUnit for modern .NET |
| Elixir | ExUnit (built-in) | Built-in test framework |

**Principle: Choose the most conventional option.** The goal is not the "best"
framework but the one the team (or a future developer) will expect.

### Step 2: Install and configure

1. **Add the framework as a dev dependency:**
   ```
   npm install --save-dev vitest          # Node/TS
   pip install pytest                     # Python
   # Go and Rust have built-in testing
   ```

2. **Create test config** (if the framework needs one):
   - `vitest.config.ts`, `jest.config.ts`, `pytest.ini`, etc.
   - Match the project's existing config patterns (TypeScript for TS projects,
     YAML if project uses YAML configs, etc.)

3. **Add test runner script** to the project manifest:
   ```json
   // package.json
   { "scripts": { "test": "vitest run" } }
   ```
   ```toml
   # pyproject.toml
   [tool.pytest.ini_options]
   testpaths = ["tests"]
   ```

4. **Create test directory** (if using separate directory convention):
   ```
   mkdir tests/          # or __tests__/ or test/
   ```

5. **Verify the framework works** by writing a trivial smoke test:
   ```typescript
   // tests/smoke.test.ts
   import { describe, it, expect } from 'vitest'

   describe('test infrastructure', () => {
     it('works', () => {
       expect(1 + 1).toBe(2)
     })
   })
   ```
   Run it. If it passes, infrastructure is ready. Delete or keep the smoke
   test as appropriate.

6. **(Strongly recommended) Add a thinnest-possible live integration
   test against the riskiest external boundary.** For a project that
   touches a database, write a hello-world round-trip against a real
   ephemeral instance (testcontainers, docker compose). For a project
   that touches an LLM, record a single-call cassette (VCR / pytest-
   recording) against the real provider. Cost: a few seconds per CI
   run. Benefit: every subsequent phase is built on a layer that has
   been proven to work against real infrastructure, not just a fake.

   This is *recommended*, not mandated, because some teams cannot
   easily provision a real test instance at bootstrap. If you defer it,
   put a `TODO(live-integration)` marker in `design.md` so the
   omission is visible.

### Step 3: Establish test conventions

Derive conventions from the project's CODE conventions:

| Code Convention | Test Convention |
|----------------|-----------------|
| Files in `src/module/foo.ts` | Tests in `src/module/foo.test.ts` (co-located) or `tests/module/foo.test.ts` (separate) |
| Functions use camelCase | Test names use camelCase: `it('createsUserWithEmail')` |
| Functions use snake_case | Test names use snake_case: `test_creates_user_with_email` |
| Modules organized by feature | Test directories mirror source directories |
| ES modules (import/export) | Tests use same import style |
| CommonJS (require) | Tests use same require style |

**Document the conventions** in design-delta.md (or design.md for greenfield)
so future tests stay consistent.

### Step 4: Write characterization tests (critical)

**Before changing any existing code**, write characterization tests that capture
the current behavior of the code you're about to modify. These are not tests
you want to pass — they are tests that document what the code ACTUALLY does,
so you can detect when your changes break something.

See [references/test-plan.md](references/test-plan.md) for the full
characterization test method.

#### What to characterize

Focus on the **affected area** identified in Phase 0:

1. **Public functions/methods** you'll call or modify: test their current
   inputs → outputs
2. **API endpoints** you'll change: test their current request → response
3. **Critical code paths** through the module: test the happy path and
   major error paths
4. **Integration points** between the module and its callers: test that
   callers get what they expect

#### How many characterization tests

- **Enhancement**: Characterize the specific functions/endpoints you'll modify
  or call. Not the entire codebase — just the affected surface.
- **Refactor**: More extensive. Characterize ALL externally observable behavior
  of the code being refactored. This is your safety net.
- **Bugfix**: Characterize the behavior around the bug. The reproduction test
  IS a characterization test (it documents the current broken behavior).

#### Characterization test naming

Use a distinct naming pattern so these are recognizable:

```
test_CHAR_[function]_[scenario]_[current_behavior]
```

Examples:
```python
def test_CHAR_create_user_with_valid_data_returns_user_object():
def test_CHAR_create_user_with_duplicate_email_raises_conflict():
def test_CHAR_get_user_nonexistent_returns_none():
```

#### Run characterization tests

All characterization tests MUST pass against the UNCHANGED code. If a
characterization test fails, your test is wrong — fix the test, not the code.

**These characterization tests become your regression suite.** They replace
the "existing tests" that the standard brownfield workflow relies on.

### Deliverable: Test infrastructure ready

After this phase:
- Framework installed and configured
- Runner command works
- Test conventions documented
- Characterization tests pass against unchanged code
- You can now proceed to Phase 4 (Test Plan) with confidence

## Phase 4: Derive Test Plan

From the spec, derive tests and write `tasks.md`. See
[references/test-plan.md](references/test-plan.md) for templates.

### Brownfield: Discover existing tests first

Before writing any new tests:
1. **Find existing test files** in the affected area (glob for `*test*`, `*spec*`)
2. **Read relevant existing tests** to understand patterns (framework, assertions,
   fixtures, naming conventions)
3. **Check for existing coverage** of the behaviors you're about to change
4. **Match existing patterns** in all new tests (same framework, same style,
   same file location conventions)

### Brownfield (no tests): Use characterization tests as baseline

If you bootstrapped tests in Phase 3:
1. **Your characterization tests ARE the existing tests.** They serve the same
   role as pre-existing tests in the standard brownfield workflow.
2. **Match the conventions you established** in Phase 3 for all new tests.
3. **Do not add more characterization tests at this point** — Phase 3 covered
   the affected area. Focus on test derivation from the spec.
4. **Map invariants to characterization tests**: Each INV-* should correspond
   to at least one characterization test that already passes.

### Test derivation rules

1. **Each EARS requirement → at least one acceptance test** (Given/When/Then)
2. **Each data model/interface → unit tests** for validation, transformation, edge cases
3. **Each integration point → integration test** using real services where possible
4. **Each invariant → regression test** confirming unchanged behavior
   - If tests already exist: existing tests cover this
   - If no tests existed: characterization tests from Phase 3 cover this
5. **Identify properties** (invariants for all inputs) → property-based tests
6. **Each top-level user story → at least one smoke / functional test**
   that starts the deployable artifact and asserts user-visible outcomes.
   See [references/test-tiers.md](references/test-tiers.md).
7. **Each boundary in the boundary inventory → at least one acceptance
   test that crosses it for real** (real HTTP, real DB, real browser,
   real process). Boundaries listed in `design.md`'s Boundary Inventory
   without a test ID are blockers.
8. **Each fake / mock → at least one mock-parity contract test** that
   asserts the fake matches the real dependency for the methods used.
   See [references/mock-parity.md](references/mock-parity.md).

### Outcome-vs-side-effect rule (required)

Every requirement's primary acceptance test must assert the outcome
**stated in the requirement**, not a proxy side effect.

- ❌ "POST /sessions produces a research report" → asserts `db.sessions.find_one() is not None` (side effect along the way)
- ✅ "POST /sessions produces a research report" → asserts `len(report.content) > 0` and the report contains the topic the user requested (the stated outcome)

Side-effect assertions are valuable as supplementary checks. They are
not sufficient as the primary acceptance for a behavioural requirement.
If the requirement says *"the system produces X"*, at least one test
must observe X.

### Mock realism contract (required for every fake)

Whenever the test plan introduces a fake that stands in for an external
dependency (e.g., `mongomock` for `AsyncMongoClient`, `FakeLLM` for
Gemini, `moto` for AWS), it must also introduce a mock-parity contract
test:

- Run the same call against the fake and against the real dependency
  (testcontainers, recorded cassette, sandbox account).
- Assert observable behaviour matches: return shape, async-ness, error
  taxonomy.

This is the structural fix for the unawaited-coroutine class of bugs
(fake returns a value, real returns a coroutine, code never awaits, all
unit tests pass). One contract test per fake is enough.

See [references/mock-parity.md](references/mock-parity.md) for patterns.

### Test isolation contract (required for state-mutating tests)

Tests that mutate persistent state — database rows, files, env vars,
service registries, prompt registries, feature flags — must declare
what they mutate and own teardown.

- Declare mutations in test metadata (e.g., a marker, a fixture name, a
  module-level docstring): `@pytest.mark.mutates_state(["prompts.compose_report"])`,
  or the equivalent in your stack.
- Provide a teardown / rollback fixture that restores the prior state.
- Never use a production stage / production registry / production key
  as the target of a state-mutating test, even if it is convenient.

This prevents the failure mode where one test pollutes shared state and
the next test reads the polluted state as ground truth.

### Two-axis coverage (required for non-trivial systems)

Replace single-aggregate coverage thresholds with **two floors**:

- **Aggregate floor**: ≥ X% line + branch coverage on the declared
  source directories.
- **Per-module floor**: no individual module under Y% in the same
  directories.

```
- REQ-NF-COV: THE SYSTEM SHALL maintain at minimum 80% line + branch
  coverage on `src/`, with no individual module below 60%.
```

The per-module floor catches 0% adapters that hide behind well-tested
neighbours. An aggregate-only threshold is insufficient: a 0% module
can sit inside a directory that averages 85%.

For pure-library projects with no production-only adapters, a single
aggregate threshold is acceptable.

### Traceability

Maintain a mapping in tasks.md:

```
| Req ID    | Test Case IDs     | Status      |
|-----------|-------------------|-------------|
| REQ-E-001 | TC-E-001, TC-E-002| Not Started |
| INV-001   | TC-REG-001        | Not Started |
```

**Brownfield traceability rules:**
- Every NEW requirement must have >= 1 test
- Every INVARIANT must have >= 1 regression test
- Existing tests from prior iterations are NOT orphans — only flag tests from
  the current iteration that don't map to a requirement
- The traceability matrix covers only the current iteration's scope

**No-tests traceability addition:**
- Characterization tests (CHAR-*) map to INV-* invariants
- Include them in the matrix with their CHAR prefix:

```
| Req ID    | Test Case IDs        | Status      |
|-----------|----------------------|-------------|
| REQ-E-001 | TC-E-001, TC-E-002   | Not Started |
| INV-001   | CHAR-create-user-001 | Passing (baseline) |
| INV-002   | CHAR-get-user-001    | Passing (baseline) |
```

### tasks.md

Break implementation into discrete, sequenced tasks. Each task:
- Maps to one or more requirements
- Has clear acceptance criteria
- Follows dependency order
- Includes "Write tests" as the FIRST subtask (TDD)
- **Brownfield**: Specifies which files are MODIFIED vs. CREATED

**No-tests task ordering:**
For brownfield-no-tests workflows, tasks.md should include the bootstrap
work as Task 0:

```
## Task 0: Bootstrap test infrastructure
- Status: [ ] Not Started
- Requirements: (infrastructure — no REQ mapping)
- Subtasks:
  1. Install [framework], create config
  2. Add test runner script
  3. Write characterization tests for affected area
  4. Verify all characterization tests pass
- Acceptance: `[test command]` runs and all characterization tests pass
```

Principle: **no big jumps in complexity**.

## Phase 5: TDD Implementation Loop

Execute tasks from tasks.md using strict TDD. This is NON-NEGOTIABLE.

### For each task:

```
1. RED      - Write failing test(s) for the task's requirements
2. RUN      - Execute test, confirm it FAILS (if it passes, test is wrong)
3. GREEN    - Write MINIMAL code to make the test pass
4. RUN      - Execute ALL tests (new + existing), confirm ALL pass
5. REFACTOR - Clean up, ensure no test breakage
6. DEMO     - Start the deployable artifact and exercise the new behavior
7. COMMIT   - Mark task complete in tasks.md
```

### Phase-end Demo gate (required)

A task / phase is **not complete** when the tests are green. It is
complete when the deployable artifact has been started and the new
behaviour observed end-to-end. This catches integration-shaped bugs
that no test pyramid can prevent.

For each task, capture the demo evidence in `tasks.md` under the task:

```
- **Demo:**
  - HTTP backend: paste curl/httpie output showing the new endpoint
    returning the expected status and body.
  - Frontend: paste a playwright-cli snapshot or screenshot showing
    the new UI, OR a short transcript of clicks + observed result.
  - CLI tool: paste the command and its output.
  - Library / pure code: explicitly state "no demo applicable; tests
    cover the full surface."
  - Background job: paste log lines or queue state showing the job ran.
```

If you cannot produce a demo because the deployable artifact does not
yet start at this phase, **the boundary inventory and bootstrap phase
were skipped or insufficient**. Stop and address that before continuing.

See [references/demo-tools.md](references/demo-tools.md) for tools by
stack (curl/httpie, playwright-cli, websocat, k6, etc.).

### Running tests (brownfield, tests exist)

- **Inner loop**: Run only the new/affected tests during Red-Green iterations
  (for speed)
- **Task boundary**: Run the FULL test suite after completing each task
  (for regression safety)
- **Final verification**: Run full suite + linters + type checks at the end

### Running tests (brownfield, no tests — after bootstrap)

- **Inner loop**: Run new tests + characterization tests during Red-Green
  iterations. The characterization tests are your regression guardrail.
- **Task boundary**: Run ALL tests (characterization + new) after each task.
- **Final verification**: Run all tests + linters + type checks at the end.

**Critical: If a characterization test fails during implementation, you broke
existing behavior.** This is the same signal as "existing test breaks" in the
standard brownfield workflow. Fix your new code first.

### Rules

- **Never write implementation before its test.**
- **Never alter the spec to satisfy a test.** Spec-derived tests are authoritative.
- **Minimal code only.** Add nothing beyond what makes the current test pass.
- **All tests green before moving to next task.**
- **Use real dependencies where feasible.** Mocks only for external services
  outside your control.
- **Decompose classes by method dependency.** Generate in dependency order,
  test each method individually.
- **Bounded repair.** 3 fix attempts max, then reassess or ask user.

### Brownfield-specific rules (all brownfield workflows)

- **Match existing patterns.** New code must follow the conventions discovered
  in Phase 0 (naming, file structure, import style, error handling).
- **Refactor only what you wrote.** Do NOT refactor existing code unless the
  task explicitly requires it. Existing code is assumed correct until proven otherwise.
- **Read before calling.** Before calling existing functions, read their actual
  signatures. Do not assume existing interfaces — verify them.
- **If an existing test breaks**, your new code caused a regression. Fix your
  new code first (existing passing tests are authoritative). Only modify an
  existing test if the spec explicitly changes that behavior.
- **If a characterization test breaks** (no-tests workflow), the same rule
  applies: your new code caused a regression. Fix your new code. The
  characterization test documents real behavior that something depends on.

### Hallucination prevention

1. Verify external APIs/libraries exist and check current interfaces
2. Chain-of-thought: reason step-by-step before coding
3. Run static analysis after generation
4. Use execution traceback (not just re-reading) to fix failures
5. **Brownfield**: Read existing code before calling it; verify signatures

## Phase 6: Verification

After all tasks complete, verify the full delivery.

### Coverage-axis checklist

- [ ] **Spec compliance:** Every new requirement has at least one passing test
- [ ] **All tests pass:** Full test suite green (new AND existing)
- [ ] **Traceability complete:** Updated matrix with final status
- [ ] **No untested new requirements:** Every REQ-* from this iteration is covered
- [ ] **Unchanged behaviors verified:** All INV-* regression tests pass
- [ ] **Static analysis clean:** No linter errors, type errors, security warnings
- [ ] **Pattern compliance (brownfield):** New code follows existing conventions
- [ ] **Aggregate coverage floor met** on declared source directories
- [ ] **Per-module coverage floor met** — no module silently sits at 0% or below the per-module threshold

### Realism-axis checklist

- [ ] **Smoke / functional tier exists** — at least one test per top-level user story starts the deployable artifact and asserts user-visible outcomes
- [ ] **Boundary inventory complete** — every entry in `design.md`'s Boundary Inventory has a passing acceptance test that crosses the boundary for real
- [ ] **Mock-parity tests pass** — every fake has at least one contract test asserting parity with the real dependency for the methods used
- [ ] **Outcome assertions present** — each behavioural REQ has at least one test that asserts the outcome stated in the REQ, not a proxy side effect
- [ ] **Phase-end demos recorded** — every task in `tasks.md` carries a Demo block with curl / playwright-cli / screenshot / log evidence, OR an explicit "no demo applicable" note for pure code
- [ ] **State-mutating tests declare and clean up** — no test leaves persistent state behind for the next test or live run
- [ ] **Premortem mitigations are tested** — every premortem failure mode in `requirements.md` has a corresponding test

### Additional no-tests verification

- [ ] **Test infrastructure works:** `[test command]` runs cleanly from project root
- [ ] **Characterization tests still pass:** All CHAR-* tests green, confirming
  no behavioral regressions in the affected area
- [ ] **Test conventions documented:** Future developers can find and follow the
  test patterns (in design-delta.md or equivalent)
- [ ] **Runner script exists:** Test command is in manifest (package.json scripts,
  Makefile, etc.) — not just a manual invocation

### Deliverables

**Greenfield:**
1. `requirements.md` — Full specification
2. `design.md` — Full technical design
3. `tasks.md` — Task list with traceability matrix
4. Test suite — All passing
5. Implementation code

**Brownfield enhancement/refactor (tests exist):**
1. `requirements.md` — New/changed requirements only (or appended to existing)
2. `design-delta.md` — What changed in the design
3. `tasks.md` — Task list with traceability matrix for this iteration
4. New/modified tests
5. Implementation changes
6. **Change summary:** Files modified, files created, behaviors added/changed

**Brownfield enhancement/refactor (no tests):**
1. `requirements.md` — New/changed requirements only (or appended to existing)
2. `design-delta.md` — What changed in the design, INCLUDING test infrastructure
   decisions (framework, conventions, directory structure)
3. `tasks.md` — Task list with traceability matrix (includes CHAR-* mappings)
4. Test infrastructure: config, runner script, directory structure
5. Characterization tests for affected area
6. New tests derived from spec
7. Implementation changes
8. **Change summary:** Files modified, files created, behaviors added/changed,
   test infrastructure established

**Bugfix:**
1. `bugfix.md` — Bug analysis with Current/Expected/Unchanged
2. Tests: reproduction, fix validation, regression
3. Fix implementation
4. **Change summary**

## Failure Recovery

```
Test fails
  |
  +-> Is it a NEW test that fails?
  |     +-> Code bug: fix implementation
  |     +-> Test wrong: does it match spec?
  |          +-> Yes: fix code (spec is authoritative)
  |          +-> No: fix test (or revisit spec with user)
  |
  +-> Is it an EXISTING test that fails? (standard brownfield)
  |     +-> Your new code caused a regression
  |     +-> Fix your new code (existing tests are authoritative)
  |     +-> Do NOT modify the existing test unless the spec
  |         explicitly changes that behavior
  |     +-> If the existing test seems wrong, confirm with user
  |         before changing it
  |
  +-> Is it a CHARACTERIZATION test that fails? (no-tests brownfield)
  |     +-> Your new code caused a behavioral regression
  |     +-> Fix your new code (characterization tests document real behavior)
  |     +-> Do NOT modify the characterization test unless the spec
  |         explicitly changes that behavior (listed in Modified Behavior
  |         with Was/Now)
  |     +-> If the characterization test documents behavior the spec
  |         INTENDS to change, update the test to match the new spec
  |
  +-> Test infrastructure won't set up? (no-tests bootstrap)
        +-> Check framework compatibility with project's Node/Python/etc. version
        +-> Check for conflicting config (e.g., module type mismatches)
        +-> Try the next framework in the recommendation table
        +-> If stuck after 2 frameworks, ask user for guidance
```

**Never silently change the spec.** Confirm with user first.
**If stuck after 3 attempts**, ask user for guidance.

## What this skill does not catch

Even with both axes of verification in place, the following classes of
defect remain the responsibility of human review and downstream
processes:

- **Non-deterministic LLM regressions.** The smoke tier exercises a
  real call; it does not guarantee the LLM will produce equally good
  output tomorrow. Use eval suites with representative prompts and a
  judge model for ongoing quality.
- **Capacity under realistic load.** Smoke tests run one user at a
  time. Use a load tier (k6, locust, wrk) for concurrency, throughput,
  and tail latency requirements.
- **End-to-end UX flow review.** Tests assert outcomes; they do not
  judge whether the flow is the right flow. A human walkthrough at
  phase boundaries is still required for any user-facing system.
- **Security review of new attack surface.** Run a security-review
  pass (or invoke `/security-review`) on any change that adds an HTTP
  surface, parses untrusted input, or touches authentication.
- **Production observability and operability.** The skill does not
  prescribe metrics, logs, traces, alerts, runbooks, or on-call
  readiness. Add these as explicit requirements when shipping a system
  others will operate.

This list exists so you don't mistake "all gates green" for "ready to
operate in production."

## Reference Files

- **Spec templates**: See [references/spec-template.md](references/spec-template.md)
  for all templates: Feature, Enhancement, Refactor, Bugfix, Codebase Context,
  Premortem, and Boundary Inventory
- **Test plan guide**: See [references/test-plan.md](references/test-plan.md)
  for test derivation, the five test tiers, outcome-vs-side-effect rule,
  existing test discovery, characterization tests, traceability, and
  Given/When/Then
- **Test tiers catalog**: See [references/test-tiers.md](references/test-tiers.md)
  for definitions and per-stack examples of unit / integration / smoke /
  quality_gate / characterization tiers
- **Mock parity patterns**: See [references/mock-parity.md](references/mock-parity.md)
  for how to write contract tests that prove a fake matches the real
  dependency
- **Demo tools by stack**: See [references/demo-tools.md](references/demo-tools.md)
  for phase-end demo tooling — curl, httpie, playwright-cli, websocat,
  k6, and equivalents
- **Edge case catalog**: See [references/edge-cases.md](references/edge-cases.md)
  for edge case categories to check during specification
