# The Workflow Router Pattern

A reusable scaffold for building AI coding agent skills that adapt to project
context instead of forcing a one-size-fits-all flow.

## Problem

A skill that always runs the same phases is brittle. "Add a new feature" and
"refactor an existing module" share *some* steps, but a strict
clarify→specify→test→implement pipeline wastes effort on a refactor (the spec
already exists in code) and skips load-bearing work on a bug fix (root cause
analysis).

Most skills handle this by being narrow: one skill per scenario. That
fragments the user experience and forces the agent to pick between
near-duplicates.

## Pattern

A **Workflow Router** is the first decision a skill makes. It inspects the
request and the repository, classifies the task into a small set of workflow
types, and then runs only the phases that workflow needs.

```
Request + Repo state
        |
        v
+-----------------+
| Workflow Router |  <-- decides workflow type
+-----------------+
        |
        +--> Greenfield  --+
        |                  |
        +--> Brownfield ---+
        |                  |
        +--> Refactor   ---+
        |                  |
        +--> Bugfix     ---+
        |                  |
        +--> Audit      ---+
                           |
                           v
                  Shared verification phase
```

### Properties

1. **One skill, many flows.** The router collapses what would otherwise be
   four or five skills into one, with a single discoverable entry point.

2. **Phases are typed.** Each workflow declares its phases up front, so the
   agent knows what's coming and the user can interrupt at a known seam.

3. **Convergence at verification.** Every workflow ends in the same shared
   verification phase (run tests, check drift, confirm against the spec or
   the docs). This keeps quality consistent regardless of entry point.

4. **Branch-specific guidance.** Brownfield work needs characterization
   tests; greenfield doesn't. Audit needs a checklist; refactor needs an
   invariant list. The router-driven structure lets each branch own its
   distinctive technique without polluting the others.

## The four-or-five canonical workflows

Across the skills in this repo, the same workflow set keeps showing up:

| Workflow    | When it applies                                         |
| ----------- | ------------------------------------------------------- |
| Greenfield  | New code, new repo, blank canvas                        |
| Brownfield  | Existing code, modify or extend                         |
| Refactor    | Behavior preserved, structure changed                   |
| Bugfix      | Behavior wrong, root cause analysis required            |
| Audit       | Read-only assessment, produce a report                  |

Not every skill uses all five. `spec-driven-tdd` uses Greenfield, Brownfield
(with and without existing tests), Refactor, Bugfix. `doc-authoring` and
`github-actions` use Greenfield, Brownfield, Audit. Pick the subset that
matches the domain.

## How to apply it

When you build a skill with this pattern:

1. **List the realistic scenarios** a user will bring to the skill.
   Group them into 3–5 workflow types. Fewer is better; if two flows differ
   only in a single phase, merge them and parameterise that phase.

2. **Write the router as a flowchart at the top of `SKILL.md`.** The agent
   should be able to read it and pick a workflow without ambiguity. Use
   yes/no questions about the repo state, not the user's intent — the user
   often doesn't know which workflow they want.

3. **Declare phases per workflow.** Each workflow should list its phases
   inline (`Phases: Discover → Plan → Implement → Verify`). The phase names
   become the contract between the agent and the user.

4. **Share a verification phase.** All workflows converge on a verification
   phase that runs tests, validates outputs, and checks drift. This is the
   quality floor.

5. **Keep references workflow-agnostic where possible.** Templates, rule
   sheets, and checklists usually apply across workflows. Put them in
   `references/` and link from each workflow's phases.

## What this is not

- **Not a state machine.** The router fires once at the start. After that,
  the agent runs the chosen workflow's phases in order. Don't try to
  re-route mid-flow; if you find yourself wanting to, the router taxonomy
  is wrong.

- **Not a substitute for clarification.** The router classifies the task.
  It does not replace asking the user about ambiguity within the chosen
  workflow.

- **Not for every skill.** Skills that do one specific thing (run a query,
  generate one artifact, summarise one input) don't need a router. The
  pattern earns its complexity only when a skill spans multiple genuinely
  different scenarios.

## Reference implementations

- `skills/spec-driven-tdd/` — five workflows, deepest router.
- `skills/doc-authoring/` — three workflows, Diataxis-typed outputs.
- `skills/github-actions/` — three workflows, security-hardened outputs.

Read the `SKILL.md` of each to see how the same pattern adapts to different
domains.
