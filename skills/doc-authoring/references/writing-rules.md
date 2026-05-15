# Documentation Writing Rules

Strict style rules enforced across all documentation. Apply these as
fixed constraints, not suggestions.

## Core Principles

1. **Accuracy over completeness.** A short, correct document beats a
   comprehensive, wrong one. Never state something you haven't verified
   against the code.

2. **Minimum viable documentation.** Write the least amount of documentation
   that provides the information the reader needs. Trim like a bonsai.

3. **Delete over update.** If content is outdated and not worth fixing,
   remove it. Dead documentation misleads.

4. **Code is authoritative.** When documentation conflicts with code,
   the code is correct. Fix the documentation, not the code (unless
   the code is the bug).

5. **Link, don't duplicate.** If information exists elsewhere, link to it.
   Never maintain the same fact in two places.

---

## Sentence-Level Rules

### Clarity

- **One idea per sentence.** If a sentence has "and" connecting two
  separate concepts, split it.
- **One idea per paragraph.** Each paragraph makes one point.
- **Short sentences.** Prefer 15-20 words. Maximum 30.
- **Active voice.** "The server returns a 404" not "A 404 is returned by the server."
- **Imperative for instructions.** "Run the command" not "You should run the command."
- **Specific subjects.** "The API server validates the token" not "It validates the token."

### Precision

- **Name things exactly.** Use the actual function name, variable name,
  file path. Not "the config file" but "`config/database.yml`".
- **State behavior, not intent.** "Returns an empty array when no results match"
  not "Tries to return results."
- **Quantify when possible.** "Retries 3 times with 1-second backoff" not
  "Retries several times."
- **Include the why only when it aids understanding.** Context is not filler.
  "Uses connection pooling (to avoid per-request TCP overhead)" is good.
  "Uses connection pooling because it's best practice" is filler.

### Words to eliminate

| Remove | Reason |
|--------|--------|
| just, simply, easily | Minimizes difficulty; unhelpful |
| very, really, extremely | Non-technical intensifiers |
| basically, essentially | Vague hedging |
| obviously, clearly, of course | Presumptuous |
| please, please note | Unnecessary politeness in technical docs |
| in order to | "To" is sufficient |
| utilize | "Use" is clearer |
| leverage | "Use" is clearer |
| we/our (in reference docs) | Use specific subjects; "we" is for tutorials only |

---

## Structure Rules

### Headings

- Use `##` for major sections, `###` for subsections. Avoid `####` or deeper
  unless necessary.
- Headings describe content, not categories. "Install dependencies" not "Installation."
- Headings are not sentences. No periods. No questions (unless Explanation type).
- Every heading must have content under it. No empty sections.
- Keep heading hierarchy flat. If nesting goes beyond 3 levels, restructure.

### Lists

- Use bullet lists for unordered items (features, options, requirements).
- Use numbered lists only for ordered sequences (steps in a procedure).
- One item per bullet. Do not pack paragraphs into list items.
- Start each bullet with a capital letter. End with a period only if it's
  a complete sentence.
- Keep list items parallel in structure (all start with verbs, or all
  start with nouns).

### Tables

- Use tables for structured data with 3+ columns (parameters, config options,
  error codes).
- Every table must have a header row.
- Keep cell content brief. If a cell needs more than one sentence, the
  information belongs in prose, not a table.
- Align columns for readability in the source Markdown.

### Code blocks

- Use fenced code blocks (triple backticks) for all code, commands, and output.
- Specify the language for syntax highlighting: ` ```bash `, ` ```python `, etc.
- Use `bash` for shell commands. Prefix commands with `$` only when mixing
  commands and output in the same block.
- For inline code: use backticks for filenames, function names, variable names,
  commands, environment variables, and any literal technical term.
  Examples: `` `npm start` ``, `` `CONFIG_PATH` ``, `` `UserService.create()` ``
- Every code example must be derived from actual source code. Never fabricate.
- Show expected output when it helps verify the command worked.

---

## Formatting Conventions

### Markdown

- Use ATX-style headings (`#`, `##`, `###`), not Setext-style (underlines).
- One blank line before and after headings, code blocks, and lists.
- No trailing whitespace.
- No HTML in Markdown unless absolutely necessary (e.g., collapsible sections).
- Links: prefer `[text](url)` inline style. Use reference-style `[text][ref]`
  only when the same URL appears multiple times.
