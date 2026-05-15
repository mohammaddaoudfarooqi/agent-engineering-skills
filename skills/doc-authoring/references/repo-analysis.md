# Repository Analysis Method

Systematic analysis of a codebase to determine what documentation is needed,
what facts to document, and what already exists. Run this before writing
any documentation.

## Analysis Sequence

Follow these steps in order. Each step informs the next.

---

## Step 1: Language Detection

Identify the primary language(s) by scanning for manifest files.

| Manifest File | Language/Platform |
|---------------|------------------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `pyproject.toml`, `setup.py`, `setup.cfg` | Python |
| `requirements.txt`, `Pipfile` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml`, `build.gradle`, `build.gradle.kts` | Java / Kotlin |
| `Gemfile`, `*.gemspec` | Ruby |
| `composer.json` | PHP |
| `*.csproj`, `*.sln` | C# / .NET |
| `Package.swift` | Swift |
| `mix.exs` | Elixir |
| `Makefile` (alone) | C / C++ (check for CMakeLists.txt too) |
| `CMakeLists.txt` | C / C++ |
| `deno.json`, `deno.jsonc` | Deno / TypeScript |

**Record:** Language name, version (from manifest if specified), runtime.

---

## Step 2: Framework and Library Detection

Based on the detected language, scan dependency files for framework signatures.

### Node.js / TypeScript

| Dependency | Framework |
|-----------|-----------|
| `express` | Express.js |
| `fastify` | Fastify |
| `@nestjs/core` | NestJS |
| `koa` | Koa |
| `hapi`, `@hapi/hapi` | Hapi |
| `next` | Next.js |
| `react` | React |
| `vue` | Vue.js |
| `@angular/core` | Angular |
| `svelte` | Svelte |
| `electron` | Electron |

### Python

| Dependency | Framework |
|-----------|-----------|
| `django` | Django |
| `flask` | Flask |
| `fastapi` | FastAPI |
| `starlette` | Starlette |
| `tornado` | Tornado |
| `celery` | Celery (task queue) |
| `click`, `typer` | CLI framework |
| `argparse` | CLI (stdlib) |

### Go

| Import Path | Framework |
|------------|-----------|
| `net/http` | stdlib HTTP |
| `github.com/gin-gonic/gin` | Gin |
| `github.com/gorilla/mux` | Gorilla Mux |
| `github.com/labstack/echo` | Echo |
| `github.com/gofiber/fiber` | Fiber |
| `github.com/spf13/cobra` | Cobra (CLI) |

### Java / Kotlin

| Dependency | Framework |
|-----------|-----------|
| `spring-boot` | Spring Boot |
| `quarkus` | Quarkus |
| `micronaut` | Micronaut |
| `vert.x` | Vert.x |

### Ruby

| Dependency | Framework |
|-----------|-----------|
| `rails` | Ruby on Rails |
| `sinatra` | Sinatra |
| `hanami` | Hanami |

**Record:** Framework name, version, and any variant-specific patterns
that affect documentation (e.g., NestJS uses decorators; Express uses
middleware chains).

---

## Step 3: Architecture Detection

Scan for structural indicators.

### Project type indicators

| Indicator | Project Type |
|-----------|-------------|
| Single `src/` directory, no services | Monolith or library |
| `docker-compose.yml` with multiple services | Multi-service system |
| `lerna.json`, `nx.json`, `turbo.json`, `pnpm-workspace.yaml` | Monorepo |
| `lib/` or `src/` with `index` exports, no server | Library/package |
| CLI framework dependency + `bin/` | CLI tool |
| `Dockerfile` only (no compose) | Single deployable service |
| `serverless.yml`, `template.yaml` (SAM) | Serverless |
| `k8s/`, `helm/`, Kubernetes manifests | Kubernetes-deployed |

### For multi-service / monorepo projects

Scan each service directory independently and classify:

| Classification | Indicators |
|---------------|-----------|
| Frontend | React/Vue/Angular/Svelte, static assets, HTML templates |
| Backend API | HTTP framework, route definitions, controllers |
| Worker/Queue | Celery, Bull, Sidekiq, message queue consumers |
| Database | Migration files, ORM config, schema definitions |
| Gateway | Nginx config, API gateway config, proxy setup |

**Record:** Architecture type, service names and roles (if multi-service),
monorepo structure (if applicable).

---

## Step 4: Entry Point Detection

Find how the application starts and what interfaces it exposes.

| Entry Point Type | How to Find |
|-----------------|-------------|
| CLI | `bin/` scripts, `"bin"` in package.json, `console_scripts` in setup.cfg/pyproject.toml, `main` package in Go, `if __name__ == "__main__"` |
| HTTP server | Framework startup calls (`app.listen()`, `uvicorn.run()`, `http.ListenAndServe()`) |
| Lambda/serverless | Handler exports matching cloud provider patterns |
| Worker | Queue consumer startup, daemon process setup |
| Library | Package exports (`index.ts`, `__init__.py`, `lib.rs`) |
| Scripts | `"scripts"` in package.json, Makefile targets, shell scripts |

**Record:** Entry point files, startup commands, exposed interfaces (HTTP
ports, CLI commands, library exports).

---

## Step 5: Public Interface Extraction

Identify all interfaces that external users or consumers interact with.

### HTTP APIs

- Scan for route definitions (`app.get()`, `@app.route()`, `@GetMapping`, etc.)
- Check for OpenAPI/Swagger specs (`openapi.yaml`, `swagger.json`)
- Check for GraphQL schemas (`schema.graphql`, `typeDefs`)
- Note authentication mechanisms (middleware, decorators, guards)

### Library APIs

- Read package exports (index files, `__all__`, `pub` in Rust, `export` in TS)
- Identify public classes, functions, types
- Check for existing JSDoc, docstrings, or doc comments

### CLI interfaces

- Parse CLI framework definitions (commands, flags, arguments)
- Check for `--help` output structure
- Note subcommand hierarchy

**Record:** List of all public interfaces with their signatures, parameters,
and return types.

---

## Step 6: Configuration Extraction

Collect all configuration that users or operators need to know about.

### Environment variables

Scan for environment variable access patterns:
- JavaScript/TypeScript: `process.env.VAR_NAME`
- Python: `os.environ`, `os.getenv()`, settings modules
- Go: `os.Getenv()`, `viper` config
- Java: `@Value`, `System.getenv()`
- Ruby: `ENV['VAR_NAME']`

### Config files

- `.env`, `.env.example`, `.env.sample`
- `config/`, `settings/` directories
- YAML/JSON/TOML config files
- Docker Compose environment sections

**Record:** For each config item: name, type, required/optional, default
value (if detectable), and purpose (inferred from usage context).

---

## Step 7: Database and Data Model Detection

| Indicator | Technology |
|-----------|-----------|
| `migrations/`, `alembic/` | SQL database with migrations |
| Sequelize, TypeORM, Prisma, SQLAlchemy models | ORM-defined schemas |
| `schema.prisma` | Prisma schema |
| `*.sql` files | Raw SQL schemas |
| `mongoose` models | MongoDB via Mongoose |
| Redis client imports | Redis cache/store |

**Record:** Database type, ORM (if any), key entities/tables, relationships.

---

## Step 8: Test Infrastructure Detection

| Indicator | Test Framework |
|-----------|---------------|
| `jest.config.*`, `@jest` imports | Jest |
| `vitest.config.*`, `vitest` imports | Vitest |
| `pytest.ini`, `conftest.py`, `pytest` imports | pytest |
| `mocha`, `.mocharc.*` | Mocha |
| `*_test.go` files | Go testing |
| `rspec`, `spec/` directory (Ruby) | RSpec |
| `@Test` annotations (Java) | JUnit |
| `cargo test`, `#[test]` | Rust built-in |

