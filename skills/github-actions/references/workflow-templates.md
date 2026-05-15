# Workflow Templates

Production-grade GitHub Actions workflow templates per language ecosystem.
Customize for each project based on repository discovery results.

**All templates follow security hardening rules:**
- Explicit `permissions:` (least privilege)
- Actions pinned to SHA (shown as `@<SHA>` — replace with actual SHAs)
- Concurrency groups enabled
- Dependency caching enabled

## Table of Contents
- [Node.js / TypeScript CI](#nodejs--typescript-ci)
- [Python CI](#python-ci)
- [Go CI](#go-ci)
- [Rust CI](#rust-ci)
- [Java (Gradle) CI](#java-gradle-ci)
- [Java (Maven) CI](#java-maven-ci)
- [.NET CI](#net-ci)
- [Ruby CI](#ruby-ci)
- [Docker Build and Push](#docker-build-and-push)
- [Terraform Validation and Deploy](#terraform-validation-and-deploy)
- [Release: npm Publish](#release-npm-publish)
- [Release: PyPI Publish](#release-pypi-publish)
- [Release: Go Binary](#release-go-binary)
- [Release: Rust Binary](#release-rust-binary)
- [Release: GitHub Release](#release-github-release)
- [Monorepo CI (Turborepo)](#monorepo-ci-turborepo)
- [Monorepo CI (Nx)](#monorepo-ci-nx)
- [Monorepo CI (Path Filter)](#monorepo-ci-path-filter)
- [Scheduled Security Scan](#scheduled-security-scan)
- [Dependabot Configuration](#dependabot-configuration)

---

## Node.js / TypeScript CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Lint, Test, Build
    runs-on: ubuntu-latest
    timeout-minutes: 15
    strategy:
      matrix:
        node-version: [20, 22]
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      # Uncomment for pnpm:
      # - name: Install pnpm
      #   uses: pnpm/action-setup@<SHA>  # v4
      #   with:
      #     version: 9

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'  # or 'pnpm' or 'yarn'

      - name: Install dependencies
        run: npm ci  # or pnpm install --frozen-lockfile

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npx tsc --noEmit
        # Remove if not using TypeScript

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

**Customization points:**
- `cache:` — match your package manager (npm, pnpm, yarn)
- `node-version:` — match your supported versions
- Install command — `npm ci`, `pnpm install --frozen-lockfile`, `yarn install --frozen-lockfile`
- Lint command — may be `npm run lint`, `npx eslint .`, `npx biome check .`
- Type check — remove if JavaScript-only
- Test command — may include coverage: `npm test -- --coverage`

---

## Python CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Lint and Test
    runs-on: ubuntu-latest
    timeout-minutes: 15
    strategy:
      matrix:
        python-version: ['3.11', '3.12', '3.13']
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@<SHA>  # v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'  # or 'pipenv' or 'poetry'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          # Or: pip install -e ".[dev]"
          # Or: poetry install --no-interaction
          # Or: uv sync

      - name: Lint
        run: ruff check .
        # Or: flake8 .

      - name: Format check
        run: ruff format --check .
        # Or: black --check .

      - name: Type check
        run: mypy .
        # Remove if not using type checking

      - name: Test
        run: pytest --tb=short --junitxml=reports/results.xml

      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@<SHA>  # v4
        with:
          name: test-results-py${{ matrix.python-version }}
          path: reports/results.xml
```

**Customization points:**
- `python-version:` — match your supported versions
- `cache:` — match your package manager (pip, pipenv, poetry)
- Install command — varies by package manager
- Lint tools — Ruff, Flake8, Pylint, etc.
- Type checker — mypy, pyright, pytype

---

## Go CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Lint, Test, Build
    runs-on: ubuntu-latest
    timeout-minutes: 15
    strategy:
      matrix:
        go-version: ['1.22', '1.23']
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Go ${{ matrix.go-version }}
        uses: actions/setup-go@<SHA>  # v5
        with:
          go-version: ${{ matrix.go-version }}
          # Caching is enabled by default in setup-go v5

      - name: Download dependencies
        run: go mod download

      - name: Verify dependencies
        run: go mod verify

      - name: Lint
        uses: golangci/golangci-lint-action@<SHA>  # v6
        with:
          version: latest

      - name: Test
        run: go test -v -race -coverprofile=coverage.out ./...

      - name: Build
        run: go build -v ./...
```

**Customization points:**
- `go-version:` — match your supported versions
- Build command — may target specific binaries: `go build -v ./cmd/myapp`
- golangci-lint config — reads from `.golangci.yml` if present
- Coverage — add `go tool cover -html=coverage.out` for HTML reports

---

## Rust CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Check, Test, Clippy
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Rust toolchain
        uses: actions-rust-lang/setup-rust-toolchain@<SHA>  # v1
        with:
          components: clippy, rustfmt

      - name: Format check
        run: cargo fmt --all --check

      - name: Clippy
        run: cargo clippy --all-targets --all-features -- -D warnings

      - name: Test
        run: cargo test --all-features

      - name: Build
        run: cargo build --release
```

**Customization points:**
- `components:` — add `llvm-tools-preview` for coverage
- Toolchain — defaults to stable; specify `toolchain: nightly` if needed
- Features — adjust `--all-features` or `--no-default-features` as needed
- Cache — `setup-rust-toolchain` includes Swatinem/rust-cache by default

---

## Java (Gradle) CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Build and Test
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup JDK 21
        uses: actions/setup-java@<SHA>  # v4
        with:
          distribution: temurin
          java-version: 21
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build --no-daemon --stacktrace

      - name: Upload test reports
        if: always()
        uses: actions/upload-artifact@<SHA>  # v4
        with:
          name: test-reports
          path: build/reports/tests/
```

---

## Java (Maven) CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Build and Test
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup JDK 21
        uses: actions/setup-java@<SHA>  # v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven

      - name: Build with Maven
        run: mvn -B verify --no-transfer-progress
```

---

## .NET CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Build and Test
    runs-on: ubuntu-latest
    timeout-minutes: 15
    strategy:
      matrix:
        dotnet-version: ['8.0', '9.0']
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup .NET ${{ matrix.dotnet-version }}
        uses: actions/setup-dotnet@<SHA>  # v4
        with:
          dotnet-version: ${{ matrix.dotnet-version }}

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore --configuration Release

      - name: Test
        run: dotnet test --no-build --configuration Release --verbosity normal --logger "trx;LogFileName=results.trx"

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@<SHA>  # v4
        with:
          name: test-results-dotnet${{ matrix.dotnet-version }}
          path: '**/TestResults/*.trx'
```

**Customization points:**
- `dotnet-version:` — match your target frameworks (6.0, 7.0, 8.0, 9.0)
- Solution file — add `--solution MySolution.sln` if not auto-detected
- Format check — add `dotnet format --verify-no-changes` step
- Static analysis — add `dotnet build /p:TreatWarningsAsErrors=true`

---

## Ruby CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Lint and Test
    runs-on: ubuntu-latest
    timeout-minutes: 15
    strategy:
      matrix:
        ruby-version: ['3.2', '3.3']
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Ruby ${{ matrix.ruby-version }}
        uses: ruby/setup-ruby@<SHA>  # v1
        with:
          ruby-version: ${{ matrix.ruby-version }}
          bundler-cache: true  # Caches gems automatically

      - name: Lint
        run: bundle exec rubocop
        # Remove if not using RuboCop

      - name: Test
        run: bundle exec rake test
        # Or: bundle exec rspec
```

**Customization points:**
- `ruby-version:` — match your supported versions
- `bundler-cache: true` — automatically caches gems based on Gemfile.lock
- Test command — `rake test` (Minitest) or `rspec` (RSpec)
- Lint — RuboCop, StandardRB, or remove if not used
- For Rails: add service containers for PostgreSQL/Redis (see pipeline-patterns.md)

---

## Docker Build and Push

```yaml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags: ['v*.*.*']

permissions:
  contents: read
  packages: write

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-push:
    name: Build and Push Image
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@<SHA>  # v3

      - name: Log in to Container Registry
        uses: docker/login-action@<SHA>  # v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels)
        id: meta
        uses: docker/metadata-action@<SHA>  # v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@<SHA>  # v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**Customization points:**
- `REGISTRY` — change to Docker Hub, ECR, GCR, ACR as needed
- `tags:` in metadata — customize tagging strategy
- `cache-from/cache-to` — uses GitHub Actions cache for layer caching
- Add `platforms: linux/amd64,linux/arm64` for multi-arch builds

---

## Terraform Validation and Deploy

```yaml
name: Terraform

on:
  push:
    branches: [main]
    paths: ['infra/**']
  pull_request:
    paths: ['infra/**']

permissions:
  contents: read
  pull-requests: write
  id-token: write  # For OIDC

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false  # Don't cancel infra deploys

defaults:
  run:
    working-directory: infra

jobs:
  plan:
    name: Terraform Plan
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@<SHA>  # v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@<SHA>  # v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Terraform Init
        run: terraform init

      - name: Terraform Format Check
        run: terraform fmt -check

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan -no-color -out=tfplan

      - name: Upload plan
        uses: actions/upload-artifact@<SHA>  # v4
        with:
          name: tfplan
          path: infra/tfplan
          retention-days: 1

      - name: Post plan to PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@<SHA>  # v7
        with:
          script: |
            // Post plan output as PR comment

  apply:
    name: Terraform Apply
    runs-on: ubuntu-latest
    timeout-minutes: 30
    needs: plan
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@<SHA>  # v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@<SHA>  # v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Terraform Init
        run: terraform init

      - name: Download plan
        uses: actions/download-artifact@<SHA>  # v4
        with:
          name: tfplan
          path: infra

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
```

---

## Release: npm Publish

```yaml
name: Publish Package

on:
  push:
    tags: ['v*.*.*']

permissions:
  contents: read
  id-token: write  # For npm provenance

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}

jobs:
  publish:
    name: Publish to npm
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Node.js
        uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: 22
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Publish
        run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## Release: PyPI Publish

```yaml
name: Publish Package

on:
  push:
    tags: ['v*.*.*']

permissions:
  contents: read
  id-token: write  # For trusted publishing

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}

jobs:
  publish:
    name: Publish to PyPI
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment: pypi  # Configure trusted publishing in PyPI
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Python
        uses: actions/setup-python@<SHA>  # v5
        with:
          python-version: '3.12'

      - name: Install build tools
        run: pip install build

      - name: Build package
        run: python -m build

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@<SHA>  # v1
        # Uses OIDC trusted publishing — no token needed
```

---

## Release: Go Binary

```yaml
name: Release

on:
  push:
    tags: ['v*.*.*']

permissions:
  contents: write  # Create GitHub Release

jobs:
  build:
    name: Build Binary
    runs-on: ${{ matrix.os }}
    timeout-minutes: 15
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            goos: linux
            goarch: amd64
            artifact: myapp-linux-amd64
          - os: macos-latest
            goos: darwin
            goarch: arm64
            artifact: myapp-darwin-arm64
          - os: windows-latest
            goos: windows
            goarch: amd64
            artifact: myapp-windows-amd64.exe
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Go
        uses: actions/setup-go@<SHA>  # v5
        with:
          go-version-file: go.mod

      - name: Build
        env:
          GOOS: ${{ matrix.goos }}
          GOARCH: ${{ matrix.goarch }}
        run: go build -o ${{ matrix.artifact }} ./cmd/myapp

      - name: Upload release asset
        uses: actions/upload-artifact@<SHA>  # v4
        with:
          name: ${{ matrix.artifact }}
          path: ${{ matrix.artifact }}

  create-release:
    name: Create GitHub Release
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@<SHA>  # v4

      - name: Create Release
        uses: actions/github-script@<SHA>  # v7
        with:
          script: |
            const fs = require('fs');
            const release = await github.rest.repos.createRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              tag_name: context.ref.replace('refs/tags/', ''),
              generate_release_notes: true,
            });
            // Upload artifacts to release
```

**Customization points:**
- Matrix entries — add/remove OS/arch combos as needed
- Build command — adjust `./cmd/myapp` to your binary path
- Use `go-version-file: go.mod` to match the project's Go version
- Add `-ldflags "-s -w -X main.version=${{ github.ref_name }}"` for version embedding

---

## Release: Rust Binary

```yaml
name: Release

on:
  push:
    tags: ['v*.*.*']

permissions:
  contents: write  # Create GitHub Release

jobs:
  build:
    name: Build Binary
    runs-on: ${{ matrix.os }}
    timeout-minutes: 30
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            target: x86_64-unknown-linux-gnu
            artifact: myapp-linux-amd64
          - os: macos-latest
            target: aarch64-apple-darwin
            artifact: myapp-darwin-arm64
          - os: windows-latest
            target: x86_64-pc-windows-msvc
            artifact: myapp-windows-amd64.exe
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Setup Rust
        uses: actions-rust-lang/setup-rust-toolchain@<SHA>  # v1
        with:
          target: ${{ matrix.target }}

      - name: Build
        run: cargo build --release --target ${{ matrix.target }}

      - name: Rename binary
        shell: bash
        run: |
          SRC=target/${{ matrix.target }}/release/myapp
          if [ "${{ runner.os }}" = "Windows" ]; then SRC="${SRC}.exe"; fi
          cp "$SRC" "${{ matrix.artifact }}"

      - name: Upload release asset
        uses: actions/upload-artifact@<SHA>  # v4
        with:
          name: ${{ matrix.artifact }}
          path: ${{ matrix.artifact }}

  create-release:
    name: Create GitHub Release
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@<SHA>  # v4

      - name: Create Release
        uses: actions/github-script@<SHA>  # v7
        with:
          script: |
            const fs = require('fs');
            const release = await github.rest.repos.createRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              tag_name: context.ref.replace('refs/tags/', ''),
              generate_release_notes: true,
            });
            // Upload artifacts to release
```

**Customization points:**
- `target:` — add targets for your supported platforms
- Binary name — replace `myapp` with your crate's binary name
- Add `--features` flags if needed for release builds
- Consider `cargo-zigbuild` for easier cross-compilation

---

## Release: GitHub Release

```yaml
name: Release

on:
  push:
    tags: ['v*.*.*']

permissions:
  contents: write

jobs:
  release:
    name: Create Release
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4
        with:
          fetch-depth: 0  # Full history for release notes

      - name: Create GitHub Release
        uses: actions/github-script@<SHA>  # v7
        with:
          script: |
            await github.rest.repos.createRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              tag_name: context.ref.replace('refs/tags/', ''),
              name: context.ref.replace('refs/tags/', ''),
              generate_release_notes: true,
            });
```

---

## Monorepo CI (Turborepo)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Build and Test
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4
        with:
          fetch-depth: 0  # Needed for turbo --filter

      - name: Setup Node.js
        uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: 22
          cache: 'npm'  # or pnpm/yarn

      - name: Install dependencies
        run: npm ci

      - name: Lint (affected)
        run: npx turbo run lint --filter='...[origin/main]'

      - name: Test (affected)
        run: npx turbo run test --filter='...[origin/main]'

      - name: Build (affected)
        run: npx turbo run build --filter='...[origin/main]'
```

---

## Monorepo CI (Nx)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    name: Build and Test (Affected)
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4
        with:
          fetch-depth: 0  # Needed for nx affected

      - name: Setup Node.js
        uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: 22
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Derive SHAs for nx affected
        uses: nrwl/nx-set-shas@<SHA>  # v4

      - name: Lint (affected)
        run: npx nx affected --target=lint

      - name: Test (affected)
        run: npx nx affected --target=test

      - name: Build (affected)
        run: npx nx affected --target=build
```

---

## Monorepo CI (Path Filter)

For monorepos without Nx or Turborepo, use path-based change detection.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: read  # Required by paths-filter for PRs

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  detect-changes:
    name: Detect Changes
    runs-on: ubuntu-latest
    timeout-minutes: 5
    outputs:
      api: ${{ steps.filter.outputs.api }}
      web: ${{ steps.filter.outputs.web }}
      shared: ${{ steps.filter.outputs.shared }}
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Detect changed paths
        uses: dorny/paths-filter@<SHA>  # v3
        id: filter
        with:
          filters: |
            api:
              - 'packages/api/**'
              - 'packages/shared/**'
            web:
              - 'packages/web/**'
              - 'packages/shared/**'
            shared:
              - 'packages/shared/**'

  test-api:
    name: Test API
    needs: detect-changes
    if: needs.detect-changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@<SHA>  # v4
      - uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: 22
          cache: 'npm'
      - run: npm ci
      - run: npm test --workspace=packages/api

  test-web:
    name: Test Web
    needs: detect-changes
    if: needs.detect-changes.outputs.web == 'true'
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@<SHA>  # v4
      - uses: actions/setup-node@<SHA>  # v4
        with:
          node-version: 22
          cache: 'npm'
      - run: npm ci
      - run: npm test --workspace=packages/web
```

---

## Scheduled Security Scan

```yaml
name: Security Scan

on:
  schedule:
    - cron: '0 6 * * 1'  # Weekly on Monday at 06:00 UTC
  workflow_dispatch:  # Allow manual trigger

permissions:
  contents: read
  security-events: write

jobs:
  codeql:
    name: CodeQL Analysis
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Checkout code
        uses: actions/checkout@<SHA>  # v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@<SHA>  # v3
        with:
          languages: javascript  # or python, go, java, etc.

      - name: Autobuild
        uses: github/codeql-action/autobuild@<SHA>  # v3

      - name: Run CodeQL Analysis
        uses: github/codeql-action/analyze@<SHA>  # v3
```

---

## Dependabot Configuration

This is not a workflow but a configuration file. Always recommend creating it.

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Keep GitHub Actions up to date
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      actions:
        patterns:
          - "*"

  # Uncomment and customize for your ecosystem:

  # Node.js
  # - package-ecosystem: "npm"
  #   directory: "/"
  #   schedule:
  #     interval: "weekly"
  #   groups:
  #     dev-dependencies:
  #       dependency-type: "development"

  # Python
  # - package-ecosystem: "pip"
  #   directory: "/"
  #   schedule:
  #     interval: "weekly"

  # Go
  # - package-ecosystem: "gomod"
  #   directory: "/"
  #   schedule:
  #     interval: "weekly"

  # Rust
  # - package-ecosystem: "cargo"
  #   directory: "/"
  #   schedule:
  #     interval: "weekly"

  # Docker
  # - package-ecosystem: "docker"
  #   directory: "/"
  #   schedule:
  #     interval: "weekly"

  # Terraform
  # - package-ecosystem: "terraform"
  #   directory: "/infra"
  #   schedule:
  #     interval: "weekly"
```
