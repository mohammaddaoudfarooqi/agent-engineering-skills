# Demo Tools by Stack

The phase-end **Demo gate** (Phase 5, step 6) requires evidence that
the new behaviour works against the running deployable artifact, not
just in tests. This file lists the recommended tools per stack.

The demo evidence pasted into `tasks.md` should be small — a few
command lines and their output, a screenshot, a short transcript. It is
not a load test or a comprehensive QA pass. It is *"I started the
thing, exercised the new behaviour, and saw the expected result."*

## HTTP backend

### curl

The lingua franca. Available everywhere; output pastes cleanly into
markdown.

```bash
$ curl -sS -X POST http://127.0.0.1:8000/api/sessions \
    -H 'Content-Type: application/json' \
    -d '{"topic":"atlas"}'
{"id":"01J9X7","status":"created","report_url":"/api/sessions/01J9X7"}

$ curl -sS http://127.0.0.1:8000/api/sessions/01J9X7
{"id":"01J9X7","status":"complete","report":{"content":"Atlas is..."}}
```

Conventions for paste-friendly output:
- `-sS` (silent + show errors) gives clean output.
- `-i` adds headers; useful when the demo proves a contract like 201 vs 200.
- `-w '\n%{http_code}\n'` appends the status code on a new line.

### httpie

Friendlier output for slides. Good when the demo target is a human reviewer.

```bash
$ http POST :8000/api/sessions topic=atlas
HTTP/1.1 201 Created
Content-Type: application/json

{ "id": "01J9X7", "status": "created", ... }
```

### grpcurl (gRPC)

```bash
$ grpcurl -plaintext -d '{"topic":"atlas"}' \
    localhost:9000 polymath.SessionService/Create
{ "id": "01J9X7", "status": "STATUS_CREATED" }
```

## Browser / frontend

### `playwright-cli`

Recommended for **exploratory phase-end demos** on UI work. Refs +
snapshot model means a 4-command verification script: navigate, click,
snapshot, screenshot. Faster than writing a full Playwright test for
exploration.

```bash
$ playwright-cli navigate http://127.0.0.1:5173/
$ playwright-cli click 'role=button[name="Generate"]' --fill 'topic=atlas'
$ playwright-cli wait-for 'data-testid=report'
$ playwright-cli snapshot --output demo.png
```

Use `playwright-cli` when:
- A frontend route or component is under verification.
- You need a screenshot or DOM snapshot to paste into `tasks.md`.
- Writing a full `playwright test` would be overkill (e.g. one-time
  visual confirmation of a phase boundary).

For tests that should **run on every CI**, write a real Playwright
spec under `tests/e2e/` or `apps/web/tests/smoke/` (see
[test-tiers.md](test-tiers.md)). `playwright-cli` is for the
exploratory demo, not for the regression suite.

### Playwright (full)

```typescript
test("user creates a session and sees the report", async ({ page }) => {
  await page.goto("http://127.0.0.1:5173/");
  await page.getByRole("textbox", { name: "Topic" }).fill("atlas");
  await page.getByRole("button", { name: "Generate" }).click();
  await expect(page.getByTestId("report")).toContainText("atlas");
});
```

### Cypress / WebdriverIO

Equivalent capability if your project already uses them. Cypress's run
mode produces a video that pastes well into a phase-end demo.

## Streaming — SSE / WebSockets

### websocat

Connects to WebSocket / SSE / pipes / TCP from the command line.

```bash
$ websocat -t 'http://127.0.0.1:8000/api/sessions/01J9X7/events'
event: session_start
data: {"id":"01J9X7"}

event: stage_progress
data: {"stage":"compose_report","tokens":42}

event: session_end
data: {"id":"01J9X7","report_id":"r_42"}
```

### sse-cli

If `websocat` is not available, [sse-cli](https://github.com/mpetazzoni/sseclient)
or `curl --no-buffer` work for SSE specifically:

```bash
$ curl --no-buffer http://127.0.0.1:8000/api/sessions/01J9X7/events
```

## CLI tools

For projects that ship a CLI as the deployable artifact, the demo is
just running the CLI. Paste the command and its output:

```bash
$ ./bin/mytool render --input fixtures/sample.json
Rendered 4 sections, 1.2 KB output written to ./out/sample.html
```

For interactive CLIs, use `expect` or `script` to capture a session.

## Background jobs / queue workers

Demonstrate by:

1. Enqueuing a job via the project's normal mechanism (CLI, API call,
   `redis-cli LPUSH`, AWS console).
2. Pasting log lines from the worker showing the job picked up.
3. Asserting the externally-visible side effect (a row inserted, an
   email caught by a catch-all, a file written).

```bash
$ redis-cli LPUSH jobs:render '{"topic":"atlas"}'
(integer) 1

$ tail -n 20 worker.log | grep render
2026-05-17T12:34:56 INFO render-worker picked up job render-01J9X7
2026-05-17T12:35:12 INFO render-worker job render-01J9X7 succeeded (took 16s)

$ ls output/
01J9X7.html
```

## Performance / load (only when the requirement is performance-related)

### k6

```javascript
// load/sessions_p95.js
import http from "k6/http";
import { check } from "k6";

export const options = {
  vus: 50,
  duration: "30s",
  thresholds: { "http_req_duration{p(95)}": ["<500"] },
};

export default function () {
  const r = http.post("http://127.0.0.1:8000/api/sessions", JSON.stringify({ topic: "atlas" }), {
    headers: { "Content-Type": "application/json" },
  });
  check(r, { "status 201": (r) => r.status === 201 });
}
```

### wrk / hey

For quick "is it broken under load?" probes. `wrk -t4 -c100 -d10s http://...`
gives a one-line answer.

## Database state

For requirements where the user-visible outcome includes a persisted
record, demo the write by querying the real datastore:

```bash
$ mongosh atlas-test --eval "db.sessions.findOne({_id: '01J9X7'})"
{
  _id: '01J9X7',
  status: 'complete',
  report_id: 'r_42',
  ...
}
```

```bash
$ psql -h localhost -d app -c "SELECT id, status FROM sessions WHERE id = '01J9X7';"
   id    | status
---------+----------
 01J9X7  | complete
```

## What goes into `tasks.md`

Per task, paste **enough** to convince a reviewer the new behaviour
works:

```markdown
- **Demo:**
  ```
  $ curl -sS -X POST http://127.0.0.1:8000/api/sessions -d '{"topic":"atlas"}'
  {"id":"01J9X7","status":"created"}

  $ curl -sS http://127.0.0.1:8000/api/sessions/01J9X7
  {"status":"complete","report":{"content":"Atlas is..."}}
  ```
  Screenshot of the rendered SessionDetailPage showing the report:
  `docs/demos/T-23.png`
```

If a task is genuinely pure code (a private utility function, an
internal type), the demo block can read:

```markdown
- **Demo:** No demo applicable; this task ships internal pure code with
  no user-visible surface. Tests cover the full function surface.
```

That note is itself the discipline — it forces you to confirm there
genuinely is no user-visible behaviour to observe, rather than skipping
the demo by default.

## When to skip the demo

The Demo gate is required. The only legitimate skip is the explicit
"no demo applicable" note above. If you find yourself wanting to skip
it for any other reason (slow start-up, missing test data, "I'll
demo later"), the right move is to fix the underlying problem — not
suppress the gate.
