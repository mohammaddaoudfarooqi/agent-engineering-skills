---
name: doc-authoring
description: >
  Documentation authoring for AI coding agents. Analyzes a repo and
  generates docs classified by Diataxis type (tutorial, how-to,
  reference, explanation), kept synchronized with the code.
  Handles greenfield (no docs exist), brownfield (refresh, enhance,
  rewrite), and doc audits with workflow-specific guidance.
  Use when the user requests project documentation: a README, API
  reference, architecture doc, developer guide, contributing guide,
  changelog, ADR, onboarding doc, or any technical writing tied to a
  codebase. Also use when existing docs need an audit, refresh, or
  restructure.
  Triggers on phrases like "write a README", "document this project",
  "API reference", "architecture doc", "developer guide", "tutorial",
  "how-to", "audit our docs", "what docs are missing", "refresh the
  docs", "Diataxis", "write a CHANGELOG", "onboarding doc", or "ADR".
  Triggers when creating or editing `README.md`, `CONTRIBUTING.md`,
  `CHANGELOG.md`, files under `docs/`, `mkdocs.yml`, Docusaurus or
  Sphinx config, or any markdown paired with code. Also triggers when
  a public API, CLI flag, config option, or env var changes and the
  user wants docs kept in sync.
  Do NOT use for standalone prose, marketing copy, blog posts, design
  documents or RFCs unrelated to a codebase, or documents whose source
  of truth is not source code.
---

# Documentation Authoring

Generate accurate, maintainable documentation from systematic codebase analysis.
Every document is classified by type, written to strict rules, and verified
against the code.

## Workflow Router

Determine the workflow type before starting. This drives which phases apply,
which templates to use, and whether to write from scratch or edit existing docs.

```
Do any docs already exist?
  |
  NO --> GREENFIELD DOCS (new project, no docs)
  |       Phases: Analyze -> Plan -> Write -> Verify
  |
  YES --> What type of change?
           |
           +-> Code changed, docs not updated
           |     --> DOC REFRESH (brownfield)
           |     Phases: Analyze -> Audit Existing -> Drift Detection -> Write (delta) -> Verify
           |
           +-> New feature/module needs documenting
           |     --> DOC ENHANCEMENT (brownfield)
           |     Phases: Analyze -> Audit Existing -> Plan (delta) -> Write -> Verify
           |
           +-> Docs exist but quality/structure is poor
           |     --> DOC AUDIT (brownfield)
           |     Phases: Analyze -> Audit Existing -> Plan -> Rewrite -> Verify
           |
           +-> User wants a specific doc rewritten
           |     --> TARGETED REWRITE (brownfield)
           |     Phases: Analyze -> Audit Existing (single doc) -> Write (replace) -> Verify
           |
           +-> User wants a specific new doc (README, API ref, etc.)
           |     --> TARGETED NEW DOC
           |     Phases: Analyze -> Write (specific) -> Verify
           |
           +-> Version bump or migration
                 --> VERSION UPDATE (brownfield)
                 Phases: Analyze -> Audit Existing -> Drift Detection -> Write (delta) -> Verify
```

**Signal detection:**

| Signal in request | Likely type |
|-------------------|-------------|
| "Write docs", "document this", "add documentation" (no docs exist) | Greenfield |
| "Write docs", "document this" (docs already exist) | Doc Enhancement |
| "README", "write a README" (none exists) | Targeted New Doc |
| "Rewrite the README", "redo the README" (one exists) | Targeted Rewrite |
| "API docs", "API reference", "document the API" | Targeted New Doc or Enhancement |
| "Architecture", "design doc", "how does this work" | Targeted New Doc |
| "Docs are outdated", "update the docs", "sync the docs" | Doc Refresh |
| "Review the docs", "audit", "docs are wrong" | Doc Audit |
| "Upgrade to v2", "migration guide", "breaking changes" | Version Update |
| "Contributing guide", "CONTRIBUTING.md" | Targeted New Doc |
| "Changelog", "release notes" | Targeted New Doc |
| "Developer guide", "onboarding" | Targeted New Doc |

---

## Phase 0: Analyze Repository

**Always run this phase.** Before writing anything, build a complete picture
of the codebase. See [references/repo-analysis.md](references/repo-analysis.md)
for the full analysis method.

### What to discover

