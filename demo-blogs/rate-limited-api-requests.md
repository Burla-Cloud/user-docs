# Parallel API calls with a global cap

External API enrichment is a common slow job. You have a few million user IDs, an endpoint that permits about 1,000 requests per second, and a local script that can either run for days or get banned.

This demo splits user IDs into chunks and runs those chunks across Burla workers while capping global parallelism. Each worker makes one request per second. With `max_parallelism=1000`, the whole job aims for about 1,000 requests per second without a central rate limiter.

## What The Job Does

The compromised version uses a local async client and sleeps between calls. That is fine for thousands of users. At millions, one machine is the bottleneck, and any retry storm is hard to reason about.

The real version reads `user_ids.txt`, groups IDs into 1,000-ID chunks, and submits up to 2,000 tasks while allowing only 1,000 workers to run at once. Each worker owns an `httpx.Client`, sends bearer-authenticated requests, respects `Retry-After` on HTTP 429, and sleeps one second after each successful request.

The driver streams chunk results and writes `enriched.jsonl`. That means the output grows continuously and memory stays bounded.

## The Code Shape

The worker keeps the policy explicit. It retries only 429s, uses the server's `Retry-After` when present, and otherwise backs off by attempt count.

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
                    wait = float(r.headers.get("Retry-After", 2 ** attempt))
                    time.sleep(wait)
                    continue
                r.raise_for_status()
                out.append({"user_id": uid, **r.json()})
                break
            time.sleep(1.0)
    return out
```

The dispatch is where the global cap lives:

```python
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

The arithmetic is easy to audit: one request per second per worker times at most 1,000 workers.

## Why It Matters

This is a useful pattern for data teams that need to enrich records from a partner API without building a queue service. The remote function contains the per-worker policy. `max_parallelism` is the coarse global knob.

Burla's role is not to hide rate limits. It makes the batch large enough to finish while leaving the rate behavior in code where a reviewer can see it.
