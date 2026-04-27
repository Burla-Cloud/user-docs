# Make millions of API calls without lying about the rate cap

The compromised version tests 5,000 ids and says the backfill is "ready." The real version calls the API for every id while respecting the provider's global cap. If the small run never hit 429s, it did not test the system you plan to run.

This demo chunks 2,000,000 ids into 2,000 tasks. Burla runs up to 1,000 workers, and each worker sleeps one second between requests, giving roughly 1,000 requests per second globally.

## what we built

Chunking is the contract. Each chunk is big enough to amortize worker startup and small enough to stream results as they finish.

```python
with open("user_ids.txt") as f:
    user_ids = [line.strip() for line in f if line.strip()]

CHUNK = 1000
chunks = [user_ids[i:i + CHUNK] for i in range(0, len(user_ids), CHUNK)]
```

The worker owns retry and local pacing. That keeps rate-limit behavior close to the HTTP call instead of hiding it in a scheduler.

```python
def enrich_chunk(ids: list[str]) -> list[dict]:
    import time
    import httpx

    out = []
    with httpx.Client(timeout=30.0, headers={"Authorization": "Bearer $API_KEY"}) as client:
        for uid in ids:
            for attempt in range(5):
                r = client.get(f"https://api.example.com/v1/users/{uid}")
                if r.status_code == 429:
                    time.sleep(float(r.headers.get("Retry-After", 2 ** attempt)))
                    continue
                r.raise_for_status()
                out.append({"user_id": uid, **r.json()})
                break
            time.sleep(1.0)
    return out
```

## how the pipeline works

`max_parallelism` is the important line. It is the global throttle for live workers.

```python
from burla import remote_parallel_map

results = remote_parallel_map(
    enrich_chunk,
    chunks,
    func_cpu=1,
    func_ram=2,
    max_parallelism=1000,
    generator=True,
    grow=True,
)
```

## why this demo is interesting

Rate limits are where toy parallelism lies. A local async script can look fast until it meets the provider's real cap, then it becomes a retry storm. The useful experiment is not "can I make requests?" It is "can I finish the whole backfill without breaking the contract?" That means chunking, global concurrency, local pacing, and resumable output.

This pattern also works for LLM providers. Replace user ids with prompts, make the per-worker sleep reflect RPM or TPM, and stream results as JSONL. Burla handles the worker count; your code still handles provider-specific behavior such as `Retry-After`, failed payloads, and partial output.

## how to build your version

Start with the provider's real rule: requests per second, concurrent sessions, daily tokens, or model TPM. Convert it into a per-worker budget. Put 429 handling inside the worker. Use `generator=True` to stream JSONL to disk so a long backfill does not hold everything in memory.

## why Burla fits

The annoying parts are queue setup, worker deployment, retries, and live concurrency. Burla supplies the worker fleet and the cap. Your function still owns API semantics, which is where they belong.

## what the tiny run misses

A compromised run proves the endpoint responds. The real run proves your pacing, retry, and streaming logic survive the full backfill. Without that, the missed discovery is the provider banning you halfway through the job.