1. **Language and runtime**: Identify from manifest files (package.json, pyproject.toml, go.mod, etc.)
2. **Framework and libraries**: Scan dependencies for framework signatures
3. **Architecture**: Directory layout, monorepo indicators, service boundaries
4. **Entry points**: main() functions, CLI interfaces, HTTP servers, scripts
5. **Public interfaces**: Exported functions, API endpoints, CLI commands
6. **Configuration**: Environment variables, config files, feature flags
7. **Data models**: ORM definitions, schemas, migrations
8. **Tests and CI**: Test frameworks, CI/CD config, build commands
9. **Existing documentation**: README, docs/ folder, inline docstrings, CLAUDE.md, wiki references
10. **Dependencies**: External services, databases, APIs consumed

### How to explore

- Read README.md and CLAUDE.md first (if they exist)
- Read manifest files (package.json, pyproject.toml, go.mod, Cargo.toml, etc.)
- Scan directory structure to identify modules and services
- Read 2-3 key source files to absorb patterns and conventions
- Check for existing doc infrastructure (Sphinx conf, JSDoc config, OpenAPI specs)
- Look for CI config that builds or tests documentation

### Output: Repository Profile

Produce a mental model (do not write a file unless requested):

```
Language: [language and version]
Framework: [framework and version]
Architecture: [monolith, microservices, library, CLI tool, etc.]
Entry points: [main files, CLI commands, HTTP servers]
Public API surface: [exported functions, endpoints, commands]
Config: [env vars, config files]
Test setup: [framework, runner command, coverage]
Existing docs: [what exists, what's missing, what's stale]
Doc infrastructure: [generators, linters, CI checks]
```

This profile informs every subsequent phase.

---

## Phase 1: Audit Existing Documentation (Brownfield Only)

**Skip this phase for greenfield projects.**

Before planning or writing anything, understand what documentation already
exists and assess its state. This is the brownfield equivalent of exploring
existing code before modifying it.

### Read every existing doc

For each doc file found in Phase 0:

1. **Read it fully.** Do not skim. Absorb its structure, voice, and content.
2. **Classify its Diataxis type** (Tutorial, How-to, Reference, Explanation, or Hybrid).
3. **Note its conventions**: heading style, code block formatting, terminology,
   section ordering, tone.
4. **Assess accuracy** by spot-checking 3-5 factual claims against source code.
5. **Assess completeness** by comparing documented interfaces against actual
   public interfaces found in Phase 0.

### Existing doc inventory

Build a mental inventory (do not write a file unless requested):

```
Existing Documentation Inventory
  [file path] | [diataxis type] | [accuracy: high/medium/low] | [completeness: high/medium/low]
  README.md   | hybrid          | medium (install cmd outdated) | low (missing API section)
  docs/api.md | reference       | low (3 endpoints missing)     | medium
  ...
```

### Identify what to preserve

**Critical for brownfield.** Existing docs may contain valuable content that
cannot be reconstructed from code alone:

| Content Type | Example | Preserve? |
|-------------|---------|-----------|
| Domain knowledge | Business rule explanations | Always |
| Design rationale | "We chose X because Y" | Always |
| User-authored context | Gotchas, tips, warnings | Always |
| Custom sections | Project-specific categories | Default yes |
| Historical notes | Migration history, deprecation context | Yes unless user says remove |
| Stale facts | Wrong version, renamed function | Replace with correct facts |
| Fabricated content | Documented endpoint that doesn't exist | Remove |
| Dead references | Links to deleted files | Remove |

**Rule: When in doubt, preserve.** You can always delete later; you cannot
reconstruct domain knowledge the original author had.

### Brownfield conventions contract

Extract the writing conventions from existing docs and follow them:

- **Heading style**: What levels are used? Are they noun phrases or verb phrases?
- **Code block language tags**: `bash` vs `shell` vs `sh`? `javascript` vs `js`?
- **Terminology**: What terms does the project use? ("service" vs "server",
  "endpoint" vs "route", "config" vs "configuration")
- **Section ordering**: What comes first? How are things grouped?
- **Tone**: Formal or casual? First-person or impersonal?

**Match existing conventions.** Do NOT impose new conventions on a brownfield
project unless the user explicitly requests a style change. Consistency within
the existing docs is more valuable than adherence to an ideal standard.

---

## Phase 2: Plan Documentation

### Greenfield planning

Derive a full documentation plan from the repository analysis.

#### Diataxis classification

Every document must be classified as exactly ONE of four types. See
[references/diataxis-guide.md](references/diataxis-guide.md) for full rules.

