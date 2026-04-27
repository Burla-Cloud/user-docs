# Finding extinct ideas in arXiv

arXiv has decades of paper metadata in one public snapshot. That makes it a good corpus for a question that is hard to answer by browsing: which research topics peaked, faded, or appeared recently?

This demo processes about 2.7 million arXiv papers from the Hugging Face mirror `jackkuo/arXiv-metadata-oai-snapshot`. It shards metadata to Parquet, embeds title and abstract text with `sentence-transformers/all-MiniLM-L6-v2` through `fastembed`, clusters the vectors, and writes HTML reports for extinct topics, newborn topics, and the loneliest paper in the corpus.

## What The Job Does

The compromised version caps the corpus at 50,000 papers and runs locally. That version is good for checking date parsing, embedding speed, and report rendering. It is bad for the actual question because niche clusters and old topics get distorted by sampling.

The real version has three stages. Stage 0 runs on one larger worker, downloads the metadata snapshot, extracts title, abstract, categories, and created date, then writes roughly 10,000-paper Parquet shards under `/workspace/shared/arxiv-fossils/raw`. The map stage sends each raw shard to one worker. The worker loads a `fastembed` ONNX model, embeds title plus abstract, normalizes vectors, and writes vector Parquet to `/workspace/shared/arxiv-fossils/vec`.

The reduce stage runs on one large worker. It loads every vector shard, fits MiniBatchKMeans on a sample, assigns all vectors, ranks topic clusters by temporal decline or recent burst, and uses FAISS HNSW nearest-neighbor search to find the paper whose fifth neighbor is farthest away.

## The Code Shape

The worker explicitly pins thread counts. ONNX Runtime otherwise sees the host CPU count inside a cgroup-limited worker and oversubscribes itself.

```python
os.environ.setdefault("OMP_NUM_THREADS", "1")
os.environ.setdefault("MKL_NUM_THREADS", "1")
os.environ.setdefault("ONNXRUNTIME_NUM_THREADS", "1")
os.environ.setdefault("OPENBLAS_NUM_THREADS", "1")

def embed_shard(raw_path: str) -> str:
    tbl = pq.read_table(raw_path)
    texts = [f"{t}\n{a}" for t, a in zip(titles, abstracts)]

    model = _get_model()
    vecs_iter = model.embed(texts, batch_size=EMBED_BATCH)
    vecs = np.asarray(list(vecs_iter), dtype="float32")
    vecs = _l2_normalize(vecs)

    pq.write_table(out_tbl, str(out_path))
    return str(out_path)
```

The dispatch keeps the stages clear:

```python
[raw_paths] = list(remote_parallel_map(
    stage_raw,
    [None],
    func_cpu=8,
    func_ram=32,
))

vec_paths = list(remote_parallel_map(
    embed_shard,
    raw_paths,
    func_cpu=1,
    func_ram=4,
))

[results_dir] = list(remote_parallel_map(
    reduce_fossils,
    [vec_paths],
    func_cpu=16,
    func_ram=64,
))
```

## Why It Matters

This is the kind of job that tempts teams into overbuilding. You need a big one-time map, a memory-heavy reduce, and durable intermediate files. You do not need a long-lived cluster framework if the job is a Python pipeline.

The useful lesson is the split: embed shards independently, store vectors as Parquet, reduce with FAISS and scikit-learn where the global matrix is needed. Burla supplies the worker pool and shared filesystem so the code can stay close to that shape.
