# Map what the world photographed

In this example we:

* Process 9,487,758 public Flickr photos from the YFCC100M subset.
* Reverse-geocode every photo with latitude and longitude.
* Build country-level signatures from user-written tags.

No captions are generated here. The text comes from people, which is part of what makes the result interesting.

### Step 1: Process one metadata shard per worker

Each worker downloads one HuggingFace metadata shard, keeps geotagged photos, reverse-geocodes them, and writes compact JSONL.

```python
REPO_ID = "dalle-mini/YFCC100M_OpenAI_subset"
OUTPUT_DIR = "/workspace/shared/wpi/shards"

def process_shard(shard_id: str) -> dict:
    import gzip, json, os, requests
    from huggingface_hub import hf_hub_url
    import reverse_geocoder as rg

    meta_url = hf_hub_url(REPO_ID, filename=f"metadata/metadata_{shard_id}.jsonl.gz", repo_type="dataset")
    rows = [json.loads(l) for l in gzip.decompress(requests.get(meta_url).content).decode("utf-8", errors="replace").split("\n") if l.strip()]
    geotagged = [r for r in rows if r.get("latitude") and r.get("longitude")]
    geo_results = rg.search([(float(r["latitude"]), float(r["longitude"])) for r in geotagged], mode=2)
```

### Step 2: Write the useful fields

The row keeps geography plus the fields needed for token counting later.

```python
with open(os.path.join(OUTPUT_DIR, f"{shard_id}.jsonl"), "w") as out_f:
    for r, geo in zip(geotagged, geo_results):
        out_f.write(json.dumps({
            "photoid": r["photoid"],
            "country_cc": geo.get("cc"),
            "admin1": geo.get("admin1"),
            "city": geo.get("name"),
            "title": (r.get("title") or "")[:300],
            "usertags": (r.get("usertags") or "")[:400],
        }) + "\n")
```

### Step 3: Reduce counters

The reduce stage partitions aggregate JSONs into 64 buckets and merges counters.

```python
bucket_blobs = remote_parallel_map(
    reduce_bucket,
    buckets,
    func_cpu=1,
    func_ram=4,
    grow=True,
    max_parallelism=64,
)
```

### What's the point?

A tag map gets better when it gets bigger. Small samples overstate tourist centers and erase regional vocabulary. The full run lets weird country signatures compete because every geotagged photo gets a vote.

My favorite part is that this is mostly not ML. Reverse geocoding and token counting answer the question directly. If the metadata already contains the signal, spend the compute on coverage instead of inventing a model step.
