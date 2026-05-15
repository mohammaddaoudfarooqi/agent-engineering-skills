# Contributing

Thanks for your interest in contributing to `agent-engineering-skills`.

## Ways to contribute

- **Start a [Discussion](https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/discussions)** —
  for open-ended questions, ideas, or "should I open an issue for this?"
  uncertainty. Use *Q&A* for usage help, *Ideas* for feature proposals,
  *Show and tell* if you've used a skill on something interesting.
- **File an [issue](https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/issues)** —
  bug reports, unclear instructions in a SKILL.md, failed scenarios,
  or concrete feature requests with a clear definition of done.
- **Improve an existing skill** — sharper rules, better templates, fixes
  for cases the router gets wrong, additional reference material.
- **Propose a new skill** — see below.

## Proposing a new skill

This repo is intentionally small. New skills earn inclusion by meeting
all three criteria:

1. **Distinct engineering domain.** It addresses a problem the existing
   three skills don't already cover. "Better tests" doesn't qualify;
   "performance profiling workflows" might.

2. **Workflow Router pattern.** The skill must start with a router that
   classifies the request into 3–5 workflow types and runs only the
   phases that workflow needs. Read
   [`docs/workflow-router-pattern.md`](docs/workflow-router-pattern.md)
   first.

3. **Public methodology, original packaging.** Wrap a known engineering
   discipline (or compose several). Cite sources. Don't reinvent
   methodology — focus the work on the agent-orchestration layer.

Start a [Discussion](https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/discussions/new?category=ideas)
under *Ideas* with a one-page proposal before writing the skill. We'll
talk through scope and naming first. Once the proposal converges, an
issue is opened to track the implementation work.

## Skill format

Each skill is a directory under `skills/` containing:

```
skills/<skill-name>/
  SKILL.md           # YAML frontmatter + body
  references/        # supporting docs the skill links to
    *.md
```

`SKILL.md` frontmatter requires:

```yaml
---
name: <kebab-case-name>
description: >
  <one to three sentences. Lead with what the skill does, end with when to
  use it and when NOT to use it.>
---
```

The body must include a Workflow Router section near the top.

## Pull requests

- Branch from `main`. Keep PRs focused — one skill change or one fix per
  PR.
- Include a short rationale in the description: what changed, why,
  what scenarios you tested.
- For SKILL.md changes, ideally run the skill end-to-end against a real
  project and note the result in the PR. At minimum, walk through the
  router and phases manually for a representative scenario.
- For documentation, link to the doc you edited and explain the gap it
  closed.

## Code of Conduct

Be kind, be specific, be technical. Disagree on substance, not on people.
The maintainer reserves the right to remove abusive comments and ban
repeat offenders.

## License

By contributing, you agree that your contributions will be licensed
under the Apache License 2.0 (see [LICENSE](LICENSE)). Apache-2.0
includes an explicit patent grant from contributors — by submitting a PR
you grant that license for your contribution.

## Questions

For usage help or open-ended questions, ask in
[Discussions → Q&A](https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/discussions/categories/q-a).
For confirmed bugs and concrete feature requests, open an
[issue](https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/issues).