- Images: include alt text that describes what the image shows.

### File naming

| File | Convention |
|------|-----------|
| README, LICENSE, CONTRIBUTING, CHANGELOG | UPPERCASE.md (at repo root) |
| docs/ folder files | lowercase-with-dashes.md |
| ADR files | `NNN-short-noun-phrase.md` |

---

## Content-Type-Specific Rules

### Commands and instructions

- Every command must be copy-pastable. No placeholders that require editing
  unless clearly marked: `[your-project-name]`.
- Show the command first, then explain what it does (if non-obvious).
- If a command has prerequisites, state them before the command.
- Group related commands. Don't interleave commands with long explanations.

### Environment variables

- Always use backtick formatting: `` `DATABASE_URL` ``
- Document: name, type, required/optional, default value, description.
- Prefer a table for 3+ variables.
- If the project has `.env.example`, keep it in sync with documentation.

### Version numbers

- State versions explicitly: "Node.js 18 or later" not "a recent version of Node."
- Pin versions in install commands when compatibility matters.
- When version ranges are acceptable, state the range: "Python 3.9-3.12."

### Error messages

- When documenting error handling, show the actual error message the user
  will see.
- Pair every documented error with its cause and resolution.

### URLs and links

- Internal links: use relative paths (`./docs/api-reference.md`).
- External links: use full URLs. Verify they resolve.
- Never use bare URLs in prose. Always use `[descriptive text](url)`.

---

## Quality Checklist

Apply this checklist to every document before considering it complete.

### Accuracy

- [ ] Every factual claim verified against source code
- [ ] Every command tested or verified against the codebase
- [ ] Every code example derived from actual code
- [ ] Version numbers match current manifest files
- [ ] Environment variable names match what the code reads
- [ ] File paths reference files that exist

### Clarity

- [ ] Each sentence conveys one idea
- [ ] No jargon without definition on first use
- [ ] Same term used for the same concept throughout
- [ ] No ambiguous pronouns ("it", "this", "that" without clear antecedent)
- [ ] Instructions use imperative voice

### Structure

- [ ] Headings describe their content
- [ ] Information is organized for scanning (lists, tables, code blocks)
- [ ] No wall-of-text paragraphs (max 3-5 sentences)
- [ ] Related information grouped together
- [ ] Prerequisite information comes before the steps that need it

### Completeness

- [ ] All public interfaces documented (for reference docs)
- [ ] All configuration options documented
- [ ] Error cases covered (not just the happy path)
- [ ] Links to related docs included

### Maintenance

- [ ] No duplicated content (link instead)
- [ ] No dead links
- [ ] Content is version-independent where possible (or version is stated)
- [ ] No TODO/TBD/FIXME placeholders remain

---

## Anti-Patterns

### Documentation anti-patterns to avoid

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Wall of text** | Unreadable, un-scannable | Break into headings, lists, code blocks |
| **FAQ section** | Becomes stale, mixes unrelated topics, delays proper documentation | Distribute answers into the correct Diataxis type |
| **Git log as changelog** | Too noisy, not human-readable | Curate into Keep a Changelog format |
| **Duplicate content** | Two places to update, guaranteed drift | Link to canonical source |
| **Placeholder docs** | "TODO: document this" erodes trust | Write now or don't create the file |
| **Screenshot-heavy** | Breaks on UI changes, inaccessible | Use text descriptions + code examples; screenshots only when essential |
| **Aspirational docs** | Documents planned features as if they exist | Document only what the code currently does |
| **Marketing tone** | "Our amazing API" provides no information | Factual, neutral descriptions only |
| **Pronoun soup** | "It calls it, which returns it" | Name the subject every time |

### Command example anti-patterns

| Bad | Good | Why |
|-----|------|-----|
| `run the test` | `` `npm test` `` | Not copy-pastable |
| `install the dependencies` | `` `pip install -r requirements.txt` `` | Exact command needed |
| `set the config variable` | `` `export DATABASE_URL=postgres://...` `` | Show the actual variable |
| `<your-value-here>` (no context) | `[your-api-key]` (with note: "Replace with your API key from the dashboard") | Placeholder must explain what to substitute |
