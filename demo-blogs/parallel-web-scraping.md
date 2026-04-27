# Scrape the archive, not the easy page sample

In this example we:

* Read 1,000,000 URLs from a file.
* Scrape them in 500-URL chunks.
* Stream JSONL rows with either parsed fields or errors.

The stale pages, redirects, 503s, and broken HTML are the dataset. A happy-path sample filters out the part you most need to measure.

### Step 1: Chunk URLs

The client only plans work. Each worker gets a chunk.

```python
with open("urls.txt") as f:
    urls = [u.strip() for u in f if u.strip()]

CHUNK = 500
chunks = [urls[i:i + CHUNK] for i in range(0, len(urls), CHUNK)]
```

### Step 2: Fetch and parse inside the worker

The worker keeps one HTTP client open, backs off on temporary failures, and returns an error row when a page fails.

```python
def scrape_chunk(urls: list[str]) -> list[dict]:
    import random, time, httpx
    from selectolax.parser import HTMLParser

    out = []
    with httpx.Client(http2=True, timeout=20.0, follow_redirects=True) as client:
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
                    out.append({"url": url, "title": title.text(strip=True) if title else None})
                    break
                except (httpx.HTTPError, httpx.TimeoutException) as e:
                    if attempt == 3:
                        out.append({"url": url, "error": str(e)})
            time.sleep(0.5 + random.random() * 0.5)
    return out
```

### Step 3: Stream the crawl

Chunks stream back as they finish, so the scrape can run for hours without building one giant result list.

```python
from burla import remote_parallel_map

for rows in remote_parallel_map(
    scrape_chunk,
    chunks,
    func_cpu=1,
    func_ram=2,
    max_parallelism=1000,
    generator=True,
    grow=True,
):
    for row in rows:
        f.write(json.dumps(row) + "\n")
```

### What's the point?

Scraping gets weird fast. DNS, TLS, parser misses, per-site politeness, and failure logging all matter.

This design is plain on purpose: chunk URLs, reuse a client, parse the fields you need, return an error row when something fails. Once the output exists, you can compute failure rates by host or retry only bad chunks.
