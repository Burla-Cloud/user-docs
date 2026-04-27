# Distilling 571 million Amazon reviews

Amazon reviews are a strange public corpus because they are both huge and personal. The `McAuley-Lab/Amazon-Reviews-2023` dataset has 571,544,386 reviews across 34 categories, served as 275 GB of JSONL from Hugging Face. You can sample it in a notebook. You cannot understand its tail behavior from a sample.

This demo scans the whole corpus for profanity, caps-lock rants, long complaint monologues, censored slurs, punctuation storms, and five-star reviews that say almost nothing. It does not use an LLM. The funny and ugly parts come from regexes, byte ranges, heaps, and enough workers to read every category in parallel.

## What The Job Does

The compromised version reads a few million rows from one category and makes a wall of weird reviews. That gets you screenshots, but the rankings are weak. A category like Video Games has a different profanity rate than Gift Cards, and the most extreme punctuation example might sit 180 GB into the corpus.

The real version divides every category file into roughly 500 MB HTTP Range chunks. Each worker opens one byte range against the Hugging Face CDN, aligns to a newline, parses JSON row by row, and keeps small top-K heaps in memory. It writes one shard JSON to `/workspace/shared/ard/shards/` or `/workspace/shared/ard_worst/shards/`. A reduce pass merges those shards into frontend artifacts.

The result is a deterministic map-reduce pipeline over raw review text. No local full download. No Spark cluster. No model pretending to judge tone.

## The Code Shape

The stream primitive is the important part. Each worker receives `(file_path, byte_start, byte_end, chunk_id)` and only reads that slice.

```python
def stream_reviews(file_path: str, start: int, end: int):
    resp = requests.get(
        HF_BASE + file_path,
        headers={"Range": f"bytes={start}-{end - 1}"},
        stream=True,
        timeout=300,
    )
    buf = b""
    first_line = True
    for raw in resp.iter_content(chunk_size=1 << 16):
        buf += raw
        lines = buf.split(b"\n")
        buf = lines.pop()
        if first_line and start > 0 and lines:
            lines.pop(0)
        first_line = False
        for line in lines:
            if line.strip():
                yield json.loads(line)
```

The dispatcher sends hundreds of these chunks to Burla:

```python
results = remote_parallel_map(
    process_main,
    jobs,
    func_cpu=1,
    func_ram=4,
    grow=True,
    max_parallelism=1000,
    spinner=True,
)
```

Inside `process_main`, the worker scores text for strong, medium, and mild profanity, caps ratio, exclamation count, and several derived signals. It keeps top examples in heaps instead of returning every scored review to the driver.

## Why It Matters

A lot of public datasets are too large for the casual tools people use to explore them. Sampling is fine for schema discovery. It is bad for extremes.

This demo is useful because it shows a cheap pattern: partition the raw bytes, push simple scoring to the workers, return only the small summaries. Burla's role is to make that pattern feel like Python instead of a batch scheduler. The interesting artifact is the ranked corpus tail, not the infrastructure.