| Type | Purpose | User is... | Voice |
|------|---------|-----------|-------|
| **Tutorial** | Guided learning experience | Studying | "First, do x. Now do y." |
| **How-to Guide** | Solve a specific problem | Working | "To achieve x, do y." |
| **Reference** | Technical description for lookup | Working | "Returns x. Accepts y." |
| **Explanation** | Context, reasoning, background | Studying | "The reason for x is..." |

**Critical rule: Never mix types.** A tutorial must not explain. A reference
must not instruct. A how-to must not teach. Link between types instead.

#### Documentation plan algorithm

1. **Always include:** README.md (hybrid: summary + links to other docs)
2. **For each knowledge area, decide if a separate doc is needed:**

| Condition | Document | Diataxis Type |
|-----------|----------|---------------|
| Project exists | README.md | Hybrid (summary + pointers) |
| Nontrivial setup | docs/getting-started.md | Tutorial |
| Public API or library | docs/api-reference.md | Reference |
| CLI tool | docs/usage.md | Reference + How-to |
| Multi-component system | docs/architecture.md | Explanation |
| Complex design decisions | docs/adr/NNN-title.md | Explanation |
| External contributors expected | CONTRIBUTING.md | How-to |
| Versioned releases | CHANGELOG.md | Reference |
| Complex config/deployment | docs/deployment.md | How-to |
| Internal development | docs/developer-guide.md | How-to |
| Test infrastructure | docs/testing.md | How-to |

3. **Omit irrelevant sections.** No API means no API docs. No CLI means no CLI usage.
4. **Merge or split by project size:**
   - Small projects: combine Installation, Usage, and Overview in README
   - Large projects: separate into focused files under docs/
5. **Order by audience need:** Getting Started before deep reference. Overview before architecture.

#### Plan output

```
Documentation Plan
- README.md (hybrid) — CREATE
- docs/getting-started.md (tutorial) — CREATE
- docs/architecture.md (explanation) — CREATE
- docs/api-reference.md (reference) — CREATE
- CONTRIBUTING.md (how-to) — CREATE
```

Present the plan to the user for approval before writing.

### Brownfield planning

**Use the doc inventory from Phase 1.** Do NOT plan from scratch. Plan only
what changes.

#### Doc Refresh: Drift detection

Compare each existing doc against current code to identify drift. See
[references/drift-detection.md](references/drift-detection.md) for the full method.

For each existing doc, produce a drift report:

```
Drift Report: [file path]
  STALE:    [fact in doc] → [current fact in code]
  MISSING:  [interface/feature not documented]
  DEAD:     [documented item that no longer exists]
  ACCURATE: [sections that are still correct]
```

#### Doc Enhancement: Delta plan

When adding documentation for new features/modules:

1. Identify which EXISTING docs need updating (e.g., README features list,
   API reference for new endpoints)
2. Identify which NEW docs need creating
3. For each existing doc, specify ONLY what changes — do not rewrite

```
Documentation Delta Plan
- README.md (hybrid) — MODIFY: add new feature to Features section, update Quick Start
- docs/api-reference.md (reference) — MODIFY: add POST /widgets endpoint
- docs/widget-guide.md (how-to) — CREATE: new guide for widget feature
```

#### Doc Audit: Restructure plan

When docs exist but quality is poor:

1. List all issues found in Phase 1 with severity
2. Propose fixes grouped by severity (Critical → High → Medium → Low)
3. For each doc, specify: KEEP (as-is), EDIT (fix specific issues),
   REWRITE (replace with new version), or DELETE (remove dead doc)
4. Present to user for approval — let them decide scope

```
Audit Findings & Plan
  README.md:
    - [Critical] Install command references removed package → EDIT: fix command
    - [High] Missing API section → EDIT: add section
    - [Low] Mixed tutorial and reference content → EDIT: extract tutorial to separate doc
  docs/old-guide.md:
    - [Critical] Entire doc references removed feature → DELETE
  docs/api.md:
    - [Medium] 3 endpoints undocumented → EDIT: add missing endpoints
```

#### Targeted Rewrite: Single-doc plan

When the user asks to rewrite a specific existing doc:

1. Read the existing doc fully (done in Phase 1)
2. Identify what to PRESERVE (domain knowledge, rationale, custom sections)
3. Identify what to REPLACE (stale facts, broken examples, wrong structure)
4. Identify what to ADD (missing content)
5. Present the plan: "I will keep X, replace Y, add Z"

