# Distill 571 million Amazon reviews

In this example we:

* Stream 275GB of Amazon review JSONL from HuggingFace.
* Split the files into byte ranges instead of downloading everything first.
* Keep top-K heaps for profanity, caps, rants, and exclamation storms.

The goal is the Wall of Rants. But the real question is bigger than funny examples: which categories produce this stuff, and how often?

### Step 1: Plan byte ranges

Each category file is huge, so we turn it into roughly 500MB jobs.

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

### Step 2: Stream records safely

The worker asks HuggingFace for a byte range, discards the first partial line when needed, and parses JSON rows.

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

### Step 3: Map and reduce

Workers keep tiny summaries. The reducer merges those summaries into the final leaderboards.

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

### What's the point?

A sample can find funny reviews. It cannot tell you whether Video Games is actually more profane than Beauty, or whether one 10,594-exclamation review is rare or part of a pattern.

I also like that this version does not need an LLM. Regexes, counters, lengths, caps, punctuation, and heaps are enough for the first pass. If you want model labels later, run them on the tiny candidate set after the reduce.
