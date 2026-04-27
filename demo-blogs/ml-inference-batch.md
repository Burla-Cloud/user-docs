# Batch ML inference over Parquet reviews

Batch inference jobs usually look easy in a notebook: load a model, read a few rows, call `model(**enc)`, write predictions. At dataset scale, the slow parts are batching, model load time, memory, and writing results without dragging every row through the laptop.

This demo reads review text from Parquet on S3, splits rows into 10,000-record batches, runs a Hugging Face sentiment model on Burla workers, and streams predictions to JSONL.

## What The Job Does

The compromised version runs the model on a few thousand reviews locally. That tests labels and tokenization. It hides the real cost: model load on many machines, chunk size, PyTorch memory, and output streaming.

The real version reads `s3://my-bucket/reviews/` with `pyarrow.dataset`, converts `review_id` and `text` to records, groups rows into batches, and submits those batches to Burla. Each worker gets 4 CPUs and 16 GB RAM. The worker lazily loads `cardiffnlp/twitter-roberta-base-sentiment-latest`, tokenizes its batch with max length 256, runs inference under `torch.no_grad()`, and returns a list of `{review_id, label, score}` rows.

The driver uses `generator=True` so prediction batches can be written to `predictions.jsonl` as they finish.

## The Code Shape

The worker caches the model on the function object so repeated calls in the same process do not reload weights.

```python
def predict_batch(rows: list[dict]) -> list[dict]:
    from transformers import AutoTokenizer, AutoModelForSequenceClassification
    import torch

    if not hasattr(predict_batch, "_model"):
        model_name = "cardiffnlp/twitter-roberta-base-sentiment-latest"
        predict_batch._tok = AutoTokenizer.from_pretrained(model_name)
        predict_batch._model = AutoModelForSequenceClassification.from_pretrained(model_name).eval()

    tok, model = predict_batch._tok, predict_batch._model
    texts = [r["text"] for r in rows]
    enc = tok(texts, padding=True, truncation=True, max_length=256, return_tensors="pt")
    with torch.no_grad():
        logits = model(**enc).logits
        probs = torch.softmax(logits, dim=-1).numpy()

    labels = ["negative", "neutral", "positive"]
    return [{"review_id": r["review_id"], "label": labels[p.argmax()], "score": float(p.max())} for r, p in zip(rows, probs)]
```

The map call is small:

```python
results = remote_parallel_map(
    predict_batch,
    batches,
    func_cpu=4,
    func_ram=16,
    generator=True,
    grow=True,
)
```

Top-level imports include `torch` and `transformers` so Burla installs those packages on workers.

## Why It Matters

This is the normal shape of many ML production chores: one model, many independent batches, a source Parquet dataset, and a line-oriented output file. It does not require a model-serving stack if the job is offline.

Burla gives the inference loop enough workers without changing the code into a service. The useful details remain in Python: batch size, tokenizer settings, RAM per worker, and streaming writes.
