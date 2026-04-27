# Scrape the archive, not the easy page sample

The compromised scraper grabs a few thousand pages and calls the parser good. The real scraper runs the same parser over the whole URL list, where the stale pages, 503s, redirects, and odd HTML live. If the failures were filtered out by scale, you tested a smaller question.

This demo scrapes 1,000,000 URLs by chunking them into groups of 500 and running up to 1,000 Burla workers.

## what we built

The client only plans work. The worker keeps one `httpx.Client` open so HTTP/2 and connection pooling work inside each chunk.

```python
with open("urls.txt") as f:
    urls = [u.strip() for u in f if u.strip()]

CHUNK = 500
chunks = [urls[i:i + CHUNK] for i in range(0, len(urls), CHUNK)]
```

The parser is intentionally small: fetch, back off on temporary failures, parse the title and price metadata.

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

## how the pipeline works

The output streams to JSONL as chunks finish. That matters when the scrape runs for hours.

```python
from burla import remote_parallel_map

for rows in remote_parallel_map(
    scrape_chunk, chunks, func_cpu=1, func_ram=2,
    max_parallelism=1000, generator=True, grow=True,
):
    for row in rows:
        f.write(json.dumps(row) + "\n")
```

## why this demo is interesting

Scraping problems are rarely solved by making a single machine busier. DNS, TLS, parsing, per-site politeness, and failure logging all matter. The real question is whether the crawler can cover the archive without turning errors into invisible holes. A compromised sample tends to over-represent fresh pages and under-represent the stale pages that poison downstream analysis.

The design here is deliberately plain: chunk URLs, reuse a client per worker, parse the fields you need, and return an error row when a page fails. That makes the output useful even before it is perfect. You can compute failure rates by host, retry only bad chunks, or inspect parser misses without replaying the whole crawl.

## how to build your version

Keep the worker polite. Set a user agent, add per-worker sleeps, and cap global workers. Put parsing next to fetching so a failure returns a useful row. If the site needs JavaScript, use a browser tool or a browser image; for static pages, `httpx` is cheaper and faster.

## why Burla fits

Burla removes the Redis/Kafka queue, Kubernetes worker pool, deploy scripts, and cluster babysitting. You write the scraper as a function over chunks and tune one global cap.

## what the page sample misses

The real archive contains the broken pages that define the quality of your dataset. The compromised scrape finds only the happy path and misses the stale catalog pages, poison HTML, and rate-limit behavior that decide whether the crawl is usable.