Also check:
- CI/CD config for test commands (GitHub Actions, GitLab CI, CircleCI)
- Coverage tools (istanbul, coverage.py, go cover)
- E2E test frameworks (Playwright, Cypress, Selenium)

**Record:** Test framework, runner command, test directory, approximate
test count, CI config file.

---

## Step 9: Existing Documentation Audit

Catalog what documentation already exists and assess its state.

### Files to check

| File/Location | Purpose |
|---------------|---------|
| `README.md` | Project overview |
| `CONTRIBUTING.md` | Contribution guide |
| `CHANGELOG.md` | Version history |
| `LICENSE` | License |
| `docs/` directory | Extended documentation |
| `CLAUDE.md` | AI agent instructions |
| `.github/` | Issue/PR templates, workflows |
| Code comments | Inline documentation |
| Docstrings | Function/class documentation |
| OpenAPI/Swagger specs | API documentation |
| `man/` pages | CLI documentation |

### For each existing doc, assess

- **Accuracy**: Does it match current code? (spot-check 3-5 claims)
- **Completeness**: Does it cover all relevant interfaces?
- **Freshness**: When was it last updated? Does it reference current versions?
- **Type purity**: Is it cleanly one Diataxis type, or does it mix?
- **Quality**: Is it clear, concise, and well-structured?

**Record:** List of existing docs with accuracy/completeness assessment.
Note gaps (what docs are missing) and drift (what docs are stale).

---

## Step 10: Compile Repository Profile

Combine all findings into a structured profile.

```yaml
repository_profile:
  language: [language, version]
  framework: [framework, version]
  architecture: [type: monolith|multi-service|library|cli|serverless]
  services:  # if multi-service
    - name: [service]
      role: [frontend|api|worker|database|gateway]
      technology: [framework]
  entry_points:
    - type: [http|cli|library|worker]
      file: [path]
      command: [startup command]
  public_interfaces:
    http_endpoints: [count, list if small]
    library_exports: [count, key modules]
    cli_commands: [count, list]
  configuration:
    env_vars: [count, list of names]
    config_files: [list of paths]
  data:
    database: [type]
    orm: [name]
    key_entities: [list]
  tests:
    framework: [name]
    runner: [command]
    directory: [path]
    ci_config: [path]
  existing_docs:
    present: [list of existing doc files]
    missing: [list of docs that should exist but don't]
    stale: [list of docs that are outdated]
    doc_infrastructure: [generators, linters, CI checks]
```

This profile drives the documentation plan in Phase 1.

---

## Confidence Scoring

Not all findings are equally certain. Flag uncertainty:

| Confidence | Meaning | Action |
|-----------|---------|--------|
| **High** | Confirmed from manifest/config | Document directly |
| **Medium** | Inferred from code patterns | Document with verification |
| **Low** | Guessed from directory structure | Ask user to confirm |

When confidence is Low for a critical fact (e.g., how to start the application,
what the primary database is), ask the user before documenting.
