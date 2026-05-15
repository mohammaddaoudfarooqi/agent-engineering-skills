---
name: github-actions
description: >
  Systematic GitHub Actions workflow authoring skill for AI coding agents.
  Analyzes repositories to determine project type, language ecosystem, and
  deployment targets, then generates production-grade CI/CD workflows with
  proper security hardening, caching, and optimization.
  Handles greenfield projects (no workflows exist), brownfield updates
  (modify, optimize, secure existing workflows), and workflow audits with
  workflow-specific guidance for each.
  Use when the user requests GitHub Actions workflows: CI pipelines, CD
  deployments, release automation, scheduled jobs, or any .github/workflows
  YAML authoring. Also use when existing workflows need auditing, optimizing,
  securing, or restructuring.
  Triggers on phrases like "set up CI", "add CI/CD", "GitHub Actions
  workflow", "release automation", "deploy on tag", "publish to npm/PyPI",
  "schedule a job", "cron workflow", "matrix build", "workflow.yml",
  "actions/checkout", "permissions", "harden this pipeline", "pin actions
  to SHA", "OIDC", "least privilege", "supply-chain", "audit my workflows",
  "speed up CI", or "cache dependencies". Triggers when creating or editing
  files under `.github/workflows/`, `action.yml`/`action.yaml` (composite or
  Docker actions), or `.github/dependabot.yml`. Triggers when the user
  mentions migrating from GitLab CI, CircleCI, Travis, Jenkins, Drone, or
  Buildkite to GitHub Actions.
  Do NOT use for non-GitHub CI systems (GitLab CI, CircleCI, Jenkins) unless
  the user is migrating TO GitHub Actions. Do NOT use for general bash
  scripting, Makefiles, or local-only build configuration.
---

# GitHub Actions Workflow Authoring

Generate production-grade GitHub Actions workflows from systematic repository
analysis. Every workflow is security-hardened, performance-optimized, and
verified against the codebase.

## Workflow Router

Determine the workflow type before starting. This drives which phases apply,
which templates to use, and whether to write from scratch or edit existing
workflows.

```
Do any GitHub Actions workflows already exist?
  (.github/workflows/*.yml or .github/workflows/*.yaml)
  |
  NO --> GREENFIELD (no workflows exist)
  |       Phases: Discover -> Plan -> Write -> Verify
  |
  YES --> What type of change?
           |
           +-> New feature/service needs a pipeline
           |     --> WORKFLOW ENHANCEMENT (brownfield)
           |     Phases: Discover -> Audit Existing -> Plan (delta) -> Write -> Verify
           |
           +-> Existing workflow needs fixing/updating
           |     --> WORKFLOW UPDATE (brownfield)
           |     Phases: Discover -> Audit Existing -> Plan (delta) -> Write (edit) -> Verify
           |
           +-> Workflows exist but have security/performance issues
           |     --> WORKFLOW AUDIT (brownfield)
           |     Phases: Discover -> Audit Existing -> Plan (assessment) -> Write (edit) -> Verify
           |
           +-> User wants a specific workflow rewritten
           |     --> TARGETED REWRITE (brownfield)
           |     Phases: Discover -> Audit Existing (single workflow) -> Write (replace) -> Verify
           |
           +-> User wants a specific new workflow type
           |     --> TARGETED NEW WORKFLOW
           |     Phases: Discover -> Write (specific) -> Verify
           |
           +-> Migrating from another CI system
                 --> CI MIGRATION
                 Phases: Discover -> Audit Source CI -> Plan -> Write -> Verify
```

**Signal detection:**