```
Rewrite Plan: README.md
  PRESERVE: Project description, design rationale in Architecture section
  REPLACE:  Install commands (outdated), Quick Start (broken example),
            Features list (missing 3 features)
  ADD:      Configuration section, link to new API docs
  REMOVE:   FAQ section (answers belong in specific docs)
```

**Get user approval before rewriting.**

#### Version Update: Migration plan

When documentation needs updating for a version bump:

1. Identify all version-specific references in existing docs
2. Identify breaking changes (from changelog, git diff, or user input)
3. Plan updates grouped by: version numbers, API changes, config changes,
   behavioral changes, deprecation notices

```
Version Update Plan: v1 → v2
  Version references: README.md (3), docs/install.md (2), docs/api.md (header)
  Breaking changes:
    - POST /users now requires `email` field → docs/api-reference.md
    - Config key `db_host` renamed to `database_url` → docs/deployment.md, README.md
  Deprecations:
    - GET /users/search deprecated in favor of query params on GET /users → docs/api-reference.md
  New features:
    - Batch API endpoints → docs/api-reference.md (add section)
```

Present the plan to the user for approval before writing.

---

## Phase 3: Write Documentation

### Greenfield: Write from scratch

Write each planned document using the appropriate template and writing rules.
See [references/doc-templates.md](references/doc-templates.md) for all templates.
See [references/writing-rules.md](references/writing-rules.md) for style rules.

#### Writing sequence

