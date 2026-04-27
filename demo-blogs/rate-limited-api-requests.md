# Make millions of API calls without lying about the rate cap

In this example we:

* Split 2,000,000 ids into 2,000 chunks.
* Run up to 1,000 Burla workers.
* Sleep inside each worker so the global rate stays around 1,000 requests per second.

A 5,000-id test does not tell you much if it never hits the provider's real limit.

### Step 1: Chunk the ids

Each chunk is large enough to amortize startup and small enough to stream results as it finishes.

```python
with open("user_ids.txt") as f:
    user_ids = [line.strip() for line in f if line.strip()]

CHUNK = 1000
chunks = [user_ids[i:i + CHUNK] for i in range(0, len(user_ids), CHUNK)]
```

### Step 2: Put pacing near the HTTP call

The worker owns local pacing and retry behavior.

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

### Step 3: Cap live workers

`max_parallelism` is the global throttle.

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

### What's the point?

Rate limits are where toy parallelism lies. A local async script can look great until it turns into a retry storm.

The useful question is: can I finish the whole backfill without breaking the provider's contract? That means chunking, local sleeps, global concurrency, and output streaming. Burla handles the worker fleet; your function still owns the API behavior.