| Signal in request | Likely type |
|-------------------|-------------|
| "Add CI", "add GitHub Actions", "set up CI/CD" (no workflows exist) | Greenfield |
| "Add CI", "add pipeline" (workflows already exist) | Workflow Enhancement |
| "Fix the CI", "workflow is broken", "build fails" | Workflow Update |
| "Workflow is slow", "optimize CI", "reduce build time" | Workflow Audit |
| "Security audit", "harden workflows", "pin actions" | Workflow Audit |
| "Rewrite the CI", "redo the pipeline" | Targeted Rewrite |
| "Add deploy workflow", "add release pipeline" | Targeted New Workflow |
| "Add Docker build", "publish to npm/PyPI" | Targeted New Workflow |
| "Migrate from Jenkins/CircleCI/GitLab" | CI Migration |
| "Add a cron job", "scheduled workflow" | Targeted New Workflow |

---

## Phase 0: Discover Repository

**Always run this phase.** Before writing any workflow, build a complete
picture of the project. See
[references/repo-discovery.md](references/repo-discovery.md) for the full
discovery method.

### What to discover

1. **Language and ecosystem**: Identify from manifest files (package.json,
   pyproject.toml, go.mod, Cargo.toml, pom.xml, build.gradle, etc.)
2. **Package manager**: npm/yarn/pnpm, pip/poetry/uv, go mod, cargo, maven/gradle
3. **Framework**: Detect from dependencies and config files (Next.js, Django,
   Spring Boot, Rails, etc.)
4. **Build system**: Webpack, Vite, Turborepo, Nx, Gradle, Maven, Make, etc.
5. **Test setup**: Test framework, runner command, test config files
6. **Lint/format tools**: ESLint, Prettier, Ruff, Black, golangci-lint, etc.
7. **Containerization**: Dockerfile presence, docker-compose, Kubernetes manifests
8. **Infrastructure as Code**: Terraform, Pulumi, CloudFormation, CDK
9. **Monorepo signals**: Multiple package.json files, workspace configs,
   Nx/Turborepo/Lerna config
10. **Deployment targets**: Cloud provider configs, serverless.yml, Kubernetes,
    Vercel/Netlify config
