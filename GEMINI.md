# GEMINI.md

Context for Gemini CLI working in this repository.

## What this repo is

`agent-engineering-skills` is a small set of agent skills built on the
[Workflow Router pattern](docs/workflow-router-pattern.md). It covers the
core engineering loop: build → ship → document.

## Available skills

The three skills live under `skills/`:

- **`spec-driven-tdd`** — disciplined spec-driven TDD. Use when the user
  asks to add a feature, build a module, refactor with test preservation,
  or fix a complex bug.
- **`github-actions`** — production-grade GitHub Actions workflow
  generation. Use when the user asks for CI/CD, workflow YAML, security
  hardening of pipelines, or migration from another CI system.
- **`doc-authoring`** — Diataxis-classified documentation. Use when the
  user asks for a README, API reference, architecture docs, developer
  guide, or doc audit.

Each skill's `SKILL.md` contains a Workflow Router that classifies the
request and runs the matching phase pipeline. Read the SKILL.md before
invoking the skill so you know which workflow you're in and what phase
comes next.

## Working in this repo

- Skills are markdown only. No build step, no package manager.
- Validation runs in CI via `.github/workflows/validate-skills.yml`.
- Reference docs live under `skills/<skill>/references/` and are linked
  from each skill's phases.
- The `docs/workflow-router-pattern.md` file documents the unifying
  pattern any new skill must follow.

## When proposing changes

- Updating an existing skill: keep the Workflow Router structure intact.
  Phase names are the user-facing contract.
- Adding a new skill: follow `CONTRIBUTING.md`. New skills must use the
  Workflow Router pattern and address a distinct engineering domain.
