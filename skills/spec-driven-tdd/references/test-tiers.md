# Test Tiers

Five tiers cover both axes of verification (coverage and realism). Each
tier crosses a different boundary and catches a different class of
defect. This file gives definitions and per-stack examples.

| Tier | Boundary crossed | Typical runtime | Catches |
| --- | --- | --- | --- |
| **unit** | None — pure in-process | < 10ms each | Logic, branches, edge cases |
| **integration** | Module-to-module, optionally with fakes | 10-500ms each | Wiring, contracts between in-process components |
| **smoke / functional** | The deployable artifact, real transport | seconds | Integration-shaped bugs the pyramid misses |
| **quality_gate** | Static analysis at compile / lint time | seconds | Bug classes preventable without running code |
| **characterization** | Documents existing behaviour for brownfield | varies | Silent regressions during refactor |

The **smoke / functional** tier is non-negotiable for any system with
I/O or multiple components. The other tiers are well-known; the rest
of this file focuses on what makes a smoke test a smoke test.

## What qualifies as a smoke / functional test

A smoke test must do all four of:

1. **Start the deployable artifact** via its real entrypoint.
2. **Hit it over real transport** — real HTTP, real SSE, real WebSocket,
   real CLI process boundary, real browser.
3. **Assert user-visible outcomes** — content of a returned report,
   text in the rendered DOM, contents of a downloaded file, exit code
   of a CLI, log lines from a background job.
4. **Be runnable in CI on every commit**, even if behind a tag.

A test that uses `httpx.AsyncClient(transport=ASGITransport(app=app))`
to call into the FastAPI app in-process is **not** a smoke test — it
never crosses the uvicorn boundary. Use it for fast integration checks,
but require a parallel smoke test that runs against `uvicorn` over a
real socket.

## Per-stack patterns

### Python — FastAPI / Starlette / Litestar / Flask backend

```python
# tests/smoke/test_sessions_journey.py
import subprocess, time, httpx, pytest

@pytest.fixture(scope="session")
def server():
    proc = subprocess.Popen(
        ["uvicorn", "myapp.main:app", "--port", "8765"],
        env={**os.environ, "ENV": "test"},
    )
    for _ in range(50):
        try:
            httpx.get("http://127.0.0.1:8765/healthz", timeout=0.2)
            break
        except httpx.ConnectError:
            time.sleep(0.1)
    yield "http://127.0.0.1:8765"
    proc.terminate()
    proc.wait(timeout=5)

def test_create_session_returns_a_real_report(server):
    r = httpx.post(f"{server}/api/sessions", json={"topic": "atlas"})
    assert r.status_code == 201   # asserts the contract, not the framework default
    sid = r.json()["id"]
    # Poll for the actual outcome — content of a report, not just a row insert
    for _ in range(30):
        r = httpx.get(f"{server}/api/sessions/{sid}")
        if r.json().get("status") == "complete":
            break
        time.sleep(1)
    body = r.json()
    assert body["status"] == "complete"
    assert "atlas" in body["report"]["content"].lower()
    assert len(body["report"]["content"]) > 500
```

Tools: `uvicorn` / `gunicorn`, `httpx`, `pytest-asyncio`, `testcontainers`,
`websocat` for SSE/WS, `playwright-cli` if there's a browser UI.

### Node / TypeScript — Next.js / Express / Fastify / Hono backend

```typescript
// tests/smoke/sessions-journey.test.ts
import { spawn } from "node:child_process";
import { setTimeout as wait } from "node:timers/promises";

let proc: ReturnType<typeof spawn>;

beforeAll(async () => {
  proc = spawn("node", ["dist/server.js"], {
    env: { ...process.env, PORT: "8765", NODE_ENV: "test" },
  });
  for (let i = 0; i < 50; i++) {
    try {
      const r = await fetch("http://127.0.0.1:8765/healthz");
      if (r.ok) break;
    } catch {}
    await wait(100);
  }
});

afterAll(() => proc.kill());

test("POST /api/sessions returns a complete report", async () => {
  const created = await fetch("http://127.0.0.1:8765/api/sessions", {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ topic: "atlas" }),
  });
  expect(created.status).toBe(201);
  const { id } = await created.json();

  for (let i = 0; i < 30; i++) {
    const r = await fetch(`http://127.0.0.1:8765/api/sessions/${id}`);
    const body = await r.json();
    if (body.status === "complete") {
      expect(body.report.content.toLowerCase()).toContain("atlas");
      expect(body.report.content.length).toBeGreaterThan(500);
      return;
    }
    await wait(1000);
  }
  throw new Error("session never completed");
});
```

### Frontend — React / Vue / Svelte SPA

Use [Playwright](https://playwright.dev) (or Cypress / WebdriverIO)
running against a **live frontend + live backend**. In-process React
testing tools (RTL, Enzyme) are unit tests, not smoke tests.

```typescript
// apps/web/tests/smoke/dashboard.spec.ts
import { test, expect } from "@playwright/test";

