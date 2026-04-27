# Summarize a million READMEs without calling an LLM

The compromised version samples popular repos and asks a model to summarize them. The real version streams 1,200,000 GitHub READMEs, applies deterministic heuristics to every one, and then asks what open-source projects say about themselves. If you used a model or a popularity sample, you ran a different experiment.

This demo uses BigQuery public GitHub data, writes a Parquet file, uploads it to Burla shared storage, maps 600 shards, and reduces into frontend JSON.

## what we built

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

The map stage uploads the Parquet once, then fans out `summarize_shard` across 600 workers.

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

## how the pipeline works

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

## why this demo is interesting

The point is not to produce pretty summaries of famous repositories. It is to count the culture of README files at scale: install instructions, badges, code fences, category words, cloned templates, and empty placeholders. An LLM would make individual rows smoother while making the aggregate harder to trust.

The real run keeps the analysis reproducible. Every category comes from visible keyword weights, every install method from a regex, and every distinctive word from TF-IDF over the reduced corpus. That makes the weird findings debuggable. If a category looks wrong, you can inspect the rules and rerun the reduce without changing a prompt.

## how to build your version

Keep the summary deterministic until you know you need a model. Stream source rows into Parquet, put the Parquet on `/workspace/shared`, write one shard per worker, and reduce heaps and counters in parallel buckets. Add an LLM later only for the small set of examples you want a human to read.

## practical notes from the build

The upload stage is easy to overlook. A 1.3 GB Parquet cannot be captured as a Python closure and shipped with the function. The demo gzips and chunks it, writes part files on `/workspace/shared`, then finalizes the Parquet on a worker. After that, every map worker reads the same shared path.

That pattern is useful outside READMEs. If the input artifact is too large to pickle, stage it once into the shared workspace and pass paths through Burla inputs. Keep the function small, keep the data near the workers, and make the reduce consume shard files rather than client memory.

## why Burla fits

Burla removes BigQuery output shipping, worker deployment, queue setup, and reducer scheduling. The shared filesystem is the bridge between map and reduce.


## ways to adapt it

This pattern is useful whenever text lives in many records but the output is aggregate behavior. You can scan docs, changelogs, package manifests, issue templates, or config files. Make workers emit compact facts, not prose. Let the reducer count, rank, and sample. That keeps the analysis explainable and avoids paying model cost before you know which rows matter.

## what the sample misses

The real run found template clones, install-method gaps, and category-level language ownership. The compromised sample of famous repos would miss the boilerplate swamp that defines the corpus.
