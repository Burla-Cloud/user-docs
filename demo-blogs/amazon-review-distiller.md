---
cover: ../.gitbook/assets/more-examples/amazon-review-distiller-cover.png
coverY: 0
layout:
  width: default
  cover:
    visible: true
    size: hero
---

# Distill 571 million Amazon reviews

In this example we:

* Stream 275 GB of Amazon review JSONL from HuggingFace.
* Split 34 category files into 545 byte-range chunks instead of downloading everything first.
* Build both the Wall of Rants and Unhinged Mode with deterministic scoring.

The goal is not just a funny sample. The repo parses 571,544,386 reviews, keeps tiny top-K heaps per shard, merges them into category findings, then runs a second worst-of-worst pass for censored strong profanity and categorized slur hits.

### Step 1: Plan byte ranges

Each category file is huge, so we turn it into roughly 500 MB jobs and keep one byte range per worker.

```python
import math
from pathlib import Path
from huggingface_hub import HfApi

def plan_chunks(chunk_mb: int = 500) -> list[tuple[str, int, int, str]]:
    infos = HfApi().list_repo_tree(
        "McAuley-Lab/Amazon-Reviews-2023",
        path_in_repo="raw/review_categories",
        repo_type="dataset",
        recursive=False,
    )
    files = sorted(
        [(i.path, i.size) for i in infos if getattr(i, "size", 0) > 0],
        key=lambda kv: -kv[1],
    )

    jobs = []
    chunk_bytes = chunk_mb * 1024 * 1024
    for path, size in files:
        n = max(1, math.ceil(size / chunk_bytes))
        span = size // n
        cat = Path(path).stem
        for i in range(n):
            start = i * span
            end = (i + 1) * span if i < n - 1 else size
            jobs.append((path, start, end, f"{cat}_{i:03d}"))
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

### Step 3: Run both scoring passes

The main pass scores profanity, caps, rants, five-star mismatch, and punctuation storms. The worst pass hunts censored strong profanity and categorized slur hits for Unhinged Mode.

```python
main_results = remote_parallel_map(
    process_main, jobs, func_cpu=1, func_ram=4,
    grow=True, max_parallelism=1000, spinner=True,
)

worst_results = remote_parallel_map(
    process_worst, jobs, func_cpu=1, func_ram=4,
    grow=True, max_parallelism=1000, spinner=True,
)
```

### Step 4: Reduce into site artifacts

One reducer merges the main shards into the Wall of Rants. Another merges the worst-of-worst shards. `analysis.py` then handles rescoring, deduping, search pools, category findings, and the Unhinged Mode JSON.

```python
import json
from pathlib import Path

[main] = remote_parallel_map(reduce_main, [0], grow=True, spinner=True)
[worst] = remote_parallel_map(reduce_worst, [0], grow=True, spinner=True)

Path("samples/ard_reduced.json").write_text(json.dumps(main))
Path("samples/ard_worst.json").write_text(json.dumps(worst))

# Then run: python analysis.py
```

### What's the point?

A sample can find funny reviews. It cannot tell you whether Video Games is actually more profane than Beauty, or whether one 10,594-exclamation review is rare or part of a pattern.

I also like that this version does not need an LLM. Regexes, counters, lengths, caps, punctuation, context classifiers, and heaps are enough to produce both the public Wall of Rants and the much harsher Unhinged Mode.
