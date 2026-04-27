# Map what the world photographed by processing the whole Flickr slice

The compromised version samples geotagged photos and gets a few cute country labels. The real version reverse-geocodes 9,487,758 public Flickr photos, tokenizes the user-written tags, and lets country-level patterns compete globally. Sampling changes the question because rare regional signatures disappear first.

This demo uses the HuggingFace `dalle-mini/YFCC100M_OpenAI_subset` metadata. No captions are generated. The text comes from users.

## what we built

Each Burla worker downloads one metadata shard, filters to photos with latitude and longitude, reverse-geocodes in-process, and writes compact JSONL to the shared filesystem.

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

The output row keeps the fields needed for later token and region aggregation.

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

## how the pipeline works

The map stage runs hundreds of shard workers. The reduce stage partitions thousands of aggregate JSONs into 64 buckets and merges counters.

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

## why this demo is interesting

A map built from tags gets better as it gets larger. Country signatures are not evenly distributed; they come from weird concentrations of people, places, and habits. Small samples overstate tourist centers and erase local vocabulary. The real run lets uncommon tags compete because every geotagged photo contributes to the counters.

The pipeline is also a good example of doing less ML. Reverse geocoding and token counting answer the question more directly than captioning images. A reader building a similar project should ask whether metadata already contains the signal. If it does, spend the compute on coverage rather than model inference.

## how to build your version

Start by deciding what the text signal is: tags, titles, OCR, filenames, captions, or metadata. Keep per-record extraction in the map worker. Write shard outputs to `/workspace/shared`, then reduce counters by country, region, user, time, or whatever unit your question uses.

## practical notes from the build

The worker writes compact rows rather than final country stats because it keeps the reduce flexible. You can change tokenization, add a theme vocabulary, or compute city signatures without redownloading HuggingFace metadata. That is the payoff of treating `/workspace/shared/wpi/shards` as a durable intermediate layer.

Reverse geocoding inside the worker is also the right trade. Shipping millions of lat/lon pairs back to the client would create a new bottleneck. The worker already has the row and the CPU. It can attach country, admin region, and city before the record ever leaves the shard.

## why Burla fits

Burla removes the worker fleet, shared filesystem wiring, and reducer scheduling. It also keeps the notebook mental model: one worker function reads one shard and writes one shard.


## ways to adapt it

This pattern works for any geotagged text: social posts, incident reports, field notes, news archives, or support tickets with location fields. Keep the worker responsible for attaching geography and writing one compact row. Keep the reducer responsible for counters, distinctive terms, and representative examples. That split lets you change the vocabulary logic without paying for another global download.

## what the sample misses

The real run finds Kazakhstan owning "expedition" and Belgium owning battlefield vocabulary because every tag got a vote. The compromised sample mostly finds countries with lots of photos and misses the strange regional signatures.
