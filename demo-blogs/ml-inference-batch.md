# Run batch inference as a job, not an endpoint

The compromised version scores 100,000 rows because setting up a serving stack for a one-shot job feels too heavy. The real experiment scores all 10 million rows with the model and batching you intend to ship. If endpoint setup changed the batch size, model, or corpus, it changed the answer.

This demo runs a HuggingFace sentiment model over Parquet reviews. Each Burla worker loads the model once and scores a batch.

## what we built

The client reads review ids and text from Parquet, then builds 10,000-row batches.

```python
import pyarrow.dataset as ds

dataset = ds.dataset("s3://my-bucket/reviews/", format="parquet")
texts = dataset.to_table(columns=["review_id", "text"]).to_pandas()

BATCH = 10_000
batches = [texts.iloc[i:i + BATCH].to_dict("records") for i in range(0, len(texts), BATCH)]
```

The worker keeps the model cached on the worker process after the first call.

```python
def predict_batch(rows: list[dict]) -> list[dict]:
    from transformers import AutoTokenizer, AutoModelForSequenceClassification
    import torch

    if not hasattr(predict_batch, "_model"):
        name = "cardiffnlp/twitter-roberta-base-sentiment-latest"
        predict_batch._tok = AutoTokenizer.from_pretrained(name)
        predict_batch._model = AutoModelForSequenceClassification.from_pretrained(name).eval()

    enc = predict_batch._tok([r["text"] for r in rows], padding=True, truncation=True, max_length=256, return_tensors="pt")
    with torch.no_grad():
        probs = torch.softmax(predict_batch._model(**enc).logits, dim=-1).numpy()
    return [{"review_id": r["review_id"], "score": float(p.max())} for r, p in zip(rows, probs)]
```

## how the pipeline works

Burla fans out the batches and streams predictions into JSONL.

```python
from burla import remote_parallel_map

for batch_out in remote_parallel_map(
    predict_batch, batches, func_cpu=4, func_ram=16, generator=True, grow=True,
):
    for row in batch_out:
        f.write(json.dumps(row) + "\n")
```

## why this demo is interesting

Batch inference is often forced through serving tools because those tools are familiar, not because the job is serving traffic. An endpoint has health checks, autoscaling, request formats, and idle capacity. A one-time corpus scoring job needs none of that. It needs a model, batches, workers, and output.

The real run also tests model economics. Batch size, max token length, RAM, and model load time decide cost and throughput. A compromised sample can hide that by running warm on one machine. Running the full corpus through Burla shows whether the model cache, batch size, and output format survive contact with real text length variation.

## how to build your version

Plan around model load time and output size. Use CPU workers for smaller transformers or sklearn models. Use a GPU image when CUDA matters. Keep top-level imports aligned with what Burla should install, and put giant model objects inside the worker so they are loaded on the worker, not serialized from the client.

## why Burla fits

Burla removes endpoint setup, SageMaker manifests, Batch queues, IAM wiring, and manual worker fleets. You keep batch inference as a Python function over records.

## what the sample misses

The compromised run tells you model code works. The real run tells you throughput, memory, and tail examples across the full corpus. The discovery you miss is usually in the long tail: the category where confidence collapses or the text length that breaks batching.
