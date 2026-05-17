# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-05-17

### Added

`spec-driven-tdd` gains a second axis of verification — **realism** —
alongside the existing coverage axis. New required artifacts and gates
catch the "passes every test, fails on first real request" failure mode
on multi-component systems. Driven by a real-world bug report from a
greenfield greenfield agent-system buildout.

#### SKILL.md

- New "Two Axes of Verification" section after the Workflow Router.
- Phase 2 (Specify) now requires a **Premortem** (top 5 failure modes
  with EARS mitigations) in `requirements.md` and a **Boundary Inventory**
  (every sync→async / mock→real / FE→BE / in-process→out-of-process
  seam) in `design.md`.
- Phase 3 (Bootstrap) recommends a thinnest-possible live integration
  test against the riskiest external boundary at bootstrap.
- Phase 4 (Test Plan) requires a **smoke / functional tier** (≥ 1 test
  per top-level user story), an **outcome-vs-side-effect rule**, a
  **mock-parity contract** per fake, a **test isolation contract** for
  state-mutating tests, and a **two-axis coverage** REQ (aggregate +
  per-module floors).
- Phase 5 (TDD Loop) gains a mandatory **Demo gate**: each task ends
  with curl / playwright-cli / screenshot evidence pasted into
  `tasks.md`, or an explicit "no demo applicable" note.
- Phase 6 (Verification) checklist split into Coverage-axis and
  Realism-axis sub-checklists.
- New "What this skill does not catch" candor section.

#### References

- `references/spec-template.md` — Premortem template, Boundary
  Inventory table, two-axis coverage REQ template added.
- `references/test-plan.md` — five-tier test model, outcome-vs-side-
  effect rule, mock-parity rule, test isolation rule, language-specific
  quality-gate examples.
- `references/test-tiers.md` (new) — definitions and per-stack
  examples of unit / integration / smoke / quality_gate /
  characterization tiers.
- `references/mock-parity.md` (new) — patterns for proving a fake
  matches the real dependency (mongomock vs AsyncMongoClient, FakeLLM
  vs VCR, moto vs LocalStack, etc.).
- `references/demo-tools.md` (new) — phase-end demo tooling per stack:
  curl, httpie, playwright-cli, websocat, k6, etc.

### Changed

- `spec-driven-tdd` SKILL.md is approximately 30% longer; the new
  realism artifacts are required, not advisory. Backwards-compatible
  for projects already producing them; new specs will have additional
  required sections.

## [0.1.0] - 2026-05-16

### Added
- Initial release with three skills built on the Workflow Router pattern:
  - `spec-driven-tdd` — spec-first TDD with greenfield, brownfield, refactor, and bugfix workflows.
  - `github-actions` — production-grade workflow generation with greenfield, brownfield, and audit workflows.
  - `doc-authoring` — Diataxis-classified documentation with greenfield, brownfield, and audit workflows.
- `docs/workflow-router-pattern.md` describing the unifying pattern.
- Apache-2.0 license, NOTICE, README, CONTRIBUTING.

[Unreleased]: https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/releases/tag/v0.2.0
[0.1.0]: https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/releases/tag/v0.1.0
