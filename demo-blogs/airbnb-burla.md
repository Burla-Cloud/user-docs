---
cover: ../.gitbook/assets/more-examples/airbnb-burla-cover.png
coverY: 0
layout:
  width: default
  cover:
    visible: true
    size: hero
---

# Test Airbnb hypotheses at public-data scale

In this example we:

* Process every public Inside Airbnb listing across 119 cities and 4 quarterly snapshots.
* CLIP-score 1.7M photos, then use Claude Haiku Vision to validate the shortlists.
* Analyze 50,686,612 reviews with a staged review funnel.

This is the demo where the infrastructure really changes the experiment. A few cities can make a nice story. The whole corpus tells you whether the story survives contact with the rest of the world.

### Step 1: Write artifacts, not vibes

Every stage writes Parquet under `/workspace/shared/airbnb`. That way a failed GPU stage does not force a listing re-download, and a new correlation test can reuse the same scored files.

```python
from burla import remote_parallel_map

results = remote_parallel_map(
    worker_fn,
    list_of_dataclass_inputs,
    func_cpu=1,
    func_ram=8,
    max_parallelism=1000,
    grow=True,
)
```

### Step 2: Score every photo first

The photo stage downloads image URLs, runs CLIP on CPU workers, and writes Parquet shards for the next stages. The broad pass stays cheap and parallel.

```python
results = remote_parallel_map(
    cpu_score_image_batch,
    batches,
    func_cpu=1,
    func_ram=4,
    max_parallelism=n_workers,
    grow=True,
    spinner=False,
)
```

### Step 3: Validate the photo shortlists

The original YOLOv8 GPU stage is still in the source repo for completeness, but it is not the live photo pipeline. The published demo uses Haiku Vision to validate the CLIP shortlists for pets, TV placement, hectic kitchens, and drug-den vibes.

```python
results = remote_parallel_map(
    haiku_validate_tv_batch,
    tv_batches,
    func_cpu=2,
    func_ram=8,
    max_parallelism=n_workers,
    grow=True,
    spinner=False,
)
```

### Step 4: Funnel the reviews

Reviews use the same shape: cheap heuristic scoring on every review, SBERT embeddings for the top 200,000, then Claude Haiku scoring for the top 12,000.

```python
results = remote_parallel_map(
    heuristic_score_batch,
    batches,
    func_cpu=1,
    func_ram=2,
    max_parallelism=n_workers,
    grow=True,
    spinner=False,
)

results = remote_parallel_map(
    embed_reviews_batch,
    embed_batches,
    func_cpu=2,
    func_ram=8,
    max_parallelism=n_workers,
    grow=True,
    spinner=False,
)
```

### What's the point?

I would not start with the website. I would start with artifact contracts: listings, photo manifest, CPU image scores, Haiku-validated photo categories, review scores, correlations, site JSON.

That sounds boring, but it is what keeps the project honest. You can rerun one stage, inspect the bad shard, or try a new hypothesis without paying for everything again. I think this is the difference between a fun notebook and an analysis you can actually believe.
