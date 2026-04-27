# Test Airbnb hypotheses at public-data scale

The compromised Airbnb analysis uses a few cities, primary listing images, and a small review sample. The real version runs across 1,097,241 listings, 1,406,718 photo URLs, 1,243,339 CPU-scored images, 48,122 GPU-detected images, and 50,686,612 reviews. If you shrink the inputs, the hypothesis changes from "is this true globally?" to "did I find a cute anecdote?"

This demo is a full project with stages, checkpoints, and shared Parquet artifacts under `/workspace/shared/airbnb`.

## what we built

Every stage is a `remote_parallel_map` over dataclass inputs. Workers write Parquet shards to the shared filesystem, and merge workers combine them.

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

The listing stage downloads and cleans each Inside Airbnb city dump.

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

## how the pipeline works

Images run through a CPU CLIP stage, then a smaller A100 YOLO stage on top candidates. Reviews run through a three-tier funnel: heuristic, embeddings and clusters, then Claude Haiku on the top 10,000.

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

## why this demo is interesting

This is the demo where infrastructure most clearly changes the experiment. A city sample can produce a slick story about decor, reviews, and demand, but it cannot tell whether the pattern survives across regions, room types, listing density, scrape failures, and language differences. The real run is messier and more honest.

It also shows why stage boundaries matter. Listings, photo manifests, CPU CLIP scores, GPU YOLO detections, review scores, correlations, and site JSON are separate artifacts. That means a bad GPU run does not force a listing re-download, and a new correlation test can reuse the same scored Parquet files.

## how to build your version

Split the project by artifact boundaries: clean listings, photo manifest, CPU image scores, GPU object detections, review scores, correlations, site JSON. Make every expensive stage resume from Parquet. Run a sample stage first when it has external cost, then run the full stage with the same code.

## practical notes from the build

The Airbnb project is also a reminder to checkpoint aggressively. Stage 2b CPU image scoring covered the full photo manifest, while Stage 3 GPU detection only ran on top candidates. That split made the GPU bill controllable and kept the broad visual features available even when the object detector had a lower success rate than planned.

For a similar project, do not start with the website. Start with artifact contracts. Decide what each stage writes, what downstream stages read, and which artifacts are cheap enough to regenerate. The site is then a view over outputs, not the thing holding the analysis together.

## why Burla fits

Burla removes the manual worker fleet, CPU/GPU split friction, shared filesystem wiring, queue setup, and scheduler code. It lets a notebook-style project grow into a multi-stage pipeline without turning into platform work.


## ways to adapt it

The same shape works for marketplaces, delivery apps, real estate portals, and local services. Keep public records as the spine, then add expensive signals only where they answer a hypothesis. CPU image embeddings can cover everything. GPU detection can focus on candidates. LLM review labeling can run after a heuristic funnel. That order keeps the cost tied to the question instead of the dataset size.

## what the city sample misses

The real run can test whether televisions, mirrors, plants, cleaning fees, and review weirdness correlate with demand across cities. The compromised city sample mostly discovers local taste and anti-bot luck.
