# Test Airbnb hypotheses at public-data scale

In this example we:

* Process 1,097,241 public Airbnb listings.
* Score 1,243,339 images on CPU and 48,122 top candidates on A100s.
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

### Step 2: Clean the listings

The listing stage downloads each Inside Airbnb city dump and writes one cleaned Parquet file per city.

```python
def download_and_clean_city(args: DownloadCityArgs) -> dict:
    import gzip, io, os, pandas as pd, requests

    r = requests.get(args.listings_url, timeout=600, headers={"User-Agent": "Mozilla/5.0"})
    df = pd.read_csv(io.BytesIO(gzip.decompress(r.content)), low_memory=False)
    df["listing_id"] = df["id"].astype("int64")
    df["price_usd"] = df["price"].apply(_parse_price_inline)
    df["demand_proxy"] = pd.to_numeric(df.get("reviews_per_month"), errors="coerce").fillna(0)
    out_path = os.path.join(args.shared_root, f"{args.city_slug}.parquet")
    df.to_parquet(out_path, compression="zstd", index=False)
    return {"ok": True, "n_rows": int(len(df)), "shared_path": out_path}
```

### Step 3: Split CPU and GPU work

Images first go through a broad CPU CLIP stage. Then the top candidates go through a smaller A100 YOLO stage.

```python
results = remote_parallel_map(
    gpu_detect_image_batch,
    batches,
    func_cpu=8,
    func_ram=64,
    func_gpu="A100_40G",
    max_parallelism=n_workers,
    grow=True,
)
```

Reviews use the same idea: cheap heuristic pass first, embeddings and clusters second, Claude Haiku only on the top 10,000.

```python
results = remote_parallel_map(
    heuristic_score_batch,
    batches,
    func_cpu=1,
    func_ram=2,
    max_parallelism=n_workers,
    grow=True,
)
```

### What's the point?

I would not start with the website. I would start with artifact contracts: listings, photo manifest, CPU image scores, GPU detections, review scores, correlations, site JSON.

That sounds boring, but it is what keeps the project honest. You can rerun one stage, inspect the bad shard, or try a new hypothesis without paying for everything again. I think this is the difference between a fun notebook and an analysis you can actually believe.
