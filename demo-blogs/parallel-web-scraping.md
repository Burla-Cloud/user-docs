# Scraping URLs with bounded parallelism

Web scraping is easy to parallelize badly. If every worker hammers the same site with no pacing, the job becomes a denial-of-service test. This demo uses Burla to spread work across many workers while keeping a polite per-worker delay and an explicit cap on global parallelism.

The source is simple: `urls.txt` becomes chunks of 500 URLs. Each worker uses `httpx` with HTTP/2, parses HTML with `selectolax`, extracts the title and a price meta tag, and returns rows. The driver streams results to `scraped.jsonl`.

## What The Job Does

The compromised version uses a local thread pool. That is fine for a few thousand pages. It runs into local CPU, DNS, socket, and bandwidth limits, and it couples the whole scrape to one machine.

The real version creates 2,000 chunks and caps the job to 1,000 concurrent Burla workers. Each worker has its own `httpx.Client`, retry loop, status handling for 429 and 503, random jitter, and a short sleep after each URL. That means the global job can be large while each worker behaves like a cautious scraper.

The output is appended locally as chunk results arrive. This avoids one giant return value and gives partial progress if the crawl is interrupted.

## The Code Shape

The worker function is regular Python. It imports `httpx` and `selectolax` inside the worker, reuses a client, and handles transient failures locally.

```python
def scrape_chunk(urls: list[str]) -> list[dict]:
    import random
    import time
    import httpx
    from selectolax.parser import HTMLParser

    out = []
    with httpx.Client(http2=True, timeout=20.0, headers=HEADERS, follow_redirects=True) as client:
        for url in urls:
            for attempt in range(4):
                try:
                    r = client.get(url)
                    if r.status_code in (429, 503):
                        time.sleep(2 ** attempt + random.random())
                        continue
                    r.raise_for_status()
                    tree = HTMLParser(r.text)
                    title = tree.css_first("title")
                    price = tree.css_first("meta[itemprop=price]")
                    out.append({"url": url, "title": title.text(strip=True) if title else None})
                    break
                except (httpx.HTTPError, httpx.TimeoutException) as e:
                    if attempt == 3:
                        out.append({"url": url, "error": str(e)})
            time.sleep(0.5 + random.random() * 0.5)
    return out
```

The dispatch uses `max_parallelism` and `generator=True`:

```python
for chunk_rows in remote_parallel_map(
    scrape_chunk,
    chunks,
    func_cpu=1,
    func_ram=2,
    max_parallelism=1000,
    generator=True,
    grow=True,
):
    for row in chunk_rows:
        f.write(json.dumps(row) + "\n")
```

## Why It Matters

The useful pattern is not "scrape faster at all costs." It is "make the unit of work small, cap global concurrency, and keep per-worker pacing obvious in code."

Burla is a fit when the URL list is much larger than one laptop should handle but the scraper is still ordinary Python. The demo keeps the operational choices visible: chunk size, retry behavior, 429 handling, and output streaming.
