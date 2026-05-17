# Mock Parity Contracts

A mock-parity contract is a test that **proves a fake matches the real
dependency** for the methods the system actually uses. It is the
structural fix for the "all unit tests pass; production fails on first
request" class of failure.

The skill **requires** at least one parity contract per fake. This file
gives patterns for the common ones.

## The shape of a parity contract

```
1. Pick a small set of method calls the system actually performs on the
   dependency. Not every method on the surface — just the ones used.
2. Run those calls against the fake.
3. Run the same calls against the real dependency
   (testcontainers / sandbox account / VCR cassette).
4. Assert observable behaviour matches:
   - Return shape (does it have the same fields?)
   - Async-ness (is it an awaitable / Promise / Future or a value?)
   - Error taxonomy (which exception does an absent doc raise?)
   - Edge cases relevant to the methods used (e.g. "find_one returns
     None when no match" vs "raises NotFound").
```

If the fake diverges, you have two choices:

- **Fix the fake** if the divergence is something you can fix (write
  a thin adapter that wraps the fake's sync call in `asyncio.sleep(0)`
  and `await`s, for example).
- **Stop using the fake** for the divergent surface. Switch that
  particular test to testcontainers or a sandbox.

## Pattern A — Mongomock vs `AsyncMongoClient`

**The bug:** `mongomock` returns sync values; `AsyncMongoClient` from
`motor` / pymongo-async returns coroutines. Code that calls
`coll.find_one(...)` without `await` works against mongomock and
crashes against Atlas.

```python
# tests/parity/test_mongo_async_parity.py
import inspect, pytest
from mongomock_motor import AsyncMongoMockClient
from motor.motor_asyncio import AsyncIOMotorClient
from testcontainers.mongodb import MongoDbContainer

@pytest.fixture(scope="module")
def real_client():
    with MongoDbContainer("mongo:7") as mongo:
        yield AsyncIOMotorClient(mongo.get_connection_url())

@pytest.fixture
def fake_client():
    return AsyncMongoMockClient()

@pytest.mark.parametrize(
    "method, args, kwargs",
    [
        ("find_one",   ({"_id": "missing"},), {}),
        ("insert_one", ({"_id": "x", "v": 1},), {}),
        ("update_one", ({"_id": "x"}, {"$set": {"v": 2}}), {}),
        ("delete_one", ({"_id": "x"},), {}),
    ],
)
async def test_async_methods_return_awaitables(
    fake_client, real_client, method, args, kwargs
):
    fake_coll = fake_client["db"]["c"]
    real_coll = real_client["db"]["c"]
    fake_call = getattr(fake_coll, method)(*args, **kwargs)
    real_call = getattr(real_coll, method)(*args, **kwargs)
    # The critical parity: both must be awaitable.
    assert inspect.isawaitable(fake_call), f"fake .{method} not awaitable"
    assert inspect.isawaitable(real_call), f"real .{method} not awaitable"
    await fake_call
    await real_call
```

If `mongomock_motor` is unavailable or insufficient, write your own
thin async wrapper around `mongomock` that always returns awaitables.
Then the parity test is asserting *your wrapper* matches the real
driver, which is the right contract anyway.

## Pattern B — FakeLLM vs real provider

**The bug:** `FakeLLM` returns a fixed string; the real provider
streams tokens, sometimes errors mid-stream, has rate limits, and
returns structured tool calls. Tests pass against the fake; the route
crashes when the real provider 503s.

Use a recorded cassette layer (e.g. [VCR.py](https://vcrpy.readthedocs.io/),
[pytest-recording](https://github.com/kiwicom/pytest-recording),
[Polly.JS](https://netflix.github.io/pollyjs/) for Node) to capture a
small set of real responses and replay them in tests.

```python
# tests/parity/test_llm_parity.py
import pytest

@pytest.mark.vcr  # replays a cassette recorded against the real provider
async def test_llm_streams_tokens_and_completes(real_llm):
    chunks = []
    async for chunk in real_llm.stream("Summarise atlas in one sentence."):
        chunks.append(chunk)
    text = "".join(chunks)
    assert len(text) > 10
    assert len(chunks) >= 2  # actually streamed, not single shot

async def test_fake_llm_streams_like_real(fake_llm):
    chunks = []
    async for chunk in fake_llm.stream("..."):
        chunks.append(chunk)
    # Assert the *shape* matches: also async-iterable, also yields strings,
    # also yields more than one chunk for prompts of moderate length.
    assert len(chunks) >= 2
    assert all(isinstance(c, str) for c in chunks)
```

Re-record cassettes when the provider's response shape changes. CI runs
in replay mode (no network); periodic local re-recording is the manual
step that keeps parity honest.

## Pattern C — moto vs real AWS

[moto](https://docs.getmoto.org/) is a faithful local fake of many AWS
services. The parity contract here is usually narrow:

- For each AWS API actually used (e.g. `s3.put_object`, `s3.get_object`,
  `sqs.send_message`), assert the moto and real responses match in
  shape and error taxonomy for the methods used.
- Use a sandbox AWS account or [LocalStack](https://localstack.cloud/)
  with real semantics for the contract test.

A practical compromise: run the parity test against LocalStack (which
matches real AWS more closely than moto) on every CI run, and against
a real sandbox account on a nightly job.

## Pattern D — In-memory queue / pub-sub

For Redis / RabbitMQ / Kafka / NATS, run the contract test against
testcontainers and the in-memory fake. Assert:

- Message ordering matches (or is documented as not matching).
- Visibility timeouts behave equivalently.
- Backpressure / queue-full behaviour matches.

## Pattern E — Fake filesystem (`pyfakefs`, `memfs`)

Common divergences:

- Permission semantics on Windows vs POSIX.
- Atomic-rename behaviour across volumes.
- `os.walk` ordering.

Pick the methods used by the system and assert parity for those.

## Where parity tests live

- Directory: `tests/parity/` (or `tests/contract/`).
- Marker: `@pytest.mark.parity` or equivalent so you can run / skip
  them as a group.
- Run on CI on every commit. Slow ones (real cloud sandboxes) can be
  scheduled less frequently with a clear note in `tasks.md`.

## What NOT to do

- **Don't write a parity test that just asserts both call the same
  method.** That's tautological. The test must observe behavioural
  parity.
- **Don't use the fake in the parity test as a control for itself.**
  Run against the real (or a high-fidelity replica like testcontainers).
- **Don't assume one parity test covers the fake forever.** Add a new
  parity case whenever the system starts using a new method on the
  dependency. The contract is "fake matches real for what we use."
