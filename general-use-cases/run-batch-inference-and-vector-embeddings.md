---
description: Score records or create embeddings without deploying an endpoint.
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: false
  tags:
    visible: true
---

# Run batch inference and vector embeddings

Use this when you need predictions or embeddings for many records, documents, images, or reviews.
Do not use this for live request serving or a tiny one-off prediction.
The unit of work is a batch of records, a file shard, or a document shard.
Each worker loads the model, scores its batch, and writes or returns compact output.
The output is usually JSONL, Parquet, vectors, or a manifest of paths.

Batch inference is a job, not an endpoint. If no user is waiting on a response, keep it as a Python job.

## The pattern

1. Build batches from your source data.
2. Run one batch per Burla worker.
3. Load the model inside the worker.
4. Write predictions or embeddings as shard outputs.
5. Reduce only if you need one final file, index, or report.

This keeps serving concerns out of an offline job. There are no health checks, request schemas, autoscaling policies, or idle endpoints.

## Build batches

Keep the planner boring. Read source rows, then split them into batches.

```python
import pyarrow.dataset as ds

dataset = ds.dataset("s3://my-bucket/reviews/", format="parquet")
table = dataset.to_table(columns=["review_id", "text"])
rows = table.to_pylist()

BATCH_SIZE = 10_000
batches = [rows[i:i + BATCH_SIZE] for i in range(0, len(rows), BATCH_SIZE)]
```

Batch size is a resource choice:

1. bigger batches reduce model load overhead
2. smaller batches reduce memory risk
3. GPU batches must fit GPU memory
4. output batches should be small enough to retry

## Write the worker

Load the model inside the worker. Do not serialize a model from your laptop into the function input.

```python
def score_reviews(batch):
    from transformers import AutoModelForSequenceClassification, AutoTokenizer
    import torch

    if not hasattr(score_reviews, "_model"):
        model_name = "cardiffnlp/twitter-roberta-base-sentiment-latest"
        score_reviews._tokenizer = AutoTokenizer.from_pretrained(model_name)
        score_reviews._model = AutoModelForSequenceClassification.from_pretrained(model_name).eval()

    tokenizer = score_reviews._tokenizer
    model = score_reviews._model
    encoded = tokenizer(
        [row["text"] for row in batch],
        padding=True,
        truncation=True,
        max_length=256,
        return_tensors="pt",
    )
    with torch.no_grad():
        probabilities = torch.softmax(model(**encoded).logits, dim=-1).numpy()

    return [
        {"review_id": row["review_id"], "score": float(probability.max())}
        for row, probability in zip(batch, probabilities)
    ]
```

The cache lives in the worker process. Later inputs that land on the same worker can reuse the loaded model.

## Run CPU batch inference

Use CPU workers for smaller transformers, sklearn models, rules, and models whose bottleneck is Python or object storage.

```python
import json
from burla import remote_parallel_map

with open("review-scores.jsonl", "w") as f:
    for scored_batch in remote_parallel_map(
        score_reviews,
        batches,
        func_cpu=4,
        func_ram=16,
        generator=True,
        grow=True,
    ):
        for row in scored_batch:
            f.write(json.dumps(row) + "\n")
```

`generator=True` keeps the client from holding every prediction in memory.

## Run embeddings on GPUs

Use GPUs when the model needs CUDA or when CPU throughput makes the job too slow.

For GPU jobs, use a CUDA image and ask for a GPU only on the embedding call.

```python
from pathlib import Path

shard_paths = [str(path) for path in Path("/workspace/shared/doc-shards").glob("*.jsonl")]
```

```python
def embed_documents(shard_path):
    import json
    from pathlib import Path

    import numpy as np
    from sentence_transformers import SentenceTransformer

    if not hasattr(embed_documents, "_model"):
        embed_documents._model = SentenceTransformer("BAAI/bge-large-en-v1.5", device="cuda")

    rows = [json.loads(line) for line in Path(shard_path).read_text().splitlines()]
    texts = [row["text"] for row in rows]
    vectors = embed_documents._model.encode(texts, batch_size=64, normalize_embeddings=True).astype("float32")

    output_path = Path("/workspace/shared/embeddings") / f"{Path(shard_path).stem}.npy"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    np.save(output_path, vectors)
    return {"input_path": shard_path, "output_path": str(output_path), "rows": len(rows)}
```

```python
embedding_reports = remote_parallel_map(
    embed_documents,
    shard_paths,
    image="us-docker.pkg.dev/my-project/burla/embedder:latest",
    func_gpu="A100",
    func_cpu=4,
    func_ram=32,
    max_parallelism=8,
    grow=True,
)
```

`max_parallelism` should usually match GPU quota or the downstream system you write to.

## Write outputs where they belong

Return rows when the output is small enough to stream through the client.

Write files when the output is large.

For vector embeddings, returning arrays through the client is usually the wrong shape. Write vector shards to `/workspace/shared` or your vector store, then return paths and counts.

```python
embedding_paths = [report["output_path"] for report in embedding_reports]
total_rows = sum(report["rows"] for report in embedding_reports)

print(f"Wrote {len(embedding_paths)} embedding shards for {total_rows} rows")
```

If you need one final manifest, reduce the reports into a table.

```python
import pandas as pd

pd.DataFrame(embedding_reports).to_parquet("embedding-manifest.parquet", index=False)
```

## Choose resources

Start with one worker's needs:

1. `func_cpu`: preprocessing threads, tokenization, or model threads
2. `func_ram`: model size plus batch size plus output buffer
3. `func_gpu`: GPU type for CUDA models
4. `image`: CUDA runtime or model dependencies
5. `max_parallelism`: GPU quota, API quota, vector database write limit, or object storage pressure

Do not put every stage on GPUs. Downloading, parsing, filtering, and manifest building are often CPU work.

## When not to use this

Use a serving endpoint when users need low-latency online responses.

Use a local script when you have a few hundred rows and the model fits comfortably on your laptop.

Use a custom pipeline when each item depends on the previous item. Batch inference works best when each input can run alone.

## Examples that use this pattern

- [Run batch inference as a job, not an endpoint](../demo-blogs/ml-inference-batch.md)
- [Put the embedding model on A100s, then ask the search question](../demo-blogs/gpu-embedding-demo.md)
- [Cluster all arXiv abstracts before naming extinct topics](../demo-blogs/arxiv-fossils.md)
- [Label-free visual search over the Met](../demo-blogs/met-weirdest-art.md)
- [Read/Write Files to Cloud Storage](../common-patterns/read-and-write-gcs-files.md)
- [Combine many results/files into one](../common-patterns/combine-many-results-files-into-one-map-reduce.md)