Write documents in this order:
1. **README.md** first (it's the front door; forces you to articulate purpose)
2. **Getting Started** (validates that setup works)
3. **Reference docs** (API, CLI usage — factual, derived from code)
4. **How-to guides** (developer guide, deployment, contributing)
5. **Explanation docs** (architecture, design decisions)
6. **Changelog** (if applicable)

#### Per-document process (greenfield)

For each document:

1. **Classify**: Confirm Diataxis type. Apply that type's rules strictly.
2. **Gather facts**: Read the relevant source code, config, and tests.
   Never write about code you haven't read.
3. **Draft**: Use the template from references/doc-templates.md.
   Follow the writing rules from references/writing-rules.md.
4. **Verify examples**: Every code example, command, or snippet must be
   derived from actual code. Never fabricate examples.
5. **Cross-reference**: Link to related docs. Never duplicate content
   that belongs in another document.

### Brownfield: Edit existing docs

**Read before writing.** For every doc you modify, read it fully first
(already done in Phase 1). Never modify a doc you haven't read.

#### Per-document process (brownfield)

For each document being modified:

1. **Re-read the doc.** Confirm your understanding from Phase 1.
2. **Apply only planned changes.** Follow the delta plan from Phase 2.
   Do NOT restructure, restyle, or "improve" sections outside scope.
3. **Match existing conventions.** Use the same heading style, terminology,
   code block formatting, and tone discovered in Phase 1.
4. **Preserve valuable content.** Domain knowledge, design rationale, user
   tips, and custom sections survive unless explicitly marked for removal.
5. **Verify every edit.** Each changed fact must be verified against code.
6. **Minimize diff.** Change only what needs changing. Smaller diffs are
   easier to review and less likely to introduce errors.

#### Brownfield-specific rules

- **Edit, don't rewrite.** Unless the user requests a rewrite, modify the
  existing doc surgically. Fix the wrong facts; leave correct content alone.
- **Match existing style.** If the existing README uses `##` for sections and
  `bash` for code blocks, continue that pattern. Do NOT switch to `###` or
  `shell` because you prefer it.
- **Preserve structure.** Keep the existing section ordering unless it's part
  of the planned changes. Users have muscle memory for where information lives.
- **Preserve authorship signals.** If existing docs have a distinctive voice,
  project-specific terminology, or custom sections, maintain them. The original
  author's domain knowledge is irreplaceable.
- **Never silently delete content.** If a section needs removal, note it in
  your plan and confirm with the user. Content that looks outdated may contain
  context you can't reconstruct from code.
- **Refactor only what you're asked to change.** Do NOT improve adjacent
  sections, fix unrelated formatting, or add sections not in the plan.
  This mirrors the TDD principle: "refactor only what you wrote."

#### Targeted Rewrite process

When rewriting a specific doc (user explicitly requested):

1. **Create from the approved plan.** Write the new version using the
   PRESERVE/REPLACE/ADD/REMOVE breakdown from Phase 2.
2. **Start from the template** in references/doc-templates.md, then
   transplant preserved content into the new structure.
3. **For preserved content**: Copy it verbatim. Do not rephrase domain
   knowledge or design rationale unless it's factually wrong.
4. **For replaced content**: Write fresh from code analysis. Verify every fact.
5. **For added content**: Follow greenfield per-document process.
6. **Present the full new doc for review** before replacing the existing file.

### Shared: README generation

The README is special — it's a hybrid document that serves as the project's
front door. It must include:

- **Title + one-line description** (what is this, what problem does it solve)
- **Badges** (optional: build status, coverage, version)
- **Installation** (copy-paste commands, minimal steps)
- **Quick start** (runnable example with sample output)
- **Key features** (bullet list of capabilities)
- **Usage examples** (code blocks with real commands/code)
- **Architecture overview** (brief summary or link to architecture doc)
- **Development** (how to build/test, or link to developer guide)
- **Contributing** (link to CONTRIBUTING.md)
- **License**

See the README template in [references/doc-templates.md](references/doc-templates.md).

**Brownfield README**: When editing an existing README, preserve its custom
sections (badges, custom headings, project-specific content). Update only the
sections identified in the delta plan.

### Shared: API reference generation

For libraries or services with a public API:

1. **Parse the code** to identify all public interfaces (exports, endpoints, commands)
2. **For each interface**, document: signature/URL, parameters, return type,
   authentication, example request, example response, error cases
3. **Mirror the code structure** in the doc structure (classes, modules, endpoints)
4. **If OpenAPI/Swagger exists**, use it as the canonical source
5. Follow Reference type rules: austere, factual, no instruction or explanation

**Brownfield API reference**: When updating, add new endpoints in the same
style as existing ones. Update changed signatures in place. Mark removed
endpoints as deprecated (do not delete unless user confirms — downstream
consumers may reference the docs).

### Shared: Architecture documentation

For multi-component systems, use the C4 model hierarchy:

1. **System Context**: What is this system? What external systems and users
   interact with it? (No technical detail — readable by non-technical people)
2. **Container diagram**: What are the major deployable units (apps, databases,
   queues)? What technologies? How do they communicate?
3. **Component diagram**: (Optional) Inside a container, what are the major
   modules and their responsibilities?

Label every element: Name + Technology + one-line responsibility.
Label every relationship: verb phrase describing what flows.

Record key design decisions as Architecture Decision Records (ADRs).

### Shared: Changelog generation

Follow the Keep a Changelog standard:

- Name the file `CHANGELOG.md`
- Maintain an "Unreleased" section at the top
- Use reverse chronological order
- Group changes under: Added, Changed, Deprecated, Removed, Fixed, Security
- Use ISO 8601 dates (YYYY-MM-DD)
- Never use raw git log as a changelog

**Brownfield changelog**: Append to existing changelog. Never rewrite history.
Add new entries at the top under Unreleased or a new version heading.

---

## Phase 4: Verify Documentation

After writing, verify every document against the code.

### Verification checklist (all workflows)

- [ ] **Accuracy**: Every factual claim matches the code
- [ ] **Commands work**: Every shell command runs successfully
- [ ] **Code examples compile/run**: Every code snippet is valid
- [ ] **Links resolve**: All internal links point to existing files/headings
- [ ] **Completeness**: All public interfaces are documented (for reference docs)
- [ ] **No fabrication**: No invented API endpoints, flags, or behaviors
- [ ] **Type purity**: Each doc stays within its Diataxis type (no mixing)
- [ ] **Consistent terminology**: Same term used for same concept everywhere
- [ ] **Config matches code**: Documented env vars, versions, and settings match actual code
- [ ] **No duplication**: Content lives in one place and is linked elsewhere

### Additional brownfield verification

- [ ] **Preserved content intact**: Domain knowledge, rationale, and custom
  sections from existing docs were not lost or mangled
- [ ] **Conventions maintained**: Heading style, terminology, code block
  formatting match the rest of the existing docs
- [ ] **No unplanned changes**: Only the sections identified in the delta plan
  were modified. No drive-by formatting fixes or unsolicited rewrites.
- [ ] **No content silently deleted**: Every removal was in the approved plan
- [ ] **Diff is minimal**: Changes are surgical, not a full rewrite
  (unless Targeted Rewrite was the chosen workflow)
- [ ] **Existing cross-references still work**: Changes didn't break links
  in OTHER docs that point to modified sections

### Verification method

For each document:

1. **Spot-check facts** against source code (grep for documented functions,
   endpoints, env vars — confirm they exist)
2. **Test commands** by reading them critically against the codebase
   (do the referenced files, scripts, and flags exist?)
3. **Validate examples** against actual code signatures and return types
4. **Check cross-references** (do linked files and headings exist?)
5. **Run doc linters** if the project has them (markdownlint, link checkers)
6. **(Brownfield) Compare against original** to confirm only planned changes
   were made and preserved content survived

### Common errors to catch

| Error | How to Detect |
|-------|---------------|
| Documented function doesn't exist | Grep for function name in source |
| Wrong parameter names/types | Compare doc to actual function signature |
| Outdated install command | Check package.json/pyproject.toml for current versions |
| Broken internal link | Check that target file/heading exists |
| Stale env var name | Grep for env var in source code |
| Example output doesn't match | Run or simulate the example mentally |
| Mixed Diataxis types | Review: does a tutorial explain? Does reference instruct? |
| Preserved content lost (brownfield) | Compare with original doc |
| Convention mismatch (brownfield) | Compare heading/code block style with existing docs |

---

## Phase 5: Maintenance Strategy

Documentation rots when it diverges from code. Embed maintenance practices.

### Docs-as-code principles

- **Version docs alongside code.** All docs live in the repo, not a wiki.
- **Update docs in the same commit as code changes.** Never defer.
- **Review docs in PRs.** Reviewers should reject PRs that change behavior
  without updating docs.
- **Delete dead docs.** Outdated documentation is worse than no documentation.
  Remove or mark deprecated.
- **Link, don't duplicate.** Point to canonical sources rather than copying.

### Automated checks (recommend to user)

Suggest these CI integrations where relevant:

| Check | Tool | Purpose |
|-------|------|---------|
| Markdown lint | markdownlint, remark-lint | Consistent formatting |
| Link checking | markdown-link-check | No broken links |
| Spell check | cspell, aspell | Catch typos |
| Doc generation | Sphinx, JSDoc, typedoc | Reference from source |
| Example testing | doctest, mdx-test | Examples actually work |
| API sync | swagger-diff, openapi-diff | API docs match spec |

---

## Failure Recovery

```
Documentation issue
  |
  +-> Can't determine what code does
  |     +-> Read more source files
  |     +-> Check tests for behavioral clues
  |     +-> Ask user for clarification
  |
  +-> Conflicting information (code vs. config vs. comments)
  |     +-> Code is authoritative (it's what actually runs)
  |     +-> Note the conflict; update incorrect sources
  |
  +-> Unsure if feature is public or internal
  |     +-> Check exports, access modifiers, public API markers
  |     +-> When in doubt, document it (users may depend on it)
  |     +-> Ask user if unclear
  |
  +-> Existing docs are extensive but wrong (brownfield)
  |     +-> Fix accuracy first, preserve structure
  |     +-> Never rewrite from scratch unless user requests it
  |     +-> Track changes for user review
  |
  +-> Existing doc has valuable content mixed with stale content (brownfield)
  |     +-> Separate: identify each section as preserve/replace/add/remove
  |     +-> Fix the stale parts, keep the valuable parts
  |     +-> When uncertain, preserve and flag for user review
  |
  +-> Conventions in existing docs conflict with best practices (brownfield)
        +-> Follow existing conventions (consistency > correctness)
        +-> Only change conventions if user explicitly requests it
        +-> If existing convention causes real problems (e.g., broken rendering),
            fix that specific issue and note the change
```

**Never fabricate.** If you cannot determine a fact from the code, say so
and ask the user. Do not guess API behaviors, default values, or configuration.

**Never write about code you haven't read.** Always read the source before
documenting it.

**Never silently rewrite.** In brownfield workflows, if you find yourself
wanting to restructure or rephrase content not in the plan, stop. That impulse
means the scope needs revisiting, not expanding.

---

## Reference Files

- **Document templates**: See [references/doc-templates.md](references/doc-templates.md)
  for README, API Reference, Architecture, Developer Guide, Contributing,
  Changelog, Getting Started, ADR, and Testing templates
- **Diataxis guide**: See [references/diataxis-guide.md](references/diataxis-guide.md)
  for the four documentation types with do/don't rules, voice, and structure
- **Repository analysis**: See [references/repo-analysis.md](references/repo-analysis.md)
  for the systematic codebase analysis method
- **Drift detection**: See [references/drift-detection.md](references/drift-detection.md)
  for the section-by-section method to identify doc/code divergence
- **Writing rules**: See [references/writing-rules.md](references/writing-rules.md)
  for style guide, formatting rules, and quality checklist