test("user creates a session and sees the report", async ({ page }) => {
  await page.goto("http://127.0.0.1:5173/");
  await page.getByRole("textbox", { name: "Topic" }).fill("atlas");
  await page.getByRole("button", { name: "Generate" }).click();
  // Wait for the SSE-driven UI update; not a network spy
  await expect(page.getByTestId("report")).toContainText("atlas", {
    timeout: 60_000,
  });
});
```

For exploratory verification at phase boundaries, prefer
[`playwright-cli`](demo-tools.md#playwright-cli) — refs+snapshot model
gives a 4-command verification script faster than writing a full
Playwright test.

### Go — net/http or chi or echo backend

```go
// internal/smoke/sessions_test.go
func TestCreateSessionReturnsRealReport(t *testing.T) {
    cmd := exec.Command("./bin/server")
    cmd.Env = append(os.Environ(), "PORT=8765")
    require.NoError(t, cmd.Start())
    t.Cleanup(func() { _ = cmd.Process.Signal(syscall.SIGTERM) })
    waitForHealth(t, "http://127.0.0.1:8765/healthz")

    resp, err := http.Post(
        "http://127.0.0.1:8765/api/sessions",
        "application/json",
        strings.NewReader(`{"topic":"atlas"}`),
    )
    require.NoError(t, err)
    require.Equal(t, 201, resp.StatusCode)
    // ...
}
```

### Rust — axum / actix backend

Use `tokio::process::Command` to start the binary, then call it with
`reqwest`. Same shape as the Go example.

### CLI tool (any language)

```bash
# tests/smoke/cli_smoke.sh
set -euo pipefail
out="$(./bin/mytool render --input fixtures/sample.json)"
[[ "$out" == *"expected substring"*  ]] || { echo "smoke fail"; exit 1; }
```

Or use `pytest` / `testify` with `subprocess` / `os/exec` to call the
real CLI binary.

### Background job / queue worker

Smoke test enqueues a job into a real queue (testcontainers Redis,
local SQS via LocalStack, etc.), starts the worker, and asserts the
job's externally-visible outcome (a row inserted, an email sent to a
catch-all, a file written).

## Boundaries → tier mapping

| Boundary | Recommended tier |
| --- | --- |
| HTTP entry (external client → server) | smoke / functional |
| Sync→Async transition (route → async DB driver) | mock-parity contract test (under integration) + smoke |
| Frontend → backend | smoke (full Playwright) for the journey; integration for individual API contracts |
| In-process module → out-of-process worker | smoke (start both processes) |
| Persisted state (repo → real DB) | integration with testcontainers; the smoke tier exercises this incidentally |
| HTTP → SSE / WebSocket | smoke with `websocat` or Playwright `page.waitForEvent` |
| JSON ↔ object | unit test on the serializer plus assertions in smoke |

## Floor: minimum smoke coverage

For the spec to pass Phase 6 verification:

- ≥ 1 smoke test per top-level user story in `requirements.md`.
- ≥ 1 smoke test per critical user journey (a sequence of stories the
  user actually runs in production).
- For systems with a frontend, ≥ 1 Playwright (or equivalent)
  smoke test that crosses the FE↔BE boundary.

## Anti-patterns

| Anti-pattern | Why it isn't a smoke test |
| --- | --- |
| `httpx.AsyncClient(transport=ASGITransport(app=app))` | In-process; never crosses uvicorn or the network |
| `supertest(app)` against an Express handler imported into the test | Same — in-process |
| React Testing Library rendering the SPA without a real backend | Doesn't cross FE↔BE |
| Asserting `mock_db.insert_one.assert_called_once()` | Side-effect proxy, not outcome |
| One test that calls 20 endpoints in sequence | Hard to debug; split into per-journey tests |
