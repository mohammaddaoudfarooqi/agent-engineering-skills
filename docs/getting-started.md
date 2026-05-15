# Getting Started

In the next ten minutes you'll install `agent-engineering-skills`, run
`doc-authoring` against a small project, and watch the Workflow Router
pick a path through it. By the end you'll have generated a real README
and seen what the skills do.

> **Note on output shown below.** Phase labels, prompts, install paths,
> and CLI output are *illustrative*. The exact wording, ordering, and
> layout depend on which agent and CLI version you use, and may evolve
> over time. The shape of the workflow — router → typed phases → verify
> — is what's invariant. If your output differs cosmetically, that's
> expected.

## Before you start

You need:

- An AI coding agent with skills support: **Claude Code**, **Cursor**,
  **Codex**, **Cline**, **Gemini CLI**, **OpenCode**, or any agent the
  [skills.sh](https://www.skills.sh) CLI supports.
- **Node.js 18 or newer** (the install CLI is `npx`-based).
- A small project to point the skill at — your own, or use the throwaway
  example below.

## Step 1: Create a throwaway project

Skip this if you have a small project handy. Otherwise, create one:

```bash
mkdir -p ~/aes-tutorial && cd ~/aes-tutorial
cat > greeter.py <<'EOF'
"""A trivial greeter module."""

def greet(name: str, shout: bool = False) -> str:
    """Return a greeting for `name`. If `shout` is True, uppercase it."""
    msg = f"Hello, {name}!"
    return msg.upper() if shout else msg


if __name__ == "__main__":
    print(greet("world"))
EOF
git init -q
git add greeter.py
git commit -q -m "Initial commit"
```

You now have a one-file project with no documentation. Perfect.

## Step 2: Install the skills

You have two install scopes. **For this tutorial, install globally** so
the skills are available across every project you work in:

```bash
npx skills add mohammaddaoudfarooqi/agent-engineering-skills -g
```

| Scope | Flag | Where it installs | When to use |
| --- | --- | --- | --- |
| **Global** *(recommended)* | `-g` | `~/.<agent>/skills/` | Everyday use — skills follow you across projects. |
| Project | *(default)* | `./.<agent>/skills/` (run from project root) | Pin specific skill versions per project, or commit them with the project so teammates get the same set. |

The CLI asks which skills to install and which agents to install them
for. Pick all three skills and your agent. The CLI symlinks each skill
into the directory your agent expects (e.g., `~/.claude/skills/` for
Claude Code in global mode).

After it finishes, the CLI prints a summary of what was installed and
where. The exact format depends on your CLI version.

## Step 3: Invoke `doc-authoring`

Open your agent in `~/aes-tutorial/` and ask:

> Use the doc-authoring skill to write a README for this project.

The agent will route through the skill's Workflow Router. Because no
README exists yet, it picks the **Greenfield** workflow and announces
its phases:

```
[Workflow Router]
  No existing docs → Greenfield
  Phases: Analyze → Plan → Write → Verify
```

## Step 4: Watch the phases run

The agent runs each phase in order, pausing at natural seams.

**Phase 0 — Analyze.** It reads `greeter.py`, notes the language
(Python), the public interface (`greet`), the entry point (the
`__main__` block), and that no test framework or CI is present.

**Phase 1 — Plan.** Because there's no API to document and the project
is tiny, it produces a minimal plan:

```
Documentation Plan
- README.md (hybrid) — CREATE
```

It asks for your approval. Say yes.

**Phase 2 — Write.** It generates a `README.md` with a title, one-line
description, install instructions, a runnable quickstart example, and
a Usage section showing the `greet` function with both `shout=False`
and `shout=True`. Every example is derived from the actual code.

**Phase 3 — Verify.** It re-reads the generated README against the code
and reports its checks: example commands run, function signatures
match, no fabricated content.

## Step 5: Inspect the result

```bash
cat README.md
```

You should see a clean README that describes `greeter.py` accurately.
The example in the Quickstart matches the function signature. There's
no invented API, no placeholder text, no fabricated configuration.

## What just happened

You used a single skill that adapted to your project's state:

1. The **Workflow Router** asked one question: *do docs already exist?*
   No → it picked the Greenfield workflow.
2. The skill ran four typed phases. You knew what was coming next at
   every step.
3. The **Verify** phase was a quality floor: no matter which workflow
   the router had picked, every result is checked against the code.

This is the same shape every skill in this repo follows. `spec-driven-tdd`
adds Clarify and Test Plan phases. `github-actions` adds Discover and
Audit Existing phases. The vocabulary stays consistent so you always
know where you are.

## What to try next

Now repeat the experiment with the other skills:

- **Add a feature.** Ask the agent to use `spec-driven-tdd` to add a
  `farewell` function to `greeter.py`. Watch it route to Brownfield (no
  tests), bootstrap a test file, then run TDD on the new function.
- **Add CI.** Ask `github-actions` to set up a GitHub Actions workflow
  for the project. It'll route to Greenfield (no `.github/workflows/`)
  and produce a security-hardened `ci.yml`.
- **Refresh the docs.** After you change `greeter.py`, ask `doc-authoring`
  to refresh the README. This time it routes to Doc Refresh: drift
  detection finds what's stale, and writes only the delta.

## Where to go from here

- [README](../README.md) — full skill catalog and install options.
- [Workflow Router Pattern](workflow-router-pattern.md) — the theory
  behind why the router exists and how to design your own.
- [Skill Format Reference](skill-format-reference.md) — for contributors
  building new skills.
- [CONTRIBUTING](../CONTRIBUTING.md) — proposing a new skill.

If something didn't work the way this guide describes, please [open an
issue](https://github.com/mohammaddaoudfarooqi/agent-engineering-skills/issues/new?template=bug_report.yml).
The issue template will collect the agent, model, and prompt that
triggered it.
