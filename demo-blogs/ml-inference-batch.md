# Run batch inference as a job, not an endpoint

In this example we:

* Read review text from Parquet.
* Load a HuggingFace sentiment model once per worker.
* Score the whole corpus in batches and stream JSONL results.

I would not build an endpoint for this. There is no traffic to serve. There is just a pile of rows that need model scores.

### Step 1: Build batches

The client reads ids and text from Parquet, then builds 10,000-row batches.

```python
import pyarrow.dataset as ds

dataset = ds.dataset("s3://my-bucket/reviews/", format="parquet")
texts = dataset.to_table(columns=["review_id", "text"]).to_pandas()

BATCH = 10_000
batches = [texts.iloc[i:i + BATCH].to_dict("records") for i in range(0, len(texts), BATCH)]
```

### Step 2: Write the worker function

Each worker loads the model the first time it runs, then reuses it for later batches on the same process.

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

### Step 3: Run it on the cluster

The output streams back as each batch finishes, so we can write JSONL without holding everything in memory.

```python
from burla import remote_parallel_map

for batch_out in remote_parallel_map(
    predict_batch,
    batches,
    func_cpu=4,
    func_ram=16,
    generator=True,
    grow=True,
):
    for row in batch_out:
        f.write(json.dumps(row) + "\n")
```

### What's the point?

The endpoint version is usually overbuilt. Health checks, autoscaling, request formats, and idle capacity are useful when users are sending traffic. They are annoying when I just need to score a dataset once.

The real question is whether the model, batch size, token length, memory, and output format survive the full corpus. A tiny sample mostly tells you the imports work.
