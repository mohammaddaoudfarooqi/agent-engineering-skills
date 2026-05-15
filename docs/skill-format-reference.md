# Skill Format Reference

Specification for a skill in this repository. For motivation and design
context, see [Workflow Router Pattern](workflow-router-pattern.md). For
contribution process, see [CONTRIBUTING](../CONTRIBUTING.md).

## Directory layout

```
skills/<skill-name>/
  SKILL.md
  references/
    <topic>.md
    ...
```

`<skill-name>` MUST be kebab-case and MUST match the `name` field in the
SKILL.md frontmatter.

`references/` is required. It contains supporting documents linked from
SKILL.md phases. A skill with no reference documents MAY ship an empty
`references/` directory.

## SKILL.md structure

```
---
name: <kebab-case-name>
description: >
  <body of description, 100-400 words>
---

# <Title Case Skill Name>

<one-paragraph overview>

## Workflow Router

<router flowchart and signal table>

---

## Phase 0: <Phase Name>

<phase content>

## Phase 1: <Phase Name>

<phase content>

...

## Reference Files

<links to references/*.md>
```

## Frontmatter

YAML between `---` fences at the top of SKILL.md.

| Field | Required | Type | Constraint |
| --- | --- | --- | --- |
| `name` | Yes | string | Kebab-case. Must match the directory name. |
| `description` | Yes | string | One block; folded scalar (`>`) recommended. See *Description content* below. |

Additional fields are permitted but not required. The CI validator
ignores them.

### Description content

The description is the agent's primary signal for auto-invocation.
Order content as follows:

1. **Summary** — one sentence stating what the skill does.
2. **Workflow coverage** — sentence listing which workflows the skill
   handles (greenfield, brownfield, refactor, bugfix, audit, or a subset).
3. **Use when** — sentence beginning with "Use when..." listing concrete
   scenarios.
4. **Triggers on phrases** — paragraph beginning "Triggers on phrases
   like..." enumerating specific phrases users say.
5. **Triggers when** — paragraph beginning "Triggers when..." listing
   file patterns, code artifacts, or repo states that signal the skill
   applies.
6. **Do NOT use** — sentence beginning "Do NOT use..." listing exclusions.

The combined description SHOULD be between 600 and 2000 characters.
Skills in this repository range from 1500 to 1700 characters.

## Body

### Title

Top-level heading (`# Title Case Skill Name`) immediately after the
frontmatter. Title MAY differ from the `name` field (e.g., `name:
spec-driven-tdd`, title: `Spec-Driven Test-Driven Development`).

### Workflow Router section (required)

A `## Workflow Router` heading containing:

1. A flowchart (ASCII or markdown table) classifying the request into
   workflow types.
2. Phases listed inline for each workflow (e.g., `Phases: Analyze ->
   Plan -> Write -> Verify`).
3. A signal table mapping user phrases to workflow types.

The router MUST be the first major section after the title. The CI
validator checks for the literal string `## Workflow Router` in the
body.

### Phase sections

One `## Phase N: <Name>` heading per phase. Phase numbering starts at
0. Each phase SHOULD include:

- What the phase produces.
- Inputs required from prior phases.
- Decision criteria when the phase has variants per workflow.
- Links to relevant `references/` files.

### Convergent phases

All workflows MUST converge on a final verification phase named
`Verify`. The verification phase MUST include a checklist that runs
regardless of workflow.

### Reference Files section

A `## Reference Files` heading at the end of the body listing each file
under `references/` with a one-line description and a relative link.

## File size

| Limit | Threshold | Source |
| --- | --- | --- |
| SKILL.md soft limit | 50,000 characters | [agent-docs-spec](https://agentdocsspec.com) page-size guidance |
| SKILL.md hard limit | 100,000 characters | Claude Code web fetch truncation |
| Total skill (SKILL.md + references) | No enforced limit | Skills.sh CLI installs full directory |

The CI validator does not enforce size limits in v0.1. Authors SHOULD
keep SKILL.md under 50,000 characters and split overflow into
`references/` files.

## CI validation rules

The `validate-skills` workflow at
`.github/workflows/validate-skills.yml` enforces:

| Rule | Failure mode |
| --- | --- |
| `skills/<dir>/SKILL.md` exists | Missing file |
| YAML frontmatter parseable, fenced by `---` | Missing or malformed frontmatter |
| `name` field present | Missing frontmatter field |
| `description` field present | Missing frontmatter field |
| `name` field equals directory name | Name/directory mismatch |
| Body contains the literal string `## Workflow Router` | Missing required section |

A markdown link checker (lychee) runs as a second job and fails on dead
links in any `*.md` file in the repo.

## Naming

| Element | Convention |
| --- | --- |
| Skill directory | kebab-case |
| `name` frontmatter field | kebab-case, matches directory |
| Title heading | Title Case |
| Phase headings | `Phase N: Title Case` |
| Reference file names | kebab-case, lowercase, `.md` extension |

## Distribution format

Skills are distributed as plain Git repositories. The
[skills.sh](https://www.skills.sh) CLI indexes any public GitHub repo
that follows this format. Install command:

```bash
npx skills add <owner>/<repo>
```

The CLI symlinks `skills/<skill-name>/` into the agent's expected
directory (e.g., `.claude/skills/<skill-name>/` for Claude Code).

## Compatibility

Skills following this format are compatible with agents that read the
[Anthropic skills format](https://docs.anthropic.com): `name` and
`description` frontmatter, body containing instructions for the agent.
Agents that auto-discover skills from a known directory (Claude Code,
Cursor, Cline, Codex, Gemini CLI, OpenCode, and others supported by the
skills.sh CLI) will load the skill without further configuration.

The Workflow Router section is a convention specific to this repository
and is not required by external agents. It enforces a contract for
authors maintaining skills in this repo.

## Versioning

Skills inherit the repository version. The repository follows
[Semantic Versioning](https://semver.org). Breaking changes to a
skill's router (changed phase names, removed workflow types) MUST bump
the major version. New phases, new workflows, or expanded reference
files are minor changes. Wording fixes, clarifications, and reference
file additions are patch changes.

## Examples

The three skills in this repository conform to this specification:

- [`skills/spec-driven-tdd/`](../skills/spec-driven-tdd/) — five
  workflows, Clarify-Specify-Test Plan-TDD-Verify pipeline.
- [`skills/github-actions/`](../skills/github-actions/) — three
  workflows, Discover-Plan-Write-Verify pipeline.
- [`skills/doc-authoring/`](../skills/doc-authoring/) — six workflow
  variants, Analyze-Plan-Write-Verify pipeline.
