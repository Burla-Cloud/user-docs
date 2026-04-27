# Put an embedding model on A100s

In this example we:

* Download 50,000 Wikipedia articles.
* Embed them with `BAAI/bge-large-en-v1.5` on A100 workers.
* Write vector shards to shared storage, then search across the combined matrix.

This is the version of the demo I actually care about. If CUDA gets removed to make the example easy, then we are no longer testing the thing people get stuck on.

### Step 1: Use a GPU image

For this job we use a custom CUDA image with the model weights already baked in.

```python
IMAGE = "jakezuliani/burla-embedder:latest"
MODEL_NAME = "BAAI/bge-large-en-v1.5"
SHARED_ROOT = "/workspace/shared/vector_embeddings_demo"
MAX_GPU_PARALLELISM = int(os.environ.get("DEMO_MAX_GPU_PARALLELISM", 8))
```

I like baking the model into the image for demos like this. Otherwise every worker tries to download the same weights at startup, which turns the benchmark into a HuggingFace download test.

### Step 2: Download text shards on CPU workers

Downloading and trimming text does not need A100s. Each CPU worker grabs a Wikipedia Parquet shard and writes JSONL to the shared folder.

```python
def download_shard(shard_idx, articles_per_shard, shared_root):
    import io, json, urllib.request
    from pathlib import Path
    import pyarrow.parquet as pq

    url = PARQUET_URL_TEMPLATE.format(shard_idx=shard_idx % 41)
    with urllib.request.urlopen(urllib.request.Request(url, headers={"User-Agent": "burla-demo/1.0"})) as response:
        table = pq.read_table(io.BytesIO(response.read())).slice(0, articles_per_shard)

    out_path = Path(shared_root) / "texts" / f"shard-{shard_idx:05d}.jsonl"
    out_path.parent.mkdir(parents=True, exist_ok=True)
    out_path.write_text("\n".join(json.dumps({"id": r["id"], "title": r["title"], "text": r["text"][:2000]}) for r in table.to_pylist()))
    return str(out_path)
```

### Step 3: Run the embedding stage on A100s

Now each GPU worker loads the model once and embeds one text shard.

```python
def embed_shard(shard_path, model_name, shared_root):
    import json, numpy as np
    from pathlib import Path
    from sentence_transformers import SentenceTransformer

    if "model" not in cache:
        cache["model"] = SentenceTransformer(model_name, device="cuda")

    rows = [json.loads(line) for line in Path(shard_path).read_text().splitlines()]
    texts = [f"{r['title']}\n\n{r['text']}" for r in rows]
    vecs = cache["model"].encode(texts, batch_size=64, normalize_embeddings=True).astype("float32")
    return {"n": len(rows), "shape": list(vecs.shape)}
```

```python
embed_results = remote_parallel_map(
    embed_shard,
    embed_inputs,
    image=IMAGE,
    grow=True,
    func_gpu="A100",
    max_parallelism=MAX_GPU_PARALLELISM,
)
```

### Step 4: Search the vectors

After embedding, the client can load the vector shards and ask the search question. The client does not need CUDA for this part. The expensive bit already happened on the A100 workers.

### What's the point?

GPU examples lie when they quietly move the model back to CPU. The hard parts are image compatibility, CUDA wheels, model load time, GPU quota, and keeping the expensive stage separate from the cheap stage.

This example keeps that shape visible. CPU workers prepare text. A100 workers embed. The shared folder is the handoff. That is the thing I would copy into a real backfill.
