# Put the embedding model on A100s, then ask the search question

The compromised vector demo embeds 500 articles on CPU and calls it semantic search. The real version uses the same GPU image and model you would use for a larger corpus, writes shards to shared storage, and searches across the combined matrix. If CUDA was removed to make the demo easy, the experiment got smaller.

This demo embeds 50,000 Wikipedia articles with `BAAI/bge-large-en-v1.5` on Burla A100 workers.

## what we built

The Docker image starts from a CUDA PyTorch runtime and bakes model weights into the image so workers do not race on HuggingFace downloads.

```python
IMAGE = "jakezuliani/burla-embedder:latest"
MODEL_NAME = "BAAI/bge-large-en-v1.5"
SHARED_ROOT = "/workspace/shared/vector_embeddings_demo"
MAX_GPU_PARALLELISM = int(os.environ.get("DEMO_MAX_GPU_PARALLELISM", 8))
```

Stage 1 downloads Wikipedia Parquet shards on CPU workers and writes JSONL to `/workspace/shared`.

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

## how the pipeline works

Stage 2 runs on A100s and caches the model in a module-level dict on each worker.

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
    embed_shard, embed_inputs,
    image=IMAGE, grow=True,
    func_gpu="A100",
    max_parallelism=MAX_GPU_PARALLELISM,
)
```

## why this demo is interesting

GPU demos often cheat by moving the model back to CPU or embedding a tiny corpus. That proves the Python imports work, but it does not test image pull time, CUDA compatibility, A100 quota, model cache behavior, or the split between CPU preprocessing and GPU embedding. Those are the parts that decide whether the real job runs.

This walkthrough also shows a practical CPU/GPU division. Download and text extraction do not need A100s. Embedding does. Query embedding can run on one GPU call so the client never needs CUDA. That split is where Burla is useful: the same script can ask for different worker shapes at different stages.

## how to build your version

Pin the Python version to match the image. Bake large weights into the image when startup downloads would dominate. Keep `/workspace/shared` as the handoff between CPU download, GPU embed, and client-side search through the GCS-backed shared workspace bucket.

## practical notes from the build

The image and client Python version must match. Burla workers run the custom CUDA image, and the client has to use the same Python major and minor version. That detail is boring until it blocks job assignment. The README calls it out because GPU jobs fail in ways that look like infrastructure when the cause is version drift.

The other habit to copy is lazy importing. Keep `torch` and `sentence_transformers` inside GPU worker functions so the client environment does not overwrite CUDA wheels inside the image.

## why Burla fits

Burla removes GPU fleet setup, mixed CPU/GPU scheduling, endpoint creation, and CUDA worker wiring. You can run CPU stages and GPU stages from the same Python file.


## ways to adapt it

For a larger corpus, keep the same stage boundaries and change only the shard plan. Raise article count, increase articles per shard until each A100 stays busy, and cap GPU parallelism to quota. For a different model, rebuild the image with the new weights already downloaded. For a vector database backfill, replace the local search stage with writes to your index after each embedding shard finishes.

## what the CPU toy misses

The real run exposes model load time, A100 quota, shard sizing, shared path behavior, and duplicate-title handling. The compromised CPU sample answers only whether the library imports.
