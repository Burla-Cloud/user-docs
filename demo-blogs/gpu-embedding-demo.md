# Embedding Wikipedia on A100 workers

Vector search demos often cheat on the expensive part. They embed a few thousand rows, ship a prebuilt FAISS index, then talk about retrieval. This demo keeps the expensive part visible: download Wikipedia Parquet shards, embed titles and text with `BAAI/bge-large-en-v1.5`, write `.npy` vector shards, embed a query, and rank by cosine similarity.

It uses a custom Docker image, `jakezuliani/burla-embedder:latest`, based on `pytorch/pytorch:2.4.0-cuda12.1-cudnn9-runtime`. That detail matters. CUDA wheels are already in the image, so the driver avoids top-level `torch` imports that would make Burla install CPU wheels over the image's GPU stack.

## What The Job Does

The compromised version embeds 1,000 or 5,000 Wikipedia rows on one GPU. It proves the model works, but it hides the queueing problem: many shards, model load time, shared output paths, query embedding, and result assembly.

The real version defaults to 50,000 articles split into shards. CPU workers download Parquet files from the Hugging Face dataset `wikimedia/wikipedia`, write JSONL text shards to `/workspace/shared/vector_embeddings_demo/texts`, then A100 workers load Sentence Transformers and write normalized float32 embeddings to `/workspace/shared/vector_embeddings_demo/embeddings`. The client downloads those vector shards from the shared GCS bucket and computes top-K similarity locally.

The demo also supports staged reruns with `DEMO_STAGE=download`, `embed`, or `search`, so a failed query step does not force another GPU embedding pass.

## The Code Shape

The worker cache is the whole trick. A GPU worker may process multiple tasks; the module-level `cache` keeps the model loaded inside that worker process.

```python
cache = {}

def embed_shard(shard_path, model_name, shared_root):
    import numpy as np
    import torch
    from sentence_transformers import SentenceTransformer

    if "model" not in cache:
        print(f"loading {model_name} (cuda={torch.cuda.is_available()}) ...")
        cache["model"] = SentenceTransformer(model_name, device="cuda")

    model = cache["model"]
    vecs = model.encode(
        texts,
        batch_size=64,
        normalize_embeddings=True,
        show_progress_bar=False,
        convert_to_numpy=True,
    ).astype("float32")
    np.save(emb_path, vecs)
    return {"emb_path": str(emb_path), "ids_path": str(ids_path), "n": len(ids)}
```

The GPU fan-out is explicit:

```python
embed_results = remote_parallel_map(
    embed_shard,
    embed_inputs,
    image=IMAGE,
    grow=True,
    func_gpu="A100",
    max_parallelism=max_par,
)
```

For the query, the same image and model run on one A100, then the client multiplies the query vector by the concatenated matrix.

## Why It Matters

Embedding pipelines fail in boring ways: Python version mismatch, CPU wheels replacing CUDA wheels, model reload per task, vector shards too large for the driver, and unclear restart boundaries. This repo addresses those directly.

For ML engineers, the useful part is the split between CPU download work, GPU embedding work, and local retrieval. Burla is the way those phases run without a separate orchestrator. The model, Docker image, GCS-backed shared workspace, and A100 limit are all visible in the code.