11. **Existing workflows**: Current .github/workflows/*.yml files
12. **Branch strategy**: Default branch (main/master), protected branches,
    release branch patterns
13. **Release patterns**: Tags, GitHub Releases, changelogs, version files

### How to explore

- Read README.md and CLAUDE.md first (if they exist)
- Read manifest files (package.json, pyproject.toml, go.mod, etc.)
- Scan `.github/` directory for existing workflows, actions, dependabot config
- Check for Dockerfiles, docker-compose files, Kubernetes manifests
- Look for IaC files (*.tf, serverless.yml, template.yaml)
- Check for monorepo indicators (workspaces, nx.json, turbo.json, lerna.json)
- Read 2-3 key source files to understand project patterns
- Check existing CI config from other systems (.circleci/, Jenkinsfile,
  .gitlab-ci.yml, .travis.yml)

### Repository discovery algorithm

Follow the deterministic detection sequence in
[references/repo-discovery.md](references/repo-discovery.md):

```
1. Scan root for manifest files -> identify language(s)
2. Read manifest -> identify package manager, deps, scripts
3. Check for test/lint config files -> identify quality tools
4. Check for Dockerfile -> plan container build
5. Check for IaC files -> plan infrastructure pipeline
6. Check for monorepo config -> plan multi-package strategy
7. Check for deployment config -> plan CD pipeline
8. Check for existing CI -> plan migration or enhancement
```

### Output: Repository Profile

Produce a mental model (do not write a file unless requested):

```
Languages: [detected languages and versions]
Package managers: [npm/yarn/pnpm, pip/poetry/uv, etc.]
Framework: [framework and version]
Build tool: [build system]
Test framework: [test runner and command]
Lint/format: [tools and commands]
Container: [Dockerfile location, registry target]
IaC: [Terraform/Pulumi/etc. and directory]
Monorepo: [yes/no, tool, packages]
Deploy target: [cloud provider, platform, method]
Existing CI: [workflow files and their purpose]
Branch strategy: [default branch, release pattern]
```

This profile drives every subsequent phase.

---

## Phase 1: Audit Existing Workflows (Brownfield Only)

**Skip this phase for greenfield projects.**

Before planning or writing anything, understand what workflows already exist
and assess their state.

### Read every existing workflow

For each `.github/workflows/*.yml` file found in Phase 0:

1. **Read it fully.** Parse the YAML structure: triggers, permissions, jobs,
   steps, secrets usage, caching, matrix strategies.
2. **Classify its purpose**: CI (test/lint/build), CD (deploy), Release
   (publish/tag), Maintenance (dependabot, stale issues), Scheduled (cron jobs).
3. **Note its conventions**: naming style, job naming, step naming, comment
   style, environment variable patterns.
4. **Assess security posture**: permissions declared? Actions pinned by SHA?
   Secrets handling? Fork protections? OIDC usage?
5. **Assess performance**: caching used? Unnecessary steps? Job parallelism?
   Matrix strategy? Concurrency groups?

### Existing workflow inventory

Build a mental inventory:

```
Existing Workflow Inventory
  [file path] | [purpose] | [triggers] | [security: good/fair/poor] | [performance: good/fair/poor]
  ci.yml       | CI        | push, PR   | fair (unpinned actions)    | fair (no caching)
  deploy.yml   | CD        | push main  | good (OIDC, pinned)        | good
  ...
```

### What to preserve in brownfield

| Content Type | Example | Preserve? |
|-------------|---------|-----------|
| Working trigger logic | Path filters, branch filters | Always |
| Environment-specific secrets | AWS credentials, deploy tokens | Always |
| Custom scripts/steps | Project-specific build logic | Always |
| Concurrency groups | Existing cancel-in-progress logic | Always |
| Job dependencies | Established `needs:` chains | Default yes |
| Unpinned actions | `uses: actions/checkout@v4` | Replace with SHA pin |
| Overly broad permissions | `permissions: write-all` | Replace with least privilege |
| Duplicated logic | Same steps in multiple workflows | Refactor to reusable |
| Dead workflows | Workflows for removed features | Remove |

**Rule: Working workflows are valuable.** A workflow that runs correctly is
better than a theoretically perfect one that breaks. Preserve working logic
and improve incrementally.

### Brownfield conventions contract

Extract the workflow conventions from existing files and follow them:

- **File naming**: kebab-case? snake_case? What prefix/suffix patterns?
- **Job naming**: How are jobs named? Descriptive or terse?
- **Step naming**: Named steps or anonymous? Style of names?
- **Comment style**: Are inline comments used? YAML comments?
- **Secret naming**: Convention for secret names (SCREAMING_SNAKE, etc.)?
- **Environment patterns**: How environments are used (if at all)?

**Match existing conventions.** Do NOT impose new conventions on a brownfield
project unless the user explicitly requests a style change.

---

## Phase 2: Plan Workflows

### Greenfield planning

Derive a complete CI/CD plan from the repository discovery.

#### Pipeline architecture

Every project needs at minimum:

1. **CI workflow** (pull request validation):
   - Lint/format check
   - Type check (if applicable)
   - Unit tests
   - Build verification

2. **CI workflow** (main branch):
   - Everything in PR CI, plus:
   - Integration tests (if applicable)
   - Artifact building
   - Trigger CD (if applicable)

3. **CD workflow** (if deployment target exists):
   - Build artifacts/containers
   - Deploy to environments (staging, production)
   - Environment protection rules

4. **Release workflow** (if library or publishable):
   - Version bumping
   - Package publishing (npm, PyPI, Maven, crates.io)
   - GitHub Release creation

5. **Maintenance workflows** (recommended):
   - Dependabot configuration
   - Stale issue/PR management
   - Scheduled security scans

#### Workflow plan algorithm

Based on the repository profile from Phase 0:

```
FOR EACH detected language:
  -> Add CI job: setup + install + lint + test + build
  -> Select template from references/workflow-templates.md

IF Dockerfile exists:
  -> Add Docker build/push job
  -> Select registry (GHCR by default)

IF IaC files exist:
  -> Add infrastructure validation job (plan on PR, apply on merge)

IF monorepo detected:
  -> Add change detection job
  -> Make CI jobs conditional on changed paths
  -> See references/pipeline-patterns.md for monorepo patterns

IF deployment target detected:
  -> Add CD workflow with environment protection
  -> See references/pipeline-patterns.md for deployment patterns

IF library (not service):
  -> Add release/publish workflow triggered by tags

ALWAYS:
  -> Apply security hardening (references/security-hardening.md)
  -> Add concurrency groups
  -> Add caching for dependencies
  -> Set explicit permissions (least privilege)
```

#### Plan output

```
Workflow Plan
- .github/workflows/ci.yml — CI pipeline (lint, test, build) on PR + push main
- .github/workflows/deploy.yml — Deploy to staging/prod with environment protection
- .github/workflows/release.yml — Publish package on tag push
- .github/dependabot.yml — Dependency update automation
```

Present the plan to the user for approval before writing.

### Brownfield planning

**Use the workflow inventory from Phase 1.** Do NOT plan from scratch. Plan
only what changes.

#### Workflow Enhancement: Delta plan

When adding a new workflow to an existing set:

1. Identify which EXISTING workflows might be affected (shared secrets,
   concurrency groups, deployment targets)
2. Identify the NEW workflow to create
3. For existing workflows, specify ONLY what changes (if anything)

```
Workflow Delta Plan
- .github/workflows/ci.yml — KEEP (no changes needed)
- .github/workflows/deploy.yml — MODIFY: add staging environment before prod
- .github/workflows/docker.yml — CREATE: Docker build + push to GHCR
```

#### Workflow Audit: Assessment plan

When auditing existing workflows for security/performance:

1. List all issues found in Phase 1 with severity
2. Propose fixes grouped by category (Security > Performance > Maintenance)
3. For each workflow: KEEP (as-is), EDIT (fix issues), REWRITE (replace),
   DELETE (remove dead workflow)
4. Present to user for approval

```
Audit Findings & Plan
  ci.yml:
    - [Critical] Actions not pinned to SHA → EDIT: pin all actions
    - [High] No permissions declared → EDIT: add least-privilege permissions
    - [Medium] No dependency caching → EDIT: add cache configuration
  deploy.yml:
    - [High] Uses long-lived AWS keys → EDIT: migrate to OIDC
    - [Low] No concurrency group → EDIT: add concurrency
  old-ci.yml:
    - [Low] Duplicate of ci.yml → DELETE
```

#### Targeted Rewrite: Single-workflow plan

When the user asks to rewrite a specific workflow:

1. Read the existing workflow fully (done in Phase 1)
2. Identify what to PRESERVE (working trigger logic, secrets, custom scripts)
3. Identify what to REPLACE (insecure patterns, poor performance, outdated actions)
4. Identify what to ADD (missing security, caching, concurrency)
5. Present the plan

```
Rewrite Plan: ci.yml
  PRESERVE: Trigger logic (push/PR), test matrix (Node 18/20), deploy job dependency
  REPLACE:  Unpinned actions (pin to SHA), npm install (use npm ci + caching),
            missing permissions (add least-privilege)
  ADD:      Concurrency group, artifact upload for test reports, type checking step
```

**Get user approval before rewriting.**

#### CI Migration plan

When migrating from another CI system:

1. Read the source CI config (.circleci/config.yml, Jenkinsfile, .gitlab-ci.yml)
2. Map each job/stage to a GitHub Actions equivalent
3. Identify feature gaps (some CI features don't map 1:1)
4. Plan the migration as a set of new workflows

```
Migration Plan: CircleCI → GitHub Actions
  CircleCI build job → .github/workflows/ci.yml (lint + test + build)
  CircleCI deploy job → .github/workflows/deploy.yml (with environment protection)
  CircleCI scheduled scan → .github/workflows/security.yml (cron schedule)
  Gap: CircleCI orbs used → Replace with equivalent GitHub Actions
```

Present the plan to the user for approval before writing.

---

## Phase 3: Write Workflows

### Greenfield: Write from scratch

Write each planned workflow using the appropriate template and security rules.
See [references/workflow-templates.md](references/workflow-templates.md) for
ecosystem-specific templates.
See [references/security-hardening.md](references/security-hardening.md) for
security rules.
See [references/pipeline-patterns.md](references/pipeline-patterns.md) for
CI/CD patterns.

#### Writing sequence

Write workflows in this order:

1. **CI workflow** first (validates that the build/test pipeline works)
2. **CD workflow** (deployment depends on CI)
3. **Release workflow** (publishing depends on build)
4. **Maintenance workflows** (dependabot, security scans)

#### Per-workflow process (greenfield)

For each workflow:

1. **Select template**: Choose from references/workflow-templates.md based
   on language/ecosystem detected in Phase 0.
2. **Customize for project**: Replace template placeholders with actual
   commands, paths, versions from the repository profile.
3. **Apply security hardening**: Follow every rule in
   references/security-hardening.md:
   - Set explicit `permissions:` (least privilege)
   - Pin all actions to full SHA
   - Use OIDC for cloud auth where possible
   - Protect secrets from fork exposure
   - Guard against script injection
4. **Add performance optimization**:
   - Add dependency caching (use setup action's built-in cache where available)
   - Add concurrency groups to cancel stale runs
   - Use matrix strategy for multi-version testing
   - Add path filters to avoid unnecessary runs
5. **Verify YAML syntax**: Ensure valid YAML, proper indentation, correct
   GitHub Actions syntax.
6. **Add inline comments**: Explain non-obvious choices (why a specific
   SHA, why a particular cache key, why a permission).

### Brownfield: Edit existing workflows

**Read before writing.** For every workflow you modify, read it fully first
(already done in Phase 1). Never modify a workflow you haven't read.

#### Per-workflow process (brownfield)

For each workflow being modified:

1. **Re-read the workflow.** Confirm your understanding from Phase 1.
2. **Apply only planned changes.** Follow the delta plan from Phase 2.
   Do NOT restructure, restyle, or "improve" sections outside scope.
3. **Match existing conventions.** Use the same naming style, comment style,
   and patterns discovered in Phase 1.
4. **Preserve working logic.** Trigger conditions, environment variables,
   secrets references, and custom scripts survive unless explicitly marked
   for change.
5. **Verify every edit.** Each changed action version, command, or path must
   be verified against the codebase.
6. **Minimize diff.** Change only what needs changing. Smaller diffs are
   easier to review.

#### Brownfield-specific rules

- **Edit, don't rewrite.** Unless the user requests a rewrite, modify the
  existing workflow surgically. Fix the specific issues; leave working
  steps alone.
- **Match existing style.** If existing workflows use `name: Build and Test`
  for jobs, continue that pattern. Do NOT switch to `name: build-and-test`
  because you prefer it.
- **Preserve structure.** Keep the existing job ordering unless it's part of
  the planned changes.
- **Never silently remove steps.** If a step needs removal, note it in your
  plan and confirm with the user. A step that looks unnecessary may have
  a non-obvious purpose.
- **Refactor only what you're asked to change.** Do NOT improve adjacent
  workflows, fix unrelated issues, or add features not in the plan.

#### Targeted Rewrite process

When rewriting a specific workflow (user explicitly requested):

1. **Create from the approved plan.** Write the new version using the
   PRESERVE/REPLACE/ADD breakdown from Phase 2.
2. **Start from the template** in references/workflow-templates.md, then
   transplant preserved logic into the new structure.
3. **For preserved content**: Keep trigger logic, secrets, and custom
   scripts intact.
4. **For replaced content**: Write fresh from repository analysis.
5. **For added content**: Follow greenfield per-workflow process.
6. **Present the full new workflow for review** before replacing.

### Shared: Workflow structure conventions

Every workflow file must follow this structure:

```yaml
# .github/workflows/<name>.yml
name: <Descriptive Name>

on:
  # Triggers section — always explicit, never use defaults
  push:
    branches: [main]
  pull_request:
    branches: [main]

# Workflow-level permissions — least privilege
permissions:
  contents: read

# Concurrency — prevent queue buildup
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # Set to false for deploy/release/IaC workflows

jobs:
  <job-name>:
    name: <Human-Readable Job Name>
    runs-on: ubuntu-latest
    # Job-level permissions if different from workflow
    steps:
      - name: Checkout code
        uses: actions/checkout@<full-sha>  # Always pinned
      # ... remaining steps
```

### Shared: Action version pinning

**Always pin actions to full commit SHA.** Never use tags or branches in
production workflows.

```yaml
# WRONG — tag can be moved to point at malicious code
uses: actions/checkout@v4

# CORRECT — immutable SHA reference
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
```

Include the tag version as an inline comment for readability.

See [references/security-hardening.md](references/security-hardening.md)
for the full SHA pinning guide and current action SHAs.

### Shared: Caching strategy

For each language ecosystem, use the appropriate caching approach:

| Ecosystem | Cache Method | Cache Key |
|-----------|-------------|-----------|
| Node.js | `actions/setup-node` with `cache: 'npm'` | Lockfile hash (automatic) |
| Python | `actions/setup-python` with `cache: 'pip'` | requirements.txt hash (automatic) |
| Go | `actions/setup-go` with `cache: true` | go.sum hash (automatic) |
| Rust | `Swatinem/rust-cache` | Cargo.lock + rustc version (automatic) |
| Java/Gradle | `actions/setup-java` with `cache: 'gradle'` | Gradle files hash (automatic) |
| Java/Maven | `actions/setup-java` with `cache: 'maven'` | pom.xml hash (automatic) |
| .NET | `actions/setup-dotnet` (NuGet cache via `actions/cache`) | `packages.lock.json` hash |
| Ruby | `ruby/setup-ruby` with `bundler-cache: true` | Gemfile.lock hash (automatic) |
| Generic | `actions/cache` | Custom key based on lockfile hash |

**Prefer setup action caching over raw `actions/cache` where available.**
Setup actions handle cache paths and keys automatically.

### Shared: Dependabot configuration

Always recommend adding `.github/dependabot.yml` for automated dependency
and action version updates:

```yaml
version: 2
updates:
  # Keep GitHub Actions up to date
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    # Group minor/patch updates to reduce PR noise
    groups:
      actions:
        patterns:
          - "*"

  # Keep project dependencies up to date
  - package-ecosystem: "<ecosystem>"  # npm, pip, gomod, cargo, etc.
    directory: "/"
    schedule:
      interval: "weekly"
```

---

## Phase 4: Verify Workflows

After writing, verify every workflow against the codebase and GitHub Actions
best practices.

### Verification checklist (all workflows)

- [ ] **Valid YAML**: Proper indentation, no syntax errors
- [ ] **Triggers correct**: Events match intended behavior (push, PR, tag, schedule)
- [ ] **Permissions declared**: Explicit `permissions:` at workflow or job level
- [ ] **Least privilege**: No unnecessary write permissions
- [ ] **Actions pinned**: All `uses:` reference full SHA (not tags or branches)
- [ ] **Commands exist**: Every `run:` command references valid scripts, tools, and paths
- [ ] **Secrets referenced correctly**: All `${{ secrets.* }}` reference expected secret names
- [ ] **Caching configured**: Dependencies are cached where possible
- [ ] **Concurrency set**: Workflow has concurrency group to prevent queue buildup
- [ ] **Branch filters correct**: Push/PR triggers reference correct branch names
- [ ] **Path filters appropriate**: Path-based triggers match actual project structure
- [ ] **Matrix strategy sensible**: Tested versions are current and supported
- [ ] **Artifact handling correct**: Upload/download artifact names are consistent
- [ ] **Environment protection**: Deploy jobs reference proper environments
- [ ] **No script injection**: Untrusted input is never used in `run:` directly
- [ ] **No hardcoded secrets**: No credentials in workflow files

### Additional brownfield verification

- [ ] **Preserved logic intact**: Trigger conditions, secrets, and custom
  scripts from existing workflows were not lost
- [ ] **Conventions maintained**: Naming, commenting, and structure style
  match the rest of the existing workflows
- [ ] **No unplanned changes**: Only the sections identified in the delta
  plan were modified
- [ ] **No steps silently removed**: Every removal was in the approved plan
- [ ] **Diff is minimal**: Changes are surgical, not a full rewrite
  (unless Targeted Rewrite was the chosen workflow type)

### Verification method

For each workflow:

1. **Validate YAML syntax** by reading it carefully for indentation and
   structure issues
2. **Check all referenced paths** exist in the repository (scripts, config
   files, directories)
3. **Verify action references** are valid (action exists, SHA is real)
4. **Verify commands** match the project's actual tools and scripts (check
   package.json scripts, Makefile targets, etc.)
5. **Check secret names** match what the project expects (documented in
   README or existing workflows)
6. **Simulate trigger scenarios** mentally: what happens on PR, push to
   main, tag push, schedule?
7. **(Brownfield) Compare against original** to confirm only planned
   changes were made

### Common errors to catch

| Error | How to Detect |
|-------|---------------|
| Unpinned action | `uses:` line has @vN instead of @sha |
| Missing permissions | No `permissions:` block at workflow or job level |
| Wrong branch name | Triggers reference `master` when default is `main` |
| Dead path filter | `paths:` references directory that doesn't exist |
| Wrong test command | `run: npm test` when project uses `pnpm test` |
| Missing cache | Setup action used without `cache:` parameter |
| Script injection | `${{ github.event.* }}` used directly in `run:` block |
| Overly broad trigger | `on: push` without branch filter runs on every push |
| Missing concurrency | No `concurrency:` block means queue buildup on rapid pushes |
| Hardcoded version | Python/Node/Go version hardcoded instead of from matrix or config |

---

## Phase 5: Maintenance Strategy

Workflows rot when they fall behind ecosystem changes. Embed maintenance
practices.

### Workflow-as-code principles

- **Version workflows alongside code.** All workflows live in `.github/workflows/`.
- **Review workflow changes in PRs.** Use CODEOWNERS on `.github/` to require
  review for workflow changes.
- **Keep actions updated.** Use Dependabot for GitHub Actions ecosystem updates.
- **Delete dead workflows.** Unused workflows create confusion and security risk.
- **Document non-obvious choices.** Inline comments explain WHY, not WHAT.

### Automated maintenance (recommend to user)

| Tool | Purpose |
|------|---------|
| Dependabot (github-actions ecosystem) | Auto-update action versions |
| actionlint | Lint workflow files for syntax and best practice issues |
| CODEOWNERS on `.github/` | Require review for workflow changes |
| OpenSSF Scorecard | Automated security assessment of CI/CD practices |
| GitHub branch protection | Require status checks to pass before merge |

### Recommend to user

Suggest adding these where relevant:

1. **Branch protection rules**: Require CI to pass before merge
2. **CODEOWNERS**: Protect `.github/` directory
3. **Dependabot**: Keep actions and dependencies updated
4. **actionlint**: Add as a CI step or pre-commit hook
5. **Environment protection rules**: Require approval for production deploys

---

## Failure Recovery

```
Workflow issue
  |
  +-> Can't determine project type
  |     +-> Read more source files and config
  |     +-> Check build scripts in package.json/Makefile
  |     +-> Ask user for clarification
  |
  +-> Conflicting CI config (multiple systems)
  |     +-> Ask user which system is canonical
  |     +-> Treat GitHub Actions as target; other as reference
  |
  +-> Unsure which actions to use
  |     +-> Prefer official actions (actions/*, github/*)
  |     +-> For ecosystem tools, use most-adopted community action
  |     +-> Check references/workflow-templates.md for recommendations
  |     +-> When in doubt, use raw `run:` commands instead of actions
  |
  +-> Existing workflows are complex but fragile (brownfield)
  |     +-> Preserve working logic first
  |     +-> Never rewrite from scratch unless user requests it
  |     +-> Fix one issue at a time, verify between changes
  |
  +-> Monorepo with unclear service boundaries
  |     +-> Look for workspace configs (package.json workspaces, nx.json, turbo.json)
  |     +-> Check for independent package.json/go.mod files in subdirectories
  |     +-> Ask user to confirm service/package boundaries
  |
  +-> Migration from another CI system
  |     +-> Map concepts: jobs->jobs, stages->jobs with needs, orbs->reusable workflows
  |     +-> Not everything maps 1:1 — document gaps
  |     +-> Prioritize: get CI working first, optimize later
  |
  +-> Conventions in existing workflows conflict with best practices (brownfield)
        +-> Follow existing conventions (consistency > correctness)
        +-> Only change conventions if user explicitly requests it
        +-> Exception: always fix critical security issues (unpinned actions,
            missing permissions, script injection) regardless of convention
```

**Never fabricate action names.** If you're unsure an action exists, use a
raw `run:` command instead. Verify action names against known actions.

**Never hardcode secrets.** Always reference `${{ secrets.* }}`.

**Never write workflows you haven't verified.** Check every command, path,
and action reference against the actual repository.

---

## Anti-Patterns

### Security anti-patterns
- **Unpinned actions**: Using `@v4` instead of `@sha` — supply chain attack vector
- **Missing permissions**: No `permissions:` block — defaults to overly broad access
- **Script injection**: Using `${{ github.event.* }}` directly in `run:` blocks
- **Long-lived credentials**: Storing AWS/GCP keys as secrets instead of using OIDC
- **Fork secret exposure**: Running `pull_request_target` with checkout of PR code
- **Overly broad `GITHUB_TOKEN`**: Granting write permissions when read suffices

### Performance anti-patterns
- **No caching**: Reinstalling dependencies from scratch every run
- **No concurrency groups**: Queue buildup on rapid pushes
- **Monolithic workflows**: One huge workflow instead of parallel jobs
- **Unnecessary triggers**: Running full CI on docs-only changes
- **Missing path filters**: Every push triggers every workflow regardless of changes
- **Excessive matrix**: Testing on 12 OS/version combos when 4 would suffice

### Architecture anti-patterns
- **Workflow sprawl**: 20 small workflow files with overlapping triggers
- **Duplicate logic**: Same steps copy-pasted across workflows (use reusable workflows)
- **Missing `needs:`**: Jobs that should be sequential running in parallel
- **No environment protection**: Deploy to production without approval gates
- **No Dependabot**: Actions and dependencies never updated

### Process anti-patterns
- **Treating first draft as final**: Always verify against the codebase
- **Ignoring existing workflows**: Not reading current workflows before modifying
- **Over-engineering**: Adding complex matrix builds for a single-language project
- **Premature optimization**: Adding self-hosted runners before measuring need

---

## Reference Files

- **Repository discovery**: See [references/repo-discovery.md](references/repo-discovery.md)
  for deterministic repository analysis: language detection, ecosystem
  identification, deployment target detection, and monorepo discovery
- **Workflow templates**: See [references/workflow-templates.md](references/workflow-templates.md)
  for complete YAML templates per ecosystem: Node.js, Python, Go, Rust,
  Java, Docker, Terraform, and monorepo configurations
- **Security hardening**: See [references/security-hardening.md](references/security-hardening.md)
  for permissions model, action pinning with current SHAs, OIDC configuration,
  secret management, fork protection, and script injection prevention
- **Pipeline patterns**: See [references/pipeline-patterns.md](references/pipeline-patterns.md)
  for CI/CD patterns: caching strategies, matrix builds, concurrency groups,
  reusable workflows, monorepo change detection, deployment strategies,
  release automation, and scheduled pipelines
