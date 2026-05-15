# Drift Detection Method

Systematic section-by-section comparison of existing documentation against
current source code to identify where docs have diverged from reality. Use
this during the Doc Refresh and Version Update workflows.

## When to Use

- Code has changed but docs haven't been updated
- Version bump with potential breaking changes
- Periodic doc health check
- Before any brownfield doc modification (to understand current state)

---

## Step 1: Categorize Doc Sections

Read the existing doc and categorize each section by what kind of fact it
asserts. Different fact types require different verification methods.

| Fact Type | Example | Verification Method |
|-----------|---------|-------------------|
| **Version claim** | "Requires Node.js 18+" | Check manifest files for engine/dependency versions |
| **Install command** | `npm install my-package` | Check package name, registry, install method |
| **Startup command** | `npm start` | Check scripts in manifest, Makefile, or entry points |
| **API endpoint** | `GET /api/users` | Grep for route definition in source |
| **Function signature** | `createUser(name, email)` | Read the actual function definition |
| **Parameter list** | "Accepts `timeout` (int, default: 30)" | Read function signature + default values |
| **Return type** | "Returns a `User` object" | Read function return type / return statements |
| **Env var reference** | `DATABASE_URL` | Grep for env var access in source |
| **Config option** | "`max_retries` in config.yaml" | Read config file / config schema |
| **File path reference** | "Edit `src/config/database.ts`" | Check that the file exists at that path |
| **Behavior description** | "Retries 3 times on failure" | Read implementation or test for the behavior |
| **Architecture claim** | "Uses Redis for caching" | Check dependencies and infrastructure config |
| **Link / cross-reference** | "[See API docs](docs/api.md)" | Check that target file and heading exist |

---

## Step 2: Verify Each Fact

For each section of the document, verify its facts using the method
from the table above.

### Verification procedure

```
For each section in the document:
  1. Identify all factual claims (commands, names, paths, behaviors)
  2. For each claim:
     a. Locate the source of truth in the codebase
     b. Compare the documented claim against the source
     c. Classify the result:
        - ACCURATE: Claim matches code
        - STALE: Claim was once true but code has changed
        - WRONG: Claim never matched code (fabrication or error)
        - UNVERIFIABLE: Cannot confirm from code alone (ask user)
  3. For the section overall:
     - If all claims ACCURATE → mark section ACCURATE
     - If any claim STALE or WRONG → mark section NEEDS UPDATE
     - If section describes something that no longer exists → mark DEAD
```

### Priority verification targets

Not all facts are equally likely to drift. Prioritize verification of
these high-drift-risk items:

| High Drift Risk | Why |
|----------------|-----|
| Version numbers | Dependencies update frequently |
| Install commands | Package names, registries, and methods change |
| API endpoints and parameters | APIs evolve with features |
| Configuration options | Config keys get renamed, added, removed |
| File paths | Refactoring moves files |
| CLI flags and commands | CLI interfaces change across versions |
| Environment variables | Env vars get renamed or deprecated |
| Port numbers | Default ports change |

| Low Drift Risk | Why |
|----------------|-----|
| Project description / purpose | Core mission rarely changes |
| Architecture overview | High-level architecture is stable |
| Design rationale | Past decisions don't change |
| License | Rarely changes |
| Contributing guidelines (process) | Contribution process is stable |

Start with high-drift-risk items. If those are accurate, the rest likely is too.

---

## Step 3: Check for Missing Coverage

After verifying existing content, check for things the code has that the
docs don't cover.

### Missing coverage detection

```
For each public interface found in Phase 0 (repo analysis):
  1. Search the existing docs for a reference to it
  2. If not found → mark as MISSING

For each env var found in Phase 0:
  1. Search the existing docs for a reference to it
  2. If not found → mark as MISSING

For each config file/option found in Phase 0:
  1. Search the existing docs for a reference to it
  2. If not found → mark as MISSING
```

### Common sources of missing coverage

| Source | What's Missing |
|--------|---------------|
| New API endpoints | Endpoints added since docs were last updated |
| New CLI commands/flags | Commands added without doc updates |
| New env vars | Config added for new features |
| New dependencies | External services not mentioned in setup docs |
| New error codes | Error responses not documented |
| Renamed exports | Old name documented, new name undocumented |

---

## Step 4: Detect Dead Content

Identify documentation that describes things that no longer exist.

### Dead content detection

```
For each documented API endpoint:
  Grep for route definition → if not found → DEAD

For each documented function/class:
  Grep for definition → if not found → DEAD

For each documented env var:
  Grep for env var access → if not found → DEAD

For each documented file path:
  Check file exists → if not found → DEAD

For each internal link:
  Check target exists → if broken → DEAD LINK

For each documented CLI command:
  Check CLI definition → if not found → DEAD
```

