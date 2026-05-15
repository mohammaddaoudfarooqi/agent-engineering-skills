# agent-engineering-skills

Engineering discipline for AI coding agents.
Spec-driven development, CI/CD, and documentation as composable skills.

[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![skills.sh](https://skills.sh/b/mohammaddaoudfarooqi/agent-engineering-skills)](https://skills.sh/mohammaddaoudfarooqi/agent-engineering-skills)

---

## What this is

A small, opinionated set of agent skills that cover the core engineering
loop: **build → ship → document**. Each skill is built on the same
[Workflow Router Pattern](docs/workflow-router-pattern.md) — a router that
classifies your request, then runs only the phases that workflow needs.

| Skill | What it does | When to use |
| --- | --- | --- |
| [`spec-driven-tdd`](skills/spec-driven-tdd/) | Turns ambiguous requests into specs, derives tests from them, and runs strict red-green-refactor TDD. Handles greenfield, brownfield (with or without existing tests), refactors, and complex bug fixes. | Building a new module, adding a feature with unclear requirements, refactoring legacy code without tests, or fixing a bug where the root cause is non-obvious. |
| [`github-actions`](skills/github-actions/) | Generates production-grade GitHub Actions workflows from repo analysis. Security-hardened (action pinning by SHA, OIDC, least-privilege), with caching and optimisation. Handles greenfield, brownfield, and audits. | Bootstrapping CI for a new project, hardening an existing pipeline against supply-chain risks, auditing workflows for security and efficiency, or migrating from another CI system. |
| [`doc-authoring`](skills/doc-authoring/) | Generates accurate, maintainable documentation classified by [Diataxis](https://diataxis.fr) type (tutorial, how-to, reference, explanation). Handles greenfield, brownfield, audits, and identifying what documentation is missing. | Generating a README for a fresh repo, refreshing stale documentation, restructuring scattered docs into a Diataxis layout, or auditing what documentation a project is missing. |

These three skills compose. Spec the feature, ship it through CI, document
it — and the agent keeps your tests, workflows, and docs in sync as the
codebase evolves.

---

## Quickstart

After installing (see below), invoke a skill from your agent. The session
below is an *illustrative example* of the workflow shape, not literal
agent output — actual phrasing, question count, and timings vary by agent
and project.

```
> use spec-driven-tdd to add a rate-limiter to the API

[Workflow Router]
  Existing codebase + has tests → Brownfield (with tests)
  Phases: Explore → Clarify → Specify (delta) → Test Plan → TDD → Verify

[Phase 1 — Explore]
  Reading src/api/handlers/, src/middleware/, tests/...
  Existing rate-limiting? None found.
  Test framework: pytest, ~120 tests, 78% coverage on src/api/.

[Phase 2 — Clarify] 3 questions:
  1. Per-IP, per-user, or per-API-key limits?
  2. In-memory or Redis-backed (multi-instance)?
  3. Reject with 429 + Retry-After, or queue?

> per-API-key, redis, 429 + Retry-After

[Phase 3 — Specify (delta)]
  REQ-01: WHEN a request arrives THE SYSTEM SHALL identify caller via API key.
  REQ-02: WHILE caller exceeds N req/min THE SYSTEM SHALL respond 429.
  ... (4 more)

[Phase 4 — Test Plan] 11 tests across 3 files. Approve? (y/n)
> y

[Phase 5 — TDD] Red-Green-Refactor for each test...
[Phase 6 — Verify] All 131 tests passing. Coverage on src/api/ now 84%.
```

Each skill follows the same shape: route → declare phases → run them →
verify. You always know what's coming next, and you can interrupt at any
phase boundary.

---

## Install

### Recommended: skills.sh

The [skills.sh](https://www.skills.sh) CLI handles installation for 55+
agents (Claude Code, Cursor, Codex, Copilot, OpenCode, Cline, Gemini,
Windsurf, and more) with a single command:

```bash
# Recommended: install globally so skills follow you across projects
npx skills add mohammaddaoudfarooqi/agent-engineering-skills -g

# Or: install into the current project only (omit -g)
npx skills add mohammaddaoudfarooqi/agent-engineering-skills

# Just one skill
npx skills add mohammaddaoudfarooqi/agent-engineering-skills -g --skill spec-driven-tdd

# Specific agents only
npx skills add mohammaddaoudfarooqi/agent-engineering-skills -g -a claude-code -a cursor
```

| Scope | Flag | Where | When |
| --- | --- | --- | --- |
| Global | `-g` | `~/.<agent>/skills/` | Everyday use across all projects. |
| Project | *(default)* | `./.<agent>/skills/` | Pin a version per-project or commit with the team. |

The CLI symlinks (or copies) the right files into each agent's expected
directory layout. See [`vercel-labs/skills`](https://github.com/vercel-labs/skills)
for the full CLI reference.

### Claude Code (manual)

If you prefer not to use the CLI:

```bash
# Clone anywhere
git clone https://github.com/mohammaddaoudfarooqi/agent-engineering-skills.git

# Symlink each skill into ~/.claude/skills/
cd agent-engineering-skills
for s in skills/*/; do
  ln -s "$PWD/$s" ~/.claude/skills/$(basename "$s")
done
```

This puts each `SKILL.md` exactly where Claude Code expects it:
`~/.claude/skills/<skill-name>/SKILL.md`.

### Other agents (manual)

Each skill is a directory containing `SKILL.md` (Anthropic skills format:
YAML frontmatter with `name` and `description`, plus a body). Most modern
AI coding agents accept this format directly or via a small adapter; the
skills.sh CLI above handles the adapter step automatically.

If you're loading manually into another agent, point it at
`skills/<skill-name>/SKILL.md` and let it discover `references/` from
relative paths.

---

## The Workflow Router Pattern

Every skill in this repo starts with a router that asks one question:
**what kind of task is this?**

```
Request + Repo state
        |
        v
+-----------------+
| Workflow Router |
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
                  Shared verification
```

*Each skill picks the subset of workflows that fits its domain.*
`spec-driven-tdd` uses Greenfield, Brownfield (×2), Refactor, and Bugfix.
`github-actions` and `doc-authoring` use Greenfield, Brownfield, and
Audit. The router collapses what would otherwise be four or five narrow
skills into one with a single discoverable entry point. All workflows
converge on a shared verification phase.

Read the full pattern: [`docs/workflow-router-pattern.md`](docs/workflow-router-pattern.md).

---

## Documentation

- [Getting Started](docs/getting-started.md) — install and run your first skill in ten minutes.
- [Workflow Router Pattern](docs/workflow-router-pattern.md) — the design pattern shared by every skill in this repo.
- [Skill Format Reference](docs/skill-format-reference.md) — frontmatter, body structure, validation rules. For contributors.
- [CONTRIBUTING](CONTRIBUTING.md) — how to propose a new skill.
- [CHANGELOG](CHANGELOG.md) — version history.

---

## What this is not

- **Not one-shot codegen.** Each skill runs a multi-phase workflow with
  human checkpoints. Approve the spec before tests are written. Approve
  the test plan before code is written. The skills slow the agent down
  on purpose.
- **Not a substitute for code review.** Generated specs, tests,
  workflows, and docs are starting points that still need a human to
  read them.
- **Not framework-specific.** The skills work across Python, JS/TS, Go,
  Rust, and other ecosystems. Domain-specific tooling (e.g. a particular
  test runner) is detected at the Discover/Explore phase.

---

## Built on

These skills wrap publicly known engineering methodologies, applied
consistently to AI coding agent workflows:

- [Diataxis](https://diataxis.fr) — Daniele Procida's documentation framework.
- [EARS notation](https://alistairmavin.com/ears/) — Easy Approach to Requirements Syntax.
- [Spec-driven development](https://github.com/github/spec-kit) — popularised by GitHub's spec-kit.
- Characterization tests — Michael Feathers, *Working Effectively with Legacy Code*.
- Red-Green-Refactor TDD — Kent Beck.

The skills don't claim to invent any of these. They package them as
agent-runnable workflows so your AI assistant follows the discipline by
default.

---

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). New skills are welcome if they
follow the Workflow Router pattern and address a distinct engineering
domain.

---

## Author

Mohammad Daoud Farooqi —
[GitHub](https://github.com/mohammaddaoudfarooqi) ·
[LinkedIn](https://www.linkedin.com/in/farooqimdd)
