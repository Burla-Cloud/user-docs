# Distill 571 million reviews with byte ranges, not vibes

The compromised version samples reviews and asks for the funniest rants. The real version streams 275 GB of Amazon review JSONL with HTTP Range requests, scores every review with deterministic rules, and then reduces the top heaps. A sample changes which categories look angry.

This demo builds the Wall of Rants and a second worst-of-worst pass from `McAuley-Lab/Amazon-Reviews-2023` on HuggingFace.

## what we built

Planning starts from file sizes. Each category JSONL becomes roughly 500 MB byte ranges.

```python
def plan_chunks(chunk_mb: int = 500) -> list[tuple[str, int, int, str]]:
    from huggingface_hub import HfApi
    files = [(i.path, i.size) for i in HfApi().list_repo_tree(
        "McAuley-Lab/Amazon-Reviews-2023",
        path_in_repo="raw/review_categories",
        repo_type="dataset",
    ) if getattr(i, "size", 0) > 0]
    jobs = []
    for path, size in files:
        span = chunk_mb * 1024 * 1024
        for start in range(0, size, span):
            jobs.append((path, start, min(start + span, size), f"{Path(path).stem}_{start}"))
    return jobs
```

The streaming primitive aligns byte ranges to newline boundaries and parses JSON rows.

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
            yield json.loads(line)
```

## how the pipeline works

Workers keep top-K heaps for signals such as profanity, caps, rants, and exclamation storms. Reducers merge shard JSONs.

```python
results = remote_parallel_map(
    process_main,
    jobs,
    func_cpu=1,
    func_ram=4,
    grow=True,
    max_parallelism=1000,
)

[result] = remote_parallel_map(reduce_main, [0], grow=True)
```

## why this demo is interesting

A review corpus this large has two different questions hiding inside it. One is example hunting: find the wildest text. The other is measurement: which categories produce more of it, and how often? The compromised sample can answer the first question, badly. The real byte-range scan answers both.

The design avoids model cost and moderation drift. Scoring is regexes, counters, length, caps, punctuation, and heaps. That makes the top lists reproducible and lets the reduce stage explain why a review won. If you want a model later, use it after the reduce on the tiny candidate set, not across 571 million rows.

## how to build your version

Use byte ranges when the dataset is huge and line-oriented. Put scoring in deterministic code first: regexes, counters, heaps, and category rollups. Save LLM calls for the tiny top set if you need labels.

## practical notes from the build

The byte-range design is the part to copy. HuggingFace serves each category as a single line-oriented JSONL file, so the worker cannot assume it starts on a record boundary. The first partial line is discarded when `start > 0`, and the rest of the stream is parsed normally. That small detail is what makes arbitrary byte chunks safe.

The reduce is intentionally heap-based. Each worker keeps only the best examples for each signal and a handful of counters. That means the map output stays small even while the input is hundreds of gigabytes. A full raw-output write would turn the reduce into another storage problem.

## why Burla fits

Burla removes the queue, worker fleet, shared shard layout, reducer dispatch, and cluster babysitting. Each byte range is an input, and workers write compact JSON summaries to `/workspace/shared`.


## ways to adapt it

The same byte-range plan works for logs, chat exports, support tickets, or product catalogs stored as JSONL. The worker can score toxicity, policy hits, refund language, medical terms, or schema quality. The reduce can keep examples and rates side by side. That pairing matters because examples make the result readable, while rates keep it honest.

## what the sample misses

The real run found Video Games as the most profane category and a 10,594-exclamation review. The compromised sample would have found funny examples, but it would not know whether they were representative or rare.
