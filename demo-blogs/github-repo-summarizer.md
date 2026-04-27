# Summarize a million READMEs without calling an LLM

In this example we:

* Export 1,200,000 GitHub READMEs from BigQuery.
* Upload the Parquet to Burla shared storage.
* Run deterministic summarizers over 600 shards and reduce into frontend JSON.

I like this one because the first instinct is to ask an LLM. That would make individual rows prettier and the aggregate harder to trust.

### Step 1: Put the Parquet where workers can read it

The worker reads a stripe of `/workspace/shared/grs/readmes.parquet` and emits one JSON shard.

```python
SHARD_OUT = "/workspace/shared/grs/shards"
PARQUET_PATH = "/workspace/shared/grs/readmes.parquet"

CATEGORIES = {
    "ml": {"tensorflow": 4, "pytorch": 4, "embedding": 2, "llm": 4},
    "web": {"react": 3, "django": 2, "graphql": 3, "frontend": 2},
    "devops": {"docker": 3, "kubernetes": 4, "terraform": 4},
}
```

### Step 2: Fan out the shards

The map stage runs `summarize_shard` across 600 workers.

```python
jobs = [(i, args.shards) for i in range(args.shards)]
results = remote_parallel_map(
    summarize_shard,
    jobs,
    func_cpu=args.func_cpu,
    func_ram=args.func_ram,
    grow=True,
    max_parallelism=args.parallelism,
)
```

### Step 3: Reduce counters and examples

The reducer keeps heaps per category and language, plus document-frequency counters for TF-IDF.

```python
def reduce_bucket(bucket_idx: int, n_buckets: int, top_per_cat: int, top_per_lang: int, sample_cap: int) -> dict:
    files = sorted(f for f in os.listdir("/workspace/shared/grs/shards") if f.endswith(".json"))
    my_files = [f for i, f in enumerate(files) if i % n_buckets == bucket_idx]
    by_cat, by_lang, doc_freq = {}, {}, {}
    cat_heaps = {}
    for fn in my_files:
        with open(os.path.join("/workspace/shared/grs/shards", fn)) as f:
            rows = json.load(f).get("rows", [])
        for row in rows:
            cat = row.get("category", "other")
            quality = row.get("badges", 0) * 1.5 + row.get("code_blocks", 0) * 0.3
            heapq.heappush(cat_heaps.setdefault(cat, []), (quality, row["repo"], row))
```

### What's the point?

Pretty summaries of famous repos are the boring version. I care about README culture at scale: install instructions, badges, code fences, category words, cloned templates, and empty placeholders.

A model would make the rows sound smoother. I do not want smoother here. I want counts I can debug. If a category looks wrong, I can inspect the keyword weights and rerun the reduce.
