# Document Templates

## Table of Contents
- [README.md Template](#readmemd-template)
- [Getting Started Template](#getting-started-template)
- [API Reference Template](#api-reference-template)
- [Architecture Document Template](#architecture-document-template)
- [Architecture Decision Record (ADR) Template](#architecture-decision-record-adr-template)
- [Developer Guide Template](#developer-guide-template)
- [Testing Guide Template](#testing-guide-template)
- [Deployment Guide Template](#deployment-guide-template)
- [Contributing Guide Template](#contributing-guide-template)
- [Changelog Template](#changelog-template)
- [Configuration Reference Template](#configuration-reference-template)

---

## README.md Template

The README is the project's front door. It is a hybrid document: not strictly
one Diataxis type, but a concise gateway that links to detailed docs.

```markdown
# Project Name

One-line description: what this project does and what problem it solves.

## Installation

[Copy-paste commands to install. Minimal steps.]

```bash
npm install project-name
```

## Quick Start

[Minimal runnable example. Show input and expected output.]

```bash
project-name init my-app
cd my-app
project-name start
# => Server running at http://localhost:3000
```

## Features

- [Feature 1]: [one-line description]
- [Feature 2]: [one-line description]
- [Feature 3]: [one-line description]

## Usage

[Core usage patterns with code examples. For CLI tools, show common commands.
For libraries, show primary API usage.]

```[language]
[code example derived from actual source]
```

## Architecture

[2-3 sentence summary of how the system is structured.
Link to docs/architecture.md for details if it exists.]

## Development

```bash
[clone, install, build, test commands]
```

[Link to docs/developer-guide.md for details if it exists.]

## Contributing

[One-line invitation. Link to CONTRIBUTING.md if it exists.]

## License

[License name. Link to LICENSE file.]
```

### README quality checklist

- [ ] Title clearly states what this project is
- [ ] Description answers "what" and "why" in one line
- [ ] Install commands are copy-pastable and current
- [ ] Quick start produces a visible result
- [ ] Every code example is derived from actual code
- [ ] No broken links
- [ ] License is stated

---

## Getting Started Template

**Diataxis type: Tutorial**

A guided learning experience for first-time users. Follow Tutorial rules
strictly: concrete steps, visible results, no choices.

```markdown
# Getting Started with [Project Name]

This tutorial walks you through [what the user will build/achieve].
By the end, you will have [concrete outcome].

## Prerequisites

- [Prerequisite 1 with version, e.g., "Node.js 18 or later"]
- [Prerequisite 2]

## Step 1: [Action verb + object]

[Instruction as imperative command.]

```bash
[exact command]
```

You should see output like:

```
[expected output]
```

## Step 2: [Action verb + object]

[Instruction.]

```bash
[exact command]
```

Notice that [point out something the learner should observe].

## Step 3: [Action verb + object]

[Instruction.]

[If the step produces visible output, show it.]

## What You Built

You have [summary of what was accomplished]. From here you can:

- [Next step or link to how-to guide]
- [Link to reference docs for deeper exploration]
```

### Tutorial rules reminder

- One path, no choices. Guide the learner through exactly one sequence.
- Every step produces a visible result.
- Do not explain concepts. Link to Explanation docs instead.
- Use "we" language: "First, we create..." or "Notice that..."
- Test every step. A broken tutorial destroys learner trust.

---

## API Reference Template

**Diataxis type: Reference**

Factual description of every public interface. Mirrors the code structure.
Austere, objective, no instruction.

### For HTTP APIs

```markdown
# API Reference

## Authentication

[Method: API key, Bearer token, OAuth2, etc.]
[Where to include it: header, query param, etc.]
[Example header:]

```
Authorization: Bearer <token>
```

## Base URL

```
https://api.example.com/v1
```

## Endpoints

### [Resource Name]

#### GET /[resource]

[One-line description of what this returns.]

**Parameters:**

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| [param] | query | string | No | [description] |
| [param] | query | integer | No | [description, default: value] |

**Response: 200 OK**

```json
{
  "[field]": "[type — description]",
  "[field]": "[type — description]"
}
```

**Example:**

```bash
curl -H "Authorization: Bearer $TOKEN" \
  https://api.example.com/v1/[resource]?[param]=value
```

```json
{
  "id": "abc-123",
  "name": "Example"
}
```

**Errors:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | INVALID_PARAM | [description] |
| 401 | UNAUTHORIZED | [description] |
| 404 | NOT_FOUND | [description] |

#### POST /[resource]

[One-line description.]

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| [field] | string | Yes | [description] |
| [field] | integer | No | [description, default: value] |

**Example Request:**

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"[field]": "value"}' \
  https://api.example.com/v1/[resource]
```

**Response: 201 Created**

```json
{
  "id": "new-id",
  "[field]": "value"
}
```
```

### For Library APIs

```markdown
# API Reference

## [Module/Class Name]

### `functionName(param1, param2)`

[One-line description.]

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| param1 | string | — | [description] |
| param2 | Options | {} | [description] |

**Returns:** `ReturnType` — [description]

**Throws:** `ErrorType` — [when condition]

**Example:**

```[language]
const result = functionName("input", { option: true });
// => expected output
```

### `ClassName`

[One-line description of what this class represents.]

#### `new ClassName(config)`

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| config | Config | [description] |

#### `instance.method(param)`

[One-line description.]

**Parameters:** [same format as above]
**Returns:** [type and description]
```

### API reference rules reminder

- Mirror the code structure (modules, classes, endpoints as they appear in code)
- Document EVERY public interface — completeness is mandatory
- Neutral, factual voice. No opinions or instruction.
- Include at least one example per endpoint/function
- Document both success and error cases
- Parameter tables must include: name, type, required/default, description

---

## Architecture Document Template

**Diataxis type: Explanation**

Provides context, reasoning, and the big picture. Answers "how does this
system work and why was it built this way?"

```markdown
# Architecture

## System Overview

[2-3 sentences: what this system does, who uses it, and its primary purpose.]

## System Context

[Describe the system's boundaries. What external systems and users interact
with it? Keep this non-technical enough for any stakeholder.]

### Users / Actors
- **[Actor name]**: [what they do with the system]

### External Systems
- **[System name]**: [what it provides to/receives from this system]

## Containers

[Major deployable units. For each: name, technology, responsibility.]

| Container | Technology | Responsibility |
|-----------|-----------|----------------|
| [Web App] | [React, Next.js, etc.] | [serves the user interface] |
| [API Server] | [Express, Django, etc.] | [handles business logic and API] |
| [Database] | [PostgreSQL, MongoDB, etc.] | [persists application data] |
| [Queue] | [Redis, RabbitMQ, etc.] | [handles async job processing] |

## Data Flow

[Describe how data moves through the system. Use numbered steps or a
text-based diagram.]

1. User sends request to [frontend]
2. [Frontend] calls [API endpoint]
3. [API] validates and processes request
4. [API] writes to [database]
5. [API] publishes event to [queue] (if async)
6. [Worker] processes event from [queue]

## Key Design Decisions

[For each significant decision, explain the context, the choice, and why.]

### [Decision Title]

**Context:** [What forces led to this decision?]
**Decision:** [What was chosen?]
**Rationale:** [Why this approach over alternatives?]
**Trade-offs:** [What was sacrificed? What risks remain?]

[For formal projects, use ADR files instead. See ADR template below.]

## Component Details

### [Component/Module Name]

**Responsibility:** [What this component does]
**Key files:** [Primary source files]
**Dependencies:** [What it depends on]
**Interfaces:** [What it exposes to other components]

## Security Considerations

[Authentication/authorization approach, data encryption, key management,
or link to a security doc.]

## Deployment

[Brief overview of how the system is deployed, or link to deployment guide.]
```

---

## Architecture Decision Record (ADR) Template

**Diataxis type: Explanation**

One ADR per decision. Store in `docs/adr/` with sequential numbering.
File naming: `NNN-short-noun-phrase.md` (e.g., `001-choose-database.md`).

```markdown
# ADR NNN: [Short Noun Phrase Title]

## Status

[Proposed | Accepted | Deprecated | Superseded by ADR-NNN]

## Date

[YYYY-MM-DD]

## Context

[Describe the forces at play: technical constraints, business requirements,
team capabilities, timeline pressure. Write in full sentences, value-neutral.
The reader is a future developer with no current context.]

## Decision

[State the decision in active voice: "We will use PostgreSQL as the primary
datastore." "We will implement authentication via JWTs stored in HTTP-only
cookies."]

## Consequences

[All effects — positive, negative, and neutral. Be honest about trade-offs.]

### Positive
- [benefit]

### Negative
- [cost or risk]

### Neutral
- [side effect that is neither clearly good nor bad]
```

### ADR rules reminder

- One decision per ADR. Never bundle.
- Context is value-neutral (describe forces, not opinions).
- Decision uses active voice ("We will...").
- Consequences include BOTH positive and negative effects.
- Never delete ADRs. Mark superseded ones and link to replacement.
- Keep to 1-2 pages maximum.
- Number sequentially. Never reuse numbers.

---

## Developer Guide Template

**Diataxis type: How-to**

Practical guide for developers working on the project. Action-oriented,
assumes the reader is competent and knows what they want to do.

```markdown
# Developer Guide

## Prerequisites

- [Tool 1]: [version, install link]
- [Tool 2]: [version, install link]

## Setup

```bash
git clone [repo-url]
cd [project-name]
[install command]
[setup command, e.g., database migration]
```

## Running Locally

```bash
[start command]
# => [expected output, e.g., "Server at http://localhost:3000"]
```

## Project Structure

```
[directory tree of key folders only]
src/
  [module]/     # [one-line purpose]
  [module]/     # [one-line purpose]
tests/          # [test framework, run command]
config/         # [what config lives here]
```

## Common Tasks

### Adding a New [Entity/Endpoint/Component]

1. [Step with file path]
2. [Step with command]
3. [Step to verify]

### Running Tests

```bash
[test command]
[coverage command, if applicable]
```

### Debugging

[How to attach a debugger, view logs, or enable verbose output.]

## Code Conventions

- **Naming:** [file naming, function naming, variable casing]
- **Imports:** [import ordering or style]
- **Error handling:** [pattern used]
- **Commits:** [commit message format, e.g., Conventional Commits]

## Build and Release

```bash
[build command]
[release/deploy command or link to deployment guide]
```
```

---

## Testing Guide Template

**Diataxis type: How-to**

```markdown
# Testing Guide

## Running Tests

```bash
# Run all tests
[full test command]

# Run a specific test file
[specific test command]

# Run tests with coverage
[coverage command]
```

## Test Structure

```
tests/
  unit/         # [what's tested here]
  integration/  # [what's tested here]
  e2e/          # [what's tested here]
```

## Writing Tests

### Test file naming
[Convention: `*.test.ts`, `test_*.py`, etc.]

### Test structure
[Framework-specific pattern: describe/it, class-based, function-based]

### Fixtures and helpers
[Where shared test utilities live, how to use them]

## CI Integration

[How tests run in CI. Link to CI config file.]
```

---

## Deployment Guide Template

**Diataxis type: How-to**

```markdown
# Deployment Guide

## Prerequisites

- [Infrastructure requirement]
- [Access/credentials needed]

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| [VAR_NAME] | Yes | — | [description] |
| [VAR_NAME] | No | [default] | [description] |

## Deploy to [Environment]

```bash
[step-by-step deployment commands]
```

## Verify Deployment

```bash
[health check command or URL]
```

## Rollback

```bash
[rollback command or procedure]
```

## Monitoring

[Where to view logs, metrics, alerts.]
```

---

## Contributing Guide Template

**Diataxis type: How-to**

```markdown
# Contributing to [Project Name]

## How to Contribute

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Run tests: `[test command]`
5. Commit with a descriptive message
6. Open a pull request

## Development Setup

[Link to developer guide or brief setup instructions.]

## Code Style

[Linter, formatter, and style requirements. Link to config files.]

## Pull Request Process

- [PR description requirements]
- [Review process]
- [CI checks that must pass]

## Reporting Issues

[How to file a bug report or feature request. Link to issue templates.]

## Code of Conduct

[Link to CODE_OF_CONDUCT.md or brief statement.]
```

---

## Changelog Template

**Diataxis type: Reference**

Follow the [Keep a Changelog](https://keepachangelog.com/) standard.

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- [new feature description]

### Changed
- [modification to existing feature]

### Fixed
- [bug fix description]

## [1.0.0] - YYYY-MM-DD

### Added
- [feature description]
- [feature description]

### Changed
- [change description]

### Deprecated
- [feature scheduled for removal]

### Removed
- [feature that was removed]

### Fixed
- [bug fix]

### Security
- [vulnerability fix]
```

### Changelog rules

- **Six categories only:** Added, Changed, Deprecated, Removed, Fixed, Security
- **Reverse chronological** order (newest first)
- **ISO 8601 dates** (YYYY-MM-DD)
- **Unreleased section** at top for upcoming changes
- **Write for humans.** Describe the user-visible effect, not the implementation.
- **Never use raw git log.** Curate and summarize.

---

## Configuration Reference Template

**Diataxis type: Reference**

```markdown
# Configuration Reference

## Environment Variables

| Variable | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| [VAR] | string | Yes | — | [what it controls] |
| [VAR] | integer | No | [default] | [what it controls] |
| [VAR] | boolean | No | false | [what it controls] |

## Configuration File

**Location:** `[path/to/config.yaml]`

```yaml
[annotated example config with all options]
```

### [Section Name]

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| [key] | [type] | [default] | [description] |
```
