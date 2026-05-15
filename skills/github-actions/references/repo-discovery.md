# Repository Discovery Framework

Deterministic repository analysis for GitHub Actions workflow generation.
Follow this sequence to identify project type, ecosystem, and pipeline needs.

## Table of Contents
- [Discovery Sequence](#discovery-sequence)
- [Step 1: Language Detection](#step-1-language-detection)
- [Step 2: Package Manager Detection](#step-2-package-manager-detection)
- [Step 3: Framework Detection](#step-3-framework-detection)
- [Step 4: Build System Detection](#step-4-build-system-detection)
- [Step 5: Test Infrastructure Detection](#step-5-test-infrastructure-detection)
- [Step 6: Lint and Format Detection](#step-6-lint-and-format-detection)
- [Step 7: Container Detection](#step-7-container-detection)
- [Step 8: Infrastructure as Code Detection](#step-8-infrastructure-as-code-detection)
- [Step 9: Monorepo Detection](#step-9-monorepo-detection)
- [Step 10: Deployment Target Detection](#step-10-deployment-target-detection)
- [Step 11: Existing CI Detection](#step-11-existing-ci-detection)
- [Step 12: Branch and Release Strategy](#step-12-branch-and-release-strategy)
- [Composite Profile](#composite-profile)
- [Pipeline Decision Matrix](#pipeline-decision-matrix)

---

## Discovery Sequence

Run each step in order. Each step builds on the previous.

```
Step 1:  Scan root files → identify language(s)
Step 2:  Read manifests → identify package manager(s)
Step 3:  Read dependencies → identify framework(s)
Step 4:  Check config files → identify build system
Step 5:  Check test config → identify test infrastructure
Step 6:  Check lint config → identify quality tools
Step 7:  Check Dockerfiles → identify container needs
Step 8:  Check IaC files → identify infrastructure pipeline
Step 9:  Check workspace config → identify monorepo structure
Step 10: Check deploy config → identify deployment targets
Step 11: Check CI config → identify existing pipelines
Step 12: Check branches/tags → identify release strategy
```

---

## Step 1: Language Detection

Scan the repository root for manifest files. Each file maps to a language.

| File | Language | Confidence |
|------|----------|------------|
| `package.json` | JavaScript / TypeScript | High |
| `tsconfig.json` | TypeScript | High |
| `pyproject.toml` | Python | High |
| `requirements.txt` | Python | Medium |
| `setup.py` | Python | Medium |
| `Pipfile` | Python | Medium |
| `go.mod` | Go | High |
| `Cargo.toml` | Rust | High |
| `pom.xml` | Java | High |
| `build.gradle` or `build.gradle.kts` | Java / Kotlin | High |
| `Gemfile` | Ruby | High |
| `*.csproj` or `*.sln` | C# / .NET | High |
| `mix.exs` | Elixir | High |
| `pubspec.yaml` | Dart / Flutter | High |
| `composer.json` | PHP | High |
| `Package.swift` | Swift | High |

**Multiple languages**: If multiple manifest files exist, the project is
polyglot. Plan separate CI jobs or a unified workflow with matrix strategy.

**TypeScript detection**: If both `package.json` and `tsconfig.json` exist,
classify as TypeScript (not just JavaScript).

---

## Step 2: Package Manager Detection

For each detected language, identify the package manager from lockfiles.

### Node.js / TypeScript

| File | Package Manager | Install Command |
|------|----------------|-----------------|
| `package-lock.json` | npm | `npm ci` |
| `yarn.lock` | Yarn (Classic or Berry) | `yarn install --frozen-lockfile` |
| `pnpm-lock.yaml` | pnpm | `pnpm install --frozen-lockfile` |
| `bun.lockb` | Bun | `bun install --frozen-lockfile` |

If no lockfile exists, check `package.json` for `packageManager` field.
Default to npm if ambiguous.

### Python

| File | Package Manager | Install Command |
|------|----------------|-----------------|
| `requirements.txt` | pip | `pip install -r requirements.txt` |
| `Pipfile.lock` | pipenv | `pipenv install --deploy` |
| `poetry.lock` | Poetry | `poetry install --no-interaction` |
| `uv.lock` | uv | `uv sync` |
| `pyproject.toml` (with `[tool.hatch]`) | Hatch | `hatch env create` |
| `pyproject.toml` (with `[build-system]`) | pip/build | `pip install -e .` |

### Go

Always `go mod download`. Check for `go.sum`.

### Rust

Always `cargo`. Check `Cargo.lock` presence (should be committed for
binaries, optional for libraries).

### Java

| File | Build Tool | Build Command |
|------|-----------|---------------|
| `pom.xml` | Maven | `mvn verify` |
| `build.gradle` | Gradle (Groovy) | `./gradlew build` |
| `build.gradle.kts` | Gradle (Kotlin) | `./gradlew build` |

### .NET / C#

Always `dotnet`. Check for `*.sln` (solution) or `*.csproj` (project) files.

| File | Indicator | Command |
|------|-----------|---------|
| `*.sln` | Multi-project solution | `dotnet build MySolution.sln` |
| `*.csproj` | Single project | `dotnet build` |
| `global.json` | Pinned SDK version | Use version from `global.json` |
| `NuGet.config` | Custom NuGet sources | Restore may need auth |

### Ruby

Always `bundler`. Check for `Gemfile.lock`.

| File | Install Command |
|------|-----------------|
| `Gemfile.lock` | `bundle install` |

### PHP

| File | Package Manager | Install Command |
|------|----------------|-----------------|
| `composer.lock` | Composer | `composer install --no-dev --prefer-dist` |

### Elixir

| File | Package Manager | Install Command |
|------|----------------|-----------------|
| `mix.lock` | Mix | `mix deps.get` |

### Dart / Flutter

| File | Package Manager | Install Command |
|------|----------------|-----------------|
| `pubspec.lock` | pub | `dart pub get` or `flutter pub get` |

### Swift

Always Swift Package Manager. Check for `Package.resolved`.

---

## Step 3: Framework Detection

Read the dependency manifest to identify frameworks.

### Node.js / TypeScript Frameworks

| Dependency or Config | Framework | Pipeline Impact |
|---------------------|-----------|----------------|
| `next` in deps, `next.config.*` | Next.js | SSR build, Vercel deploy |
| `nuxt` in deps, `nuxt.config.*` | Nuxt.js | SSR build |
| `react` in deps (no Next) | React (CRA/Vite) | Static build |
| `@angular/core` in deps, `angular.json` | Angular | `ng build`, `ng test` |
| `vue` in deps (no Nuxt) | Vue.js | Static build |
| `express` in deps | Express.js | Server, Dockerfile likely |
| `@nestjs/core` in deps | NestJS | Server, Dockerfile likely |
| `svelte` in deps, `svelte.config.*` | SvelteKit | Build + deploy |

### Python Frameworks

| Dependency or Config | Framework | Pipeline Impact |
|---------------------|-----------|----------------|
| `django` in deps, `manage.py` | Django | `python manage.py test` |
| `flask` in deps | Flask | pytest, Dockerfile likely |
| `fastapi` in deps | FastAPI | pytest, Dockerfile likely |
| `scrapy` in deps | Scrapy | pytest |

### Go Frameworks

| File or Pattern | Framework | Pipeline Impact |
|----------------|-----------|----------------|
| `cmd/` directory | CLI / binary | `go build ./cmd/...` |
| HTTP handler imports | Web service | Dockerfile likely |

### Java Frameworks

| Dependency | Framework | Pipeline Impact |
|-----------|-----------|----------------|
| `spring-boot` in pom/gradle | Spring Boot | `./gradlew bootJar` |
| `quarkus` in pom/gradle | Quarkus | Native build option |

### .NET / C# Frameworks

| Dependency or Config | Framework | Pipeline Impact |
|---------------------|-----------|----------------|
| `Microsoft.AspNetCore` in csproj | ASP.NET Core | `dotnet publish`, Dockerfile likely |
| `Microsoft.Maui` in csproj | .NET MAUI | Cross-platform build matrix |
| `Blazor` references in csproj | Blazor | `dotnet publish`, static or server |
| `Microsoft.Azure.Functions` | Azure Functions | `func azure functionapp publish` |

### Ruby Frameworks

| Dependency or Config | Framework | Pipeline Impact |
|---------------------|-----------|----------------|
| `rails` in Gemfile, `config/routes.rb` | Ruby on Rails | `rails test`, service containers (DB) |
| `sinatra` in Gemfile | Sinatra | `rake test` or `rspec` |
| `jekyll` in Gemfile | Jekyll | `jekyll build`, static site deploy |

### PHP Frameworks

| Dependency or Config | Framework | Pipeline Impact |
|---------------------|-----------|----------------|
| `laravel/framework` in composer.json | Laravel | `php artisan test`, service containers |
| `symfony/` in composer.json | Symfony | `php bin/phpunit` |

---

## Step 4: Build System Detection

Check for build orchestration tools beyond language-native tooling.

| File | Build System | Impact |
|------|-------------|--------|
| `Makefile` | Make | Use `make` targets for CI steps |
| `justfile` | Just | Use `just` commands for CI steps |
| `Taskfile.yml` | Task | Use `task` commands for CI steps |
| `turbo.json` | Turborepo | Use `turbo run build` for monorepo |
| `nx.json` | Nx | Use `nx affected` for monorepo |
| `webpack.config.*` | Webpack | `npm run build` produces bundle |
| `vite.config.*` | Vite | `npm run build` produces bundle |
| `rollup.config.*` | Rollup | Library bundling |
| `esbuild.*` or esbuild in scripts | esbuild | Fast bundling |

---

## Step 5: Test Infrastructure Detection

Check for test framework configuration.

### Test Config Files

| File | Test Framework | Run Command |
|------|---------------|-------------|
| `jest.config.*` | Jest | `npx jest` or `npm test` |
| `vitest.config.*` | Vitest | `npx vitest run` or `npm test` |
| `.mocharc.*` | Mocha | `npx mocha` |
| `playwright.config.*` | Playwright | `npx playwright test` |
| `cypress.json` or `cypress.config.*` | Cypress | `npx cypress run` |
| `pytest.ini` or `conftest.py` | pytest | `pytest` |
| `tox.ini` | tox | `tox` |
| `.noxfile` | nox | `nox` |
| `*_test.go` files | Go testing | `go test ./...` |
| `Cargo.toml` (with `[dev-dependencies]`) | cargo test | `cargo test` |
| `build.gradle` (with test dependencies) | JUnit | `./gradlew test` |
| `*.Tests.csproj` or `xunit`/`nunit` refs | xUnit / NUnit | `dotnet test` |
| `spec/` directory, `.rspec` | RSpec | `bundle exec rspec` |
| `test/` directory (Ruby) | Minitest | `bundle exec rake test` |
| `phpunit.xml` or `phpunit.xml.dist` | PHPUnit | `./vendor/bin/phpunit` |
| `test/` directory (Elixir) | ExUnit | `mix test` |

### Test Script Detection

Check the manifest for test commands:

- `package.json` → `scripts.test`
- `Makefile` → `test` target
- `pyproject.toml` → `[tool.pytest]` or `[tool.tox]`

---

## Step 6: Lint and Format Detection

Check for quality tooling configuration.

| File | Tool | Command |
|------|------|---------|
| `.eslintrc.*` or `eslint.config.*` | ESLint | `npx eslint .` |
| `.prettierrc.*` | Prettier | `npx prettier --check .` |
| `biome.json` | Biome | `npx biome check .` |
| `ruff.toml` or `[tool.ruff]` in pyproject.toml | Ruff | `ruff check .` |
| `.flake8` or `[flake8]` in setup.cfg | Flake8 | `flake8 .` |
| `mypy.ini` or `[tool.mypy]` in pyproject.toml | mypy | `mypy .` |
| `pyright` config or `pyrightconfig.json` | Pyright | `pyright` |
| `.golangci.yml` | golangci-lint | `golangci-lint run` |
| `clippy` in Rust deps | Clippy | `cargo clippy` |
| `rustfmt.toml` | rustfmt | `cargo fmt --check` |
| `checkstyle.xml` | Checkstyle | Via Gradle/Maven plugin |
| `spotbugs` in build config | SpotBugs | Via Gradle/Maven plugin |
| `.editorconfig` + `dotnet format` | dotnet format | `dotnet format --verify-no-changes` |
| `.rubocop.yml` | RuboCop | `bundle exec rubocop` |
| `phpcs.xml` or `.php-cs-fixer.php` | PHP_CodeSniffer / PHP CS Fixer | `./vendor/bin/phpcs` |
| `phpstan.neon` | PHPStan | `./vendor/bin/phpstan analyse` |
| `.credo.exs` | Credo (Elixir) | `mix credo` |
| `.formatter.exs` | Elixir formatter | `mix format --check-formatted` |

---

## Step 7: Container Detection

Check for Docker-related files.

| File | Indicates | Pipeline Action |
|------|-----------|----------------|
| `Dockerfile` (root) | Container build | Add Docker build/push job |
| `Dockerfile.*` (variants) | Multi-stage/multi-target | Build specific target |
| `docker-compose.yml` | Multi-container app | Use for integration tests |
| `.dockerignore` | Docker build context | Confirms container use |
| `Dockerfile` in subdirectories | Per-service builds (monorepo) | Build per changed service |

### Registry Detection

| Config | Registry |
|--------|----------|
| Default / `.github/` context | GitHub Container Registry (ghcr.io) |
| `AWS_ACCOUNT_ID` secrets | Amazon ECR |
| `GCP_PROJECT` secrets | Google Artifact Registry |
| `DOCKER_USERNAME` secret | Docker Hub |
| `ACR_LOGIN_SERVER` secret | Azure Container Registry |

Default to GHCR unless another registry is clearly indicated.

---

## Step 8: Infrastructure as Code Detection

| File/Directory | Tool | Pipeline Action |
|---------------|------|----------------|
| `*.tf` files, `terraform/` | Terraform | Plan on PR, apply on merge |
| `Pulumi.yaml` | Pulumi | Preview on PR, up on merge |
| `template.yaml` (SAM) | AWS SAM | Build + deploy |
| `serverless.yml` | Serverless Framework | Deploy on merge |
| `cdk.json` | AWS CDK | Synth + deploy |
| `bicep` files | Azure Bicep | What-if on PR, deploy on merge |
| `cloudformation/` | CloudFormation | Validate + deploy |

---

## Step 9: Monorepo Detection

| Indicator | Tool | Strategy |
|-----------|------|----------|
| `workspaces` in package.json | npm/yarn workspaces | Path-filtered workflows |
| `pnpm-workspace.yaml` | pnpm workspaces | Path-filtered workflows |
| `nx.json` | Nx | `nx affected` for change detection |
| `turbo.json` | Turborepo | `turbo run --filter` for change detection |
| `lerna.json` | Lerna | Per-package CI |
| Multiple `go.mod` files | Go multi-module | Per-module CI |
| Multiple `Cargo.toml` with `[workspace]` | Cargo workspace | `cargo test --workspace` |
| Multiple independent `package.json` | Loose monorepo | dorny/paths-filter |
| `packages/`, `services/`, `apps/` directories | Convention-based | Path-filtered workflows |

### Monorepo pipeline decision

```
Monorepo detected?
  |
  NO --> Single workflow covers everything
  |
  YES --> Does a build orchestrator exist? (Nx, Turborepo)
           |
           YES --> Use orchestrator's affected/filter commands
           |       Single workflow with change detection
           |
           NO  --> Use path-based triggers or dorny/paths-filter
                   Separate workflows per package/service OR
                   Single workflow with conditional jobs
```

---

## Step 10: Deployment Target Detection

| Indicator | Platform | Deploy Method |
|-----------|----------|---------------|
| `vercel.json` or `vercel` in deps | Vercel | Vercel CLI or auto-deploy |
| `netlify.toml` | Netlify | Netlify CLI or auto-deploy |
| `fly.toml` | Fly.io | `flyctl deploy` |
| `render.yaml` | Render | Auto-deploy or API |
| `railway.json` | Railway | Railway CLI |
| `app.yaml` (GAE) | Google App Engine | `gcloud app deploy` |
| `appspec.yml` | AWS CodeDeploy | CodeDeploy integration |
| `Procfile` | Heroku | Heroku CLI |
| Kubernetes manifests (`k8s/`, `deploy/`) | Kubernetes | `kubectl apply` |
| `helm/` directory | Helm | `helm upgrade --install` |
| `kustomization.yaml` | Kustomize | `kubectl apply -k` |
| `argocd/` or ArgoCD annotations | ArgoCD | GitOps (push manifests) |
| AWS CDK / SAM / Serverless config | AWS | Framework-specific deploy |
| `azuredeploy.json` or Bicep | Azure | `az deployment` |
| GitHub Pages config | GitHub Pages | `actions/deploy-pages` |

---

## Step 11: Existing CI Detection

Check for CI configurations from any system.

| File/Directory | CI System | Migration Notes |
|---------------|-----------|----------------|
| `.github/workflows/*.yml` | GitHub Actions | Already using Actions |
| `.circleci/config.yml` | CircleCI | Map jobs/orbs to Actions |
| `Jenkinsfile` | Jenkins | Map stages to jobs |
| `.gitlab-ci.yml` | GitLab CI | Map stages/jobs to Actions |
| `.travis.yml` | Travis CI | Map matrix to Actions matrix |
| `azure-pipelines.yml` | Azure Pipelines | Map stages to jobs |
| `bitbucket-pipelines.yml` | Bitbucket Pipelines | Map steps to Actions steps |
| `buildkite.yml` or `.buildkite/` | Buildkite | Map steps to Actions |
| `cloudbuild.yaml` | Google Cloud Build | Map steps to Actions |
| `Makefile` with CI targets | Make-based CI | Wrap make targets in Actions |

### Existing GitHub Actions analysis

If `.github/workflows/` already exists:
1. Count workflow files
2. For each: read triggers, jobs, purpose
3. Identify gaps (e.g., CI exists but no CD, no security scans)
4. Identify issues (unpinned actions, missing permissions)

---

## Step 12: Branch and Release Strategy

### Branch detection

```bash
# Check default branch
git remote show origin | grep 'HEAD branch'

# Check for release branches
git branch -r | grep -E 'release|main|master|develop'

# Check for tags
git tag --list | tail -5
```

| Pattern | Strategy | Pipeline Impact |
|---------|----------|----------------|
| `main` only | Trunk-based | CI on PR, CD on merge to main |
| `main` + `develop` | Git Flow | CI on PR to develop, CD on merge to main |
| `main` + `release/*` | Release branches | CI always, CD on release branch |
| Semantic version tags | Tag-based release | Release workflow on tag push |
| `v*` tags | Versioned releases | Publish on tag creation |

### Release detection

| Indicator | Release Method |
|-----------|---------------|
| `CHANGELOG.md` exists | Manual releases with changelog |
| `release-please-config.json` | Automated release with Release Please |
| `.releaserc` or `release.config.js` | Semantic Release |
| `version` in package.json/pyproject.toml | Version in manifest |
| GitHub Releases page has entries | GitHub Releases used |
| NPM/PyPI package exists with same name | Package publishing |

---

## Composite Profile

After running all steps, compile the results:

```
=== Repository Profile ===
Languages: TypeScript (primary), Python (secondary)
Package managers: pnpm (Node), uv (Python)
Framework: Next.js (frontend), FastAPI (backend)
Build system: Turborepo (monorepo orchestrator)
Test framework: Vitest (Node), pytest (Python)
Lint/format: ESLint + Prettier (Node), Ruff + mypy (Python)
Container: Dockerfile in services/api/
IaC: Terraform in infra/
Monorepo: Yes (Turborepo, packages: web, api, shared)
Deploy target: Vercel (web), AWS ECS (api)
Existing CI: ci.yml (lint + test), incomplete (no CD)
Branch strategy: main + PR, semantic version tags
Release: tag-based (v*.*.*)
```

---

## Pipeline Decision Matrix

Use this matrix to determine which workflows to generate based on
the profile above.

| Profile Attribute | Workflow to Generate |
|-------------------|---------------------|
| Any language detected | CI workflow (lint + test + build) |
| Dockerfile exists | Docker build/push workflow |
| IaC files exist | Infrastructure validation/deploy workflow |
| Deploy target detected | CD workflow with environments |
| Library (npm/PyPI/crates.io) | Release/publish workflow |
| Monorepo detected | Change detection + conditional jobs |
| No Dependabot config | `.github/dependabot.yml` |
| No branch protection | Recommend branch protection rules |
| Existing CI from other system | Migration workflow(s) |

### Pipeline complexity tiers

| Tier | Profile | Workflows |
|------|---------|-----------|
| **Simple** | Single language, no deploy | 1 CI workflow + dependabot |
| **Standard** | Single language + deploy target | CI + CD + dependabot |
| **Full** | Multi-language or monorepo + deploy | CI + CD + release + dependabot |
| **Enterprise** | Monorepo + IaC + multi-env deploy | CI + CD + IaC + release + security + dependabot |