### Handling dead content

| Dead Content Type | Action |
|------------------|--------|
| Entire section about removed feature | Mark for removal (confirm with user) |
| Single reference to renamed item | Update to new name |
| Link to moved file | Update link to new location |
| Link to deleted file | Remove link, check if content moved elsewhere |
| Deprecated API endpoint | Mark as deprecated (do not delete — consumers may reference it) |
| Example using removed function | Rewrite example with current equivalent |

---

## Step 5: Compile Drift Report

Combine all findings into a structured report for each doc file.

### Drift report format

```
Drift Report: [file path]
Last meaningful code change: [approximate date from git log if available]

ACCURATE sections (no changes needed):
  - [Section heading]: [brief note]
  - [Section heading]: [brief note]

STALE facts (need updating):
  - [Section heading]: [documented fact] → [current fact in code]
  - [Section heading]: [documented fact] → [current fact in code]

DEAD content (references things that no longer exist):
  - [Section heading]: [what it references] → [no longer exists because...]

MISSING coverage (code exists but docs don't cover it):
  - [Interface/feature]: [what exists in code but not in docs]

UNVERIFIABLE claims (cannot confirm from code):
  - [Section heading]: [claim] → [ask user to confirm]

Severity summary:
  Critical (wrong facts): [count]
  High (missing coverage): [count]
  Medium (stale but not dangerous): [count]
  Low (style/structure): [count]
```

### Severity classification

| Severity | Criteria | Examples |
|----------|---------|---------|
| **Critical** | Doc states something factually wrong that would cause errors if followed | Wrong install command, wrong API endpoint, wrong env var name |
| **High** | Public interface exists but has no documentation | New endpoint, new CLI command, new config option |
| **Medium** | Doc is outdated but not dangerous to follow | Old version number, renamed but redirected path, stale screenshot |
| **Low** | Cosmetic or structural issue | Inconsistent formatting, mixed Diataxis types, stale "last updated" date |

---

## Step 6: Plan Fixes from Drift Report

Convert the drift report into a concrete edit plan.

### Fix priority order

1. **Critical fixes first.** Wrong facts can cause user errors. Fix these
   before anything else.
2. **Dead content removal.** Remove references to things that don't exist.
   (Confirm with user before deleting entire sections.)
3. **High-priority additions.** Add documentation for undocumented public
   interfaces.
4. **Medium staleness updates.** Update version numbers, renamed items, etc.
5. **Low-priority improvements.** Style fixes, restructuring. Only if user
   requested or scope allows.

### Edit plan format

```
Edit Plan: [file path]
  1. [CRITICAL] Line ~[N]: Replace "[old install cmd]" with "[new install cmd]"
  2. [CRITICAL] Line ~[N]: Update endpoint from "/api/v1/users" to "/api/v2/users"
  3. [HIGH] After section "[X]": Add documentation for POST /api/widgets
  4. [DEAD] Remove section "[Y]" (references removed feature Z) — CONFIRM WITH USER
  5. [MEDIUM] Line ~[N]: Update version "3.1" to "4.0"
```

---

## Version Update: Special Considerations

When drift is caused by a version bump rather than gradual code evolution,
apply these additional checks:

### Version-specific drift patterns

| Change Type | Where to Look in Code | What to Update in Docs |
|-------------|----------------------|----------------------|
| **Breaking API changes** | Changelog, git diff of route files | API reference, migration guide |
| **Renamed config keys** | Git diff of config files/schemas | Config reference, deployment guide, README |
| **Changed defaults** | Git diff of default values | Config reference, getting started |
| **Removed features** | Changelog, git diff of exports | All docs referencing the feature |
| **New required dependencies** | Manifest file diff | Prerequisites, install guide |
| **Changed CLI interface** | Git diff of CLI definitions | Usage docs, README quick start |
| **Database migrations** | Migration files | Data model docs, upgrade guide |

### Migration documentation

For significant version bumps, recommend creating a migration guide:

```markdown
# Migrating from v[OLD] to v[NEW]

## Breaking Changes

### [Change 1 title]
**Before (v[OLD]):** [old behavior/syntax]
**After (v[NEW]):** [new behavior/syntax]
**Migration:** [what the user needs to do]

### [Change 2 title]
...

## Deprecated Features

### [Feature name]
**Status:** Deprecated in v[NEW], removal planned for v[NEXT]
**Alternative:** [what to use instead]

## New Features

- [Feature]: [one-line description, link to docs]
```
