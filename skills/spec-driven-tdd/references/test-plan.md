# Test Plan Guide

## Table of Contents
- [Existing Test Discovery (Brownfield)](#existing-test-discovery-brownfield)
- [Characterization Tests (No-Tests Brownfield)](#characterization-tests-no-tests-brownfield)
- [Test Derivation from Requirements](#test-derivation-from-requirements)
- [Acceptance Test Patterns (Given/When/Then)](#acceptance-test-patterns)
- [Property-Based Testing](#property-based-testing)
- [Traceability Matrix Format](#traceability-matrix-format)
- [Test Execution Strategy](#test-execution-strategy)
- [Test Categories and When to Use Each](#test-categories)

---

## Existing Test Discovery (Brownfield)

Before writing any new tests in a brownfield project, discover what already exists.

### Discovery steps

1. **Find test files**: Glob for `*test*`, `*spec*`, `__tests__`, `tests/`
2. **Identify the framework**: Look at imports (jest, pytest, mocha, vitest, etc.)
3. **Read the test runner command**: Check package.json scripts, Makefile, CI config
4. **Read 2-3 existing test files** in the affected area to absorb:
   - Assertion style (expect, assert, should)
   - File naming convention (`.test.ts`, `_test.py`, `.spec.js`)
   - Directory structure (co-located, separate `tests/` folder)
   - Fixture/setup patterns (beforeEach, setUp, factories)
   - Mocking patterns (what's mocked, what's real)
5. **Check for existing coverage** of behaviors you're about to change
6. **Note test infrastructure**: Docker compose for test DBs, seed scripts, CI config

### Pattern matching rules

All new tests MUST match existing patterns:
- Same framework and assertion style
- Same file naming convention
- Same directory structure
- Same setup/teardown patterns
- Same import style

If no existing tests exist, proceed to the
[Characterization Tests](#characterization-tests-no-tests-brownfield) section
to bootstrap test infrastructure and establish a baseline before writing
spec-derived tests.

### Test quality checklist (for each new test)

Before accepting a new test, verify:

1. **Scope**: Tests the public API, not implementation details?
2. **Focus**: Tests one behavior per test case?
3. **Clarity**: Understandable without reading external code?
4. **Resilience**: Won't break during harmless refactoring?
5. **Simplicity**: No loops, conditionals, or math in test body?
6. **Naming**: Name describes action + expected result?

### When to update existing tests

| Change type | Update existing tests? | Rationale |
|-------------|----------------------|-----------|
| Pure refactoring | No | Internal restructuring doesn't affect contracts |
| New features | No | Existing behaviors remain valid |
| Bug fixes | No (add new tests) | Missing tests should be added, not old ones modified |
| Behavior changes | Yes | Only when the spec explicitly changes that behavior |

---

## Characterization Tests (No-Tests Brownfield)

Characterization tests capture the CURRENT behavior of existing code before
you modify it. They are not aspirational — they document what the code actually
does, warts and all. They serve as the regression safety net that pre-existing
tests would normally provide.

### When to write characterization tests

Write characterization tests when:
- The brownfield project has no test infrastructure (Phase 3 bootstrap)
- The affected area of a brownfield project has no test coverage
- You need to refactor code and want proof that behavior is preserved

Do NOT write characterization tests when:
- Adequate tests already exist for the affected area
- The change is greenfield (no existing behavior to characterize)

### The characterization test method

```
For each function/endpoint/behavior in the affected area:
  1. Call the function with representative inputs
  2. Observe what it actually returns / does
  3. Write a test that asserts that exact behavior
  4. Run the test — it MUST pass against unchanged code
  5. If it fails, your test is wrong — fix the test, not the code
```

**Key principle:** The code is the oracle. You are not testing what the code
SHOULD do. You are recording what it DOES do. Even if the behavior looks
wrong, write the test to match reality. You can change the behavior later
in the TDD loop.

### What to characterize

Focus on the **affected area** identified in Phase 0. Prioritize by risk:

| Priority | What to Test | Why |
|----------|-------------|-----|
| **1. Public API surface** | Exported functions, HTTP endpoints, CLI commands | These are what external consumers depend on |
| **2. Functions you'll modify** | Any function body you'll change | Direct regression risk |
| **3. Functions you'll call** | Existing functions your new code will invoke | Need to know their actual interface |
| **4. Integration boundaries** | Where the affected module connects to others | Ripple effects from changes |
| **5. Error paths** | What happens with bad input, missing data, failures | Often forgotten, frequently broken |

### How many characterization tests

Scale to the risk of the change:

| Change Type | Characterization Scope |
|-------------|----------------------|
| **Enhancement** (add feature) | Functions you'll modify or call directly. 5-15 tests typical. |
| **Refactor** (change structure) | ALL externally observable behavior of refactored code. 15-50+ tests. |
| **Bugfix** | Behavior around the bug + adjacent functions. 3-10 tests. The bug reproduction itself is a characterization test. |

**Stop when:** You've covered the public surface of the affected area
and the critical paths through it. You don't need to characterize the
entire codebase — just the blast radius of your change.

### Characterization test naming convention

Use a `CHAR` prefix to distinguish these from spec-derived tests:

```
test_CHAR_[function/endpoint]_[scenario]_[observed_behavior]
```

**Examples by language:**

```python
# Python
def test_CHAR_create_user_valid_data_returns_user_dict():
def test_CHAR_create_user_duplicate_email_raises_value_error():
def test_CHAR_get_users_empty_db_returns_empty_list():
def test_CHAR_delete_user_nonexistent_returns_false():
```

```typescript
// TypeScript
describe('CHAR: UserService', () => {
  it('CHAR_createUser_validData_returnsUserObject', () => { ... })
  it('CHAR_createUser_duplicateEmail_throwsConflictError', () => { ... })
  it('CHAR_getUsers_emptyDb_returnsEmptyArray', () => { ... })
})
```

```go
// Go
func TestCHAR_CreateUser_ValidData_ReturnsUser(t *testing.T) { ... }
func TestCHAR_CreateUser_DuplicateEmail_ReturnsError(t *testing.T) { ... }
```

### Characterization test structure

Each test follows the same pattern:

```python
def test_CHAR_[function]_[scenario]_[behavior]():
    # ARRANGE: Set up the precondition
    input_data = {"name": "Alice", "email": "alice@example.com"}

    # ACT: Call the function as it exists today
    result = create_user(input_data)

    # ASSERT: Document the actual behavior
    assert result["name"] == "Alice"
    assert result["email"] == "alice@example.com"
    assert "id" in result  # it generates an ID
    assert "created_at" in result  # it adds a timestamp
```

**Rules:**
- One behavior per test (same as any unit test)
- Assert observable outputs, not internal state
- Use concrete values, not fuzzy matchers (you're recording exact behavior)
- Include comments if the behavior is surprising: `# NOTE: returns None, not error`

### Handling side effects in characterization tests

Some functions have side effects (database writes, file creation, API calls).
Characterize these by verifying the side effect occurred:

```python
def test_CHAR_create_user_valid_data_writes_to_database():
    create_user({"name": "Alice", "email": "alice@example.com"})
    # Verify the side effect
    user = db.query("SELECT * FROM users WHERE email = 'alice@example.com'")
    assert user is not None
    assert user.name == "Alice"
```

For external API calls, mock the external service but verify the call was made
with the expected arguments:

```python
def test_CHAR_send_notification_calls_email_api():
    with mock.patch('services.email.send') as mock_send:
        notify_user(user_id=123, message="Hello")
        mock_send.assert_called_once_with(
            to="user123@example.com",  # document actual behavior
            body="Hello"
        )
```

### Verifying characterization tests

After writing all characterization tests:

1. **Run them against unchanged code.** Every test MUST pass. If any fail,
   your test doesn't match reality — fix the test.
2. **Intentionally break something** in the affected area (add a `return None`
   at the top of a function). Verify that at least one characterization test
   fails. If nothing fails, your coverage has a gap.
3. **Revert the intentional break.** All tests should pass again.

### Mapping characterization tests to invariants

Each INV-* in the spec should correspond to at least one characterization test:

```
| Invariant | Characterization Test | Status |
|-----------|----------------------|--------|
| INV-001: create_user returns user object | test_CHAR_create_user_valid_data_returns_user_dict | Passing |
| INV-002: duplicate email raises error | test_CHAR_create_user_duplicate_email_raises_value_error | Passing |
| INV-003: get_users returns list | test_CHAR_get_users_empty_db_returns_empty_list | Passing |
```

If an INV-* has no corresponding characterization test, write one now.

### When characterization tests conflict with the spec

If the spec says behavior should CHANGE (Modified Behavior with Was/Now):

1. The characterization test documents the OLD behavior (the "Was")
2. During TDD, the new test documents the NEW behavior (the "Now")
3. After implementing the change, UPDATE the characterization test to match
   the new behavior — or delete it if a spec-derived test now covers the same case
4. Do NOT leave a characterization test that asserts the old, now-incorrect behavior

---

## Test Derivation from Requirements

Each requirement type maps to a specific test approach:

| Requirement Type | Test Approach | Example |
|-----------------|---------------|---------|
| EARS Event-Driven (WHEN...SHALL) | Acceptance test | WHEN user submits form -> test submission |
| EARS Ubiquitous (SHALL) | Property-based test | SHALL encrypt -> test for random payloads |
| EARS Unwanted (IF...THEN SHALL) | Error/edge case test | IF DB fails -> test with DB down |
| Non-Functional (performance) | Benchmark test | Response < 200ms -> load test |
| Invariant (SHALL CONTINUE TO) | Regression test | Login still works after change |
| Modified Behavior (Was/Now) | Before/after test pair | Old behavior removed, new verified |

### Derivation steps

1. Read each REQ-* in requirements.md (or bugfix.md / refactor.md)
2. For each requirement, write the test FIRST (before any code)
3. Name tests descriptively: `test_[feature]_[scenario]_[expected]`
4. Map every test to its requirement ID in the traceability matrix
5. For invariants (INV-*), write regression tests that verify unchanged behavior

### Modified behavior testing (brownfield enhancement)

When a requirement changes existing behavior (has a "Was/Now" section):
1. Write a test for the NEW expected behavior (it should fail initially)
2. Verify existing tests that cover the OLD behavior — they should still pass
   before your change and may need updating after
3. Implement the change
4. Update or replace old tests to match new behavior
5. Confirm regression tests for unrelated behavior still pass

---

## Acceptance Test Patterns

### Standard Given/When/Then

```gherkin
Scenario: [One behavior, descriptively named]
  Given [initial state / precondition]
    And [additional precondition if needed]
  When [single action / event]
  Then [expected observable outcome]
    And [additional assertion if needed]
```

### Rules

- **One scenario = one behavior** (the cardinal rule)
- **Declarative, not imperative** (describe WHAT happens, not HOW)
- **< 10 steps per scenario**
- **No mixing When-Then-When-Then** in one scenario
- Given = past/present state; When = present action; Then = present/future outcome

### Parameterized scenarios

```gherkin
Scenario Outline: Validate input boundaries
  Given the system accepts values between <min> and <max>
  When the user provides <input>
  Then the system should <result>

  Examples:
    | min | max | input | result          |
    | 1   | 100 | 0     | reject (below)  |
    | 1   | 100 | 1     | accept (at min) |
    | 1   | 100 | 100   | accept (at max) |
    | 1   | 100 | 101   | reject (above)  |
    | 1   | 100 | ""    | reject (empty)  |
```

### Error scenario pattern

```gherkin
Scenario: Handle [error condition] gracefully
  Given [normal preconditions]
    And [the error condition is present]
  When [user performs action]
  Then the system should [error handling behavior]
    And the system should [user feedback]
    And the system should NOT [data corruption / side effect]
```

### Regression scenario pattern (brownfield)

```gherkin
Scenario: [Existing behavior] still works after [change]
  Given [same preconditions as before the change]
  When [same action as before the change]
  Then [same outcome as before the change]
```

---

## Property-Based Testing

Properties are invariants that hold for ALL valid inputs.

### When to use

- Sorting: output is sorted AND contains same elements
- Serialization: deserialize(serialize(x)) == x
- Validation: all valid inputs accepted, all invalid rejected
- Idempotency: f(f(x)) == f(x)
- State machines: valid transitions preserve invariants
- **Refactors**: old_function(x) == new_function(x) for all x

### Pattern

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_preserves_elements(input_list):
    result = my_sort(input_list)
    assert sorted(result) == sorted(input_list)
    assert all(result[i] <= result[i+1] for i in range(len(result)-1))
```

### Refactor equivalence testing (brownfield)

When refactoring, property-based testing can verify the refactored code
produces identical results:

```python
@given(st.text(), st.integers())
def test_refactored_matches_original(text_input, num_input):
    assert old_implementation(text_input, num_input) == \
           new_implementation(text_input, num_input)
```

### Derivation from EARS requirements

| EARS Requirement | Property |
|-----------------|----------|
| SHALL return results in ascending order | result[i] <= result[i+1] for all i |
| WHEN saved SHALL preserve all fields | load(save(entity)) == entity |
| SHALL reject inputs > 255 chars | validate(string > 255) returns error |
| SHALL CONTINUE TO [behavior] | old_behavior(x) == new_behavior(x) |

---

## Traceability Matrix Format

### Current iteration scope (for tasks.md)

```markdown
| Req ID | Requirement | Test IDs | Status |
|--------|-------------|----------|--------|
| REQ-E-001 | [new behavior] | TC-E-001, TC-E-002 | Pass |
| REQ-E-002 | [new behavior] | TC-E-003 | Pass |
| INV-001 | [unchanged behavior] | TC-REG-001 | Pass |
| INV-002 | [unchanged behavior] | TC-REG-002 | Pass |
```

### Full verification format

```markdown
| Req ID | Requirement | Test IDs | Status | Code Location | Verified |
|--------|-------------|----------|--------|--------------|----------|
| REQ-E-001 | [text] | TC-E-001 | 1/1 Pass | src/new.ts:L10 | Yes |
| INV-001 | [unchanged] | TC-REG-001 | 1/1 Pass | (unchanged) | Yes |

### Coverage Summary
| Metric | Count |
|--------|-------|
| New requirements | [n] |
| New requirements with passing tests | [n] |
| Invariants | [n] |
| Invariants with passing regression tests | [n] |
| Existing test suite | [n] total, [n] passing |
```

### Brownfield traceability rules

- **Scope to current iteration.** The matrix tracks only this iteration's
  requirements and invariants, not the entire project's history.
- **Existing tests are NOT orphans.** Only flag NEW tests (from this iteration)
  that don't map to a requirement. Pre-existing tests belong to prior work.
- **100% new requirement coverage.** Every REQ-* from this iteration must have
  >= 1 passing test.
- **100% invariant coverage.** Every INV-* must have >= 1 passing regression test.
- **Existing suite must pass.** The full pre-existing test suite must be green,
  confirming no regressions beyond what's tracked in the matrix.

---

## Test Execution Strategy

### Greenfield

Run all tests at every step (test suite is small).

### Brownfield (tests exist)

Use a two-tier approach for speed + safety:

| When | What to run | Why |
|------|-------------|-----|
| During Red-Green iterations | Only new/affected tests | Speed: fast feedback loop |
| After completing each task | Full test suite | Safety: catch regressions |
| At final verification | Full suite + linter + type checker | Completeness |

**Finding the right test subset:**
- Run tests in the same file/directory as the code you changed
- Run tests that import/use the modules you modified
- If unsure which tests are affected, run the full suite

**If the full suite is very slow (>5 min):**
- Use test runner flags to run only affected tests (e.g., `jest --changedSince`,
  `pytest -k "test_affected"`)
- Still run the full suite at task boundaries and final verification

### Brownfield (no tests — after bootstrap)

Characterization tests ARE your test suite. Use the same two-tier approach:

| When | What to run | Why |
|------|-------------|-----|
| During Red-Green iterations | New tests + characterization tests | Speed + regression safety |
| After completing each task | ALL tests (characterization + new) | Full regression check |
| At final verification | All tests + linter + type checker | Completeness |

**If a characterization test fails:** Your new code broke existing behavior.
Same response as a pre-existing test failing: fix your new code first.

**If a characterization test conflicts with the spec:** The spec intends to
change that behavior. Update the characterization test AFTER implementing
the change, not before.

---

## Test Categories

```
What are you testing?
  |
  +-> Single function/method logic
  |     -> Unit test
  |
  +-> Interaction between components
  |     -> Integration test (real dependencies where feasible)
  |
  +-> End-to-end user workflow
  |     -> Acceptance test (Given/When/Then)
  |
  +-> Invariant for ALL inputs
  |     -> Property-based test
  |
  +-> Performance threshold
  |     -> Benchmark/load test
  |
  +-> Existing behavior must not change
  |     -> Regression test
  |
  +-> Refactored code matches original behavior
  |     -> Property-based equivalence test
  |
  +-> Documenting current behavior of untested code before modifying it
        -> Characterization test (CHAR prefix)
```

### Priority order

1. **Characterization tests** (if no tests exist) — Establish baseline BEFORE any changes
2. **Acceptance tests** — Validate requirements end-to-end (highest value)
3. **Unit tests** — Cover logic branches, edge cases, validation
4. **Regression tests** — Protect unchanged behaviors (critical for brownfield)
5. **Property-based tests** — Verify invariants, refactor equivalence
6. **Integration tests** — Validate external system interactions
7. **Performance tests** — Verify non-functional requirements
