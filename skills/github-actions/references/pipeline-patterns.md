# Pipeline Patterns

CI/CD patterns, strategies, and design decisions for GitHub Actions workflows.

## Table of Contents
- [Caching Strategies](#caching-strategies)
- [Matrix Builds](#matrix-builds)
- [Concurrency Groups](#concurrency-groups)
- [Job Dependencies and Parallelism](#job-dependencies-and-parallelism)
- [Artifacts vs Caches](#artifacts-vs-caches)
- [Reusable Workflows](#reusable-workflows)
- [Composite Actions](#composite-actions)
- [Monorepo Change Detection](#monorepo-change-detection)
- [Deployment Strategies](#deployment-strategies)
- [Release Automation](#release-automation)
- [Scheduled Pipelines](#scheduled-pipelines)
- [Conditional Execution](#conditional-execution)
- [Service Containers](#service-containers)
- [Cost Optimization](#cost-optimization)

---

## Caching Strategies

### Setup action caching (preferred)

Most setup actions have built-in caching. Prefer this over raw `actions/cache`.

```yaml
# Node.js — automatic lockfile-based caching
- uses: actions/setup-node@<SHA>  # v4
  with:
    node-version: 22
    cache: 'npm'  # or 'pnpm' or 'yarn'

# Python — automatic requirements-based caching
- uses: actions/setup-python@<SHA>  # v5
  with:
    python-version: '3.12'
    cache: 'pip'  # or 'pipenv' or 'poetry'

# Go — automatic go.sum-based caching (enabled by default)
- uses: actions/setup-go@<SHA>  # v5
  with:
    go-version: '1.23'

# Java — automatic pom/gradle-based caching
- uses: actions/setup-java@<SHA>  # v4
  with:
    distribution: temurin
    java-version: 21
    cache: gradle  # or 'maven'
```

### Generic caching with actions/cache

For cases where setup actions don't cover your needs:

```yaml
- name: Cache dependencies
  uses: actions/cache@<SHA>  # v4
  with:
    path: |
      ~/.cache/custom-tool
      node_modules
    key: ${{ runner.os }}-custom-${{ hashFiles('**/lockfile') }}
    restore-keys: |
      ${{ runner.os }}-custom-
```

### Cache key design

| Pattern | Use Case |
|---------|----------|
| `${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}` | Exact lockfile match |
| `${{ runner.os }}-npm-` | Prefix fallback (partial restore) |
| `${{ runner.os }}-build-${{ github.sha }}` | Per-commit cache (build artifacts) |
| `${{ runner.os }}-${{ matrix.node-version }}-npm-${{ hashFiles('...') }}` | Per-matrix-entry cache |

### Docker layer caching

```yaml
- uses: docker/build-push-action@<SHA>  # v6
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

The `type=gha` uses GitHub Actions cache backend for Docker layers.

---

## Matrix Builds

### Basic matrix

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
  fail-fast: false  # Don't cancel other jobs if one fails
```

### Multi-dimensional matrix

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    node-version: [20, 22]
  fail-fast: false
```

### Matrix with includes/excludes

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    python-version: ['3.11', '3.12', '3.13']
    exclude:
      - os: macos-latest
        python-version: '3.11'  # Don't test old Python on macOS
    include:
      - os: ubuntu-latest
        python-version: '3.13'
        experimental: true  # Extra flag for specific combo
```

### Dynamic matrix

Generate matrix values from a previous job:

```yaml
jobs:
  detect:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          # Generate matrix from changed packages, config files, etc.
          echo "matrix={\"package\":[\"api\",\"web\"]}" >> $GITHUB_OUTPUT

  build:
    needs: detect
    strategy:
      matrix: ${{ fromJSON(needs.detect.outputs.matrix) }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building ${{ matrix.package }}"
```

### When to use matrix builds

| Scenario | Matrix? | Strategy |
|----------|---------|----------|
| Library supporting multiple versions | Yes | Version matrix (2-3 versions) |
| Cross-platform binary | Yes | OS matrix |
| Monorepo with similar packages | Maybe | Dynamic matrix from change detection |
| Single application, single version | No | Single job |
| Testing against multiple databases | Yes | Include matrix with service container config |

### Matrix best practices

- **`fail-fast: false`** — usually want to see all failures, not just first
- **Limit dimensions** — 2-3 values per dimension; don't create a 12+ job matrix
- **Test on ubuntu-latest by default** — add other OSes only if needed
- **Use LTS versions** — don't test every minor version; test current + previous LTS

---

## Concurrency Groups

### Prevent queue buildup

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

This cancels previous runs of the same workflow on the same branch when
a new push arrives. Essential for busy repositories.

### Concurrency patterns

| Pattern | Use Case |
|---------|----------|
| `${{ github.workflow }}-${{ github.ref }}` | Per-workflow, per-branch (most common) |
| `deploy-${{ github.ref }}` | Shared across deploy workflows on same branch |
| `deploy-production` | Global lock for production deploys |
| `${{ github.workflow }}-${{ github.event.pull_request.number }}` | Per-PR |

### When NOT to cancel in progress

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: false  # Don't cancel active deployments
```

Never cancel in-progress deployments or infrastructure changes. Use
`cancel-in-progress: false` for:
- Production deployments
- Terraform apply
- Database migrations
- Release publishing

---

## Job Dependencies and Parallelism

### Sequential jobs

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  test:
    needs: lint  # Runs after lint succeeds
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: test  # Runs after test succeeds
    runs-on: ubuntu-latest
    steps: [...]
```

### Parallel jobs

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  test:
    runs-on: ubuntu-latest
    steps: [...]

  # lint and test run in parallel
  # build runs after BOTH complete

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps: [...]
```

### Fan-out / fan-in pattern

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    steps: [...]  # Shared setup, output artifacts

  test-unit:
    needs: setup
    runs-on: ubuntu-latest
    steps: [...]

  test-integration:
    needs: setup
    runs-on: ubuntu-latest
    steps: [...]

  test-e2e:
    needs: setup
    runs-on: ubuntu-latest
    steps: [...]

  # All tests run in parallel after setup
  # Deploy only after ALL tests pass

  deploy:
    needs: [test-unit, test-integration, test-e2e]
    runs-on: ubuntu-latest
    steps: [...]
```

### Job design guidelines

- **Fast feedback first**: Run lint and type checks before tests
- **Independent jobs run parallel**: Lint and test don't depend on each other
- **Deploy depends on everything**: Don't deploy if any check fails
- **Balance granularity**: Too many tiny jobs add overhead; too few reduce parallelism
- **Share data via artifacts**: Use `upload-artifact`/`download-artifact` between jobs

---

## Artifacts vs Caches

| Feature | Artifacts | Caches |
|---------|-----------|--------|
| **Purpose** | Store build outputs | Speed up dependency install |
| **Lifetime** | Retained for 90 days (configurable) | Evicted by LRU, 7 days max |
| **Scope** | Per workflow run | Per branch (with fallback to default branch) |
| **Sharing** | Between jobs in same run | Between runs on same branch |
| **Size** | Up to 10 GB per artifact | Up to 10 GB total per repo |
| **Downloadable** | Yes (UI + API) | No (internal only) |

### When to use artifacts

- Build outputs needed by downstream jobs (binaries, Docker images)
- Test reports for inspection (JUnit XML, coverage reports)
- Logs or diagnostics from CI runs
- Release assets to upload to GitHub Releases

### When to use caches

- Dependencies (`node_modules`, `.pip/cache`, `~/.m2/repository`)
- Build tool caches (Gradle daemon, Turborepo cache)
- Docker layer caches

---

## Reusable Workflows

### What they are

Reusable workflows are called from other workflows using `workflow_call`.
They're defined once and invoked from multiple calling workflows.

### When to use

- Same CI pipeline used across multiple repositories
- Standard deployment process across services
- Organization-wide required checks

### Defining a reusable workflow

```yaml
# .github/workflows/reusable-ci.yml
name: Reusable CI

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version to use'
        required: false
        type: string
        default: '22'
      working-directory:
        description: 'Directory to run commands in'
        required: false
        type: string
        default: '.'
    secrets:
      NPM_TOKEN:
        required: false

permissions:
  contents: read

jobs:
  ci:
    name: CI
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@<SHA>  # v4
      - uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

### Calling a reusable workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '22'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

  # Or from another repository:
  ci-external:
    uses: org/ci-templates/.github/workflows/node-ci.yml@v1
    with:
      node-version: '22'
```

### Reusable workflow limitations

- Max 4 levels of nesting (workflow calling workflow calling...)
- `env:` context not available in calling workflow's `with:` (use `inputs`)
- Reusable workflows can't reference secrets from calling workflow unless
  explicitly passed
- Max 20 reusable workflows per workflow file

---

## Composite Actions

### What they are

Composite actions bundle multiple steps into a single reusable action.
They run within a single job (unlike reusable workflows which define entire jobs).

### When to use

- Reusable step sequences within a job
- Shared setup logic (install tools, configure environment)
- Custom logic that's too complex for `run:` but not worth a separate job

### Reusable workflows vs composite actions

| Feature | Reusable Workflow | Composite Action |
|---------|------------------|-----------------|
| Scope | Entire jobs | Steps within a job |
| Triggers | `workflow_call` | `uses:` in any step |
| Secrets | Must be explicitly passed | Inherited from parent job |
| Runner | Each job gets its own runner | Shares parent job's runner |
| Matrix | Can define its own matrix | No (uses parent's matrix) |
| Best for | Multi-job pipelines | Shared step sequences |

### Defining a composite action

```yaml
# .github/actions/setup-project/action.yml
name: 'Setup Project'
description: 'Install dependencies and build tools'

inputs:
  node-version:
    description: 'Node.js version'
    required: false
    default: '22'

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@<SHA>  # v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'

    - name: Install dependencies
      shell: bash
      run: npm ci
```

### Using a composite action

```yaml
steps:
  - uses: actions/checkout@<SHA>  # v4
  - uses: ./.github/actions/setup-project
    with:
      node-version: '22'
  - run: npm test
```

---

## Monorepo Change Detection

### Strategy comparison

| Method | Pros | Cons |
|--------|------|------|
| Path triggers (`on.push.paths`) | Simple, native | Can't share detection across jobs |
| `dorny/paths-filter` | Flexible, outputs for conditionals | Extra step, PR permissions needed |
| Turborepo (`--filter`) | Understands package graph | Requires Turborepo |
| Nx (`affected`) | Understands package graph | Requires Nx |

### Path triggers (simplest)

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'packages/api/**'
      - 'packages/shared/**'
  pull_request:
    paths:
      - 'packages/api/**'
      - 'packages/shared/**'
```

Limitation: the entire workflow runs or doesn't. Can't conditionally run
individual jobs within the workflow.

### dorny/paths-filter (flexible)

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: read
    outputs:
      api: ${{ steps.filter.outputs.api }}
      web: ${{ steps.filter.outputs.web }}
    steps:
      - uses: actions/checkout@<SHA>  # v4
      - uses: dorny/paths-filter@<SHA>  # v3
        id: filter
        with:
          filters: |
            api:
              - 'packages/api/**'
              - 'packages/shared/**'
            web:
              - 'packages/web/**'
              - 'packages/shared/**'

  test-api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    steps: [...]
```

### Turborepo (graph-aware)

```yaml
- name: Build affected packages
  run: npx turbo run build --filter='...[origin/main]'

- name: Test affected packages
  run: npx turbo run test --filter='...[origin/main]'
```

Turborepo understands the package dependency graph: if `shared` changes,
it knows `api` and `web` depend on it and rebuilds them too.

### Nx (graph-aware)

```yaml
- uses: nrwl/nx-set-shas@<SHA>  # v4
- run: npx nx affected --target=build
- run: npx nx affected --target=test
```

---

## Deployment Strategies

### Environment-gated deployment

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - run: deploy --env staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - run: deploy --env production
```

Configure `production` environment in GitHub settings to require:
- Manual approval (required reviewers)
- Branch restriction (only from `main`)
- Wait timer (optional cooldown period)

### Rollback pattern

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy (leave empty for latest)'
        required: false
      rollback:
        description: 'Rollback to previous version?'
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Determine version
        id: version
        env:
          INPUT_ROLLBACK: ${{ inputs.rollback }}
          INPUT_VERSION: ${{ inputs.version }}
          COMMIT_SHA: ${{ github.sha }}
        run: |
          if [ "$INPUT_ROLLBACK" = "true" ]; then
            echo "version=$(get-previous-version)" >> $GITHUB_OUTPUT
          elif [ -n "$INPUT_VERSION" ]; then
            echo "version=$INPUT_VERSION" >> $GITHUB_OUTPUT
          else
            echo "version=$COMMIT_SHA" >> $GITHUB_OUTPUT
          fi

      - name: Deploy
        env:
          DEPLOY_VERSION: ${{ steps.version.outputs.version }}
        run: deploy --version "$DEPLOY_VERSION"
```

### Blue-green with containers

```yaml
- name: Deploy new version
  run: |
    kubectl set image deployment/app \
      app=${{ env.IMAGE }}:${{ github.sha }}

- name: Wait for rollout
  run: kubectl rollout status deployment/app --timeout=300s

- name: Verify health
  run: |
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://example.com/health)
    if [ "$STATUS" != "200" ]; then
      echo "Health check failed, rolling back"
      kubectl rollout undo deployment/app
      exit 1
    fi
```

---

## Release Automation

### Tag-triggered release

```yaml
on:
  push:
    tags: ['v*.*.*']
```

### Release Please (automated version bumping)

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release-please:
    runs-on: ubuntu-latest
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: googleapis/release-please-action@<SHA>  # v4
        id: release
        with:
          release-type: node  # or python, go, etc.

  publish:
    needs: release-please
    if: needs.release-please.outputs.release_created
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<SHA>  # v4
      - run: npm ci && npm run build && npm publish
```

### Semantic Release

```yaml
- name: Semantic Release
  uses: cycjimmy/semantic-release-action@<SHA>  # v4
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## Scheduled Pipelines

### Common schedules

```yaml
on:
  schedule:
    # Cron syntax: minute hour day-of-month month day-of-week
    - cron: '0 6 * * 1'      # Weekly: Monday at 06:00 UTC
    - cron: '0 0 * * *'      # Daily: midnight UTC
    - cron: '0 */6 * * *'    # Every 6 hours
    - cron: '0 9 1 * *'      # Monthly: 1st at 09:00 UTC
```

### Use cases for scheduled pipelines

| Schedule | Purpose |
|----------|---------|
| Daily | Security scans, dependency audits |
| Weekly | Performance benchmarks, stale issue cleanup |
| Monthly | License compliance checks |
| Nightly | Full integration test suite (expensive tests) |

### Always add workflow_dispatch

```yaml
on:
  schedule:
    - cron: '0 6 * * 1'
  workflow_dispatch:  # Allow manual trigger for debugging
```

---

## Conditional Execution

### Common conditions

```yaml
# Run only on main branch
if: github.ref == 'refs/heads/main'

# Run only on pull requests
if: github.event_name == 'pull_request'

# Run only on tag push
if: startsWith(github.ref, 'refs/tags/')

# Skip if commit message contains [skip ci]
if: "!contains(github.event.head_commit.message, '[skip ci]')"

# Run only if previous job succeeded
if: success()

# Run even if previous job failed (cleanup)
if: always()

# Run only if previous job failed
if: failure()

# Run only for non-fork PRs (has secret access)
if: github.event.pull_request.head.repo.full_name == github.repository
```

### Path-based conditionals (in steps)

```yaml
- name: Check for doc changes
  id: docs
  run: |
    if git diff --name-only HEAD~1 | grep -q '^docs/'; then
      echo "changed=true" >> $GITHUB_OUTPUT
    fi

- name: Build docs
  if: steps.docs.outputs.changed == 'true'
  run: npm run docs:build
```

---

## Service Containers

### Database for integration tests

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@<SHA>  # v4
      - run: npm ci
      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
        run: npm run test:integration
```

---

## Cost Optimization

### Strategies to reduce Actions minutes

| Strategy | Impact | How |
|----------|--------|-----|
| **Path filters** | High | Skip CI for docs/config-only changes |
| **Concurrency groups** | High | Cancel stale runs on rapid pushes |
| **Caching** | High | Avoid re-downloading dependencies |
| **Matrix reduction** | Medium | Test 2-3 versions, not 6+ |
| **Conditional jobs** | Medium | Skip deploy job on PRs |
| **Smaller runners** | Low | Use ubuntu-latest unless you need more |
| **Shorter timeouts** | Low | Set `timeout-minutes` to catch hung jobs |

### Timeout best practices

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15  # Kill if tests hang
    steps: [...]

  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 30  # Deploy might be slower
    steps: [...]
```

### When to consider self-hosted runners

- Builds consistently use >16 GB RAM
- Builds take >30 minutes on GitHub-hosted runners
- Need GPU for ML workloads
- Need persistent local caches (very large dependencies)
- Need access to internal network resources
- Monthly Actions bill exceeds runner infrastructure cost
