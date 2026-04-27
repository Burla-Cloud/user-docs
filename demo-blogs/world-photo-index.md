# A world index from YFCC photo metadata

The YFCC100M subset on Hugging Face is not just images. It has geotags, user tags, titles, descriptions, and shard metadata. This demo builds a country and city text index from that public photo metadata without putting ML on the hot path.

Each Burla worker handles one shard from `dalle-mini/YFCC100M_OpenAI_subset`, downloads the shard metadata JSONL, filters geotagged photos, reverse-geocodes latitude and longitude with `reverse_geocoder`, and writes compact JSONL rows to `/workspace/shared/wpi/shards`. Later aggregation stages count photos and phrases by country, admin region, and city.

## What The Job Does

The compromised version samples a handful of shards and prints the top countries. That proves the API and reverse geocoder work. It does not produce a useful world index because many countries only appear in the long tail of shards.

The real version lists all 4,094 metadata shards and runs them across up to 1,000 one-vCPU Burla workers. The expected wall time in the repo notes is minutes, not hours, because each worker does small CPU and network work: download a `.jsonl.gz`, parse a few hundred rows, batch reverse-geocode points against an embedded KD-tree, and write JSONL.

The demo keeps CLIP image embedding out of the main path. That is a good decision. The first-order artifact is a geographic text index, and adding image models to every shard would make the pipeline slower without improving the main output.

## The Code Shape

The worker is small and self-contained.

```python
def process_shard(shard_id: str) -> dict:
    os.makedirs(OUTPUT_DIR, exist_ok=True)
    meta_url = hf_hub_url(
        REPO_ID,
        filename=f"metadata/metadata_{shard_id}.jsonl.gz",
        repo_type="dataset",
    )
    m = requests.get(meta_url, timeout=60)
    m.raise_for_status()

    rows = [
        json.loads(l)
        for l in gzip.decompress(m.content).decode("utf-8", errors="replace").split("\n")
        if l.strip()
    ]
    geotagged = [r for r in rows if r.get("latitude") and r.get("longitude")]
    points = [(float(r["latitude"]), float(r["longitude"])) for r in geotagged]
    geo_results = rg.search(points, mode=2) if points else []

    with open(out_path, "w") as out_f:
        for r, geo in zip(geotagged, geo_results):
            out_f.write(json.dumps({...}) + "\n")
    return {"shard": shard_id, "written": written, "output_path": out_path}
```

The scale-out file lists shards and sends them to Burla:

```python
results = remote_parallel_map(
    process_shard,
    shards,
    func_cpu=args.func_cpu,
    func_ram=args.func_ram,
    grow=True,
    max_parallelism=args.max_parallelism,
    spinner=True,
)
```

The reduce phase uses another `remote_parallel_map` call to split aggregate JSON files into buckets, pickle partial counters, and merge them locally.

## Why It Matters

This is a good example of resisting unnecessary ML. A global photo index sounds like an embedding problem. In the first pass, it is a metadata and geography problem.

Burla is useful because the work is embarrassingly parallel but annoying to run by hand: thousands of shard downloads, a small reverse-geocoder cache on each worker, and shared output files. The demo keeps each worker's contract simple and leaves heavier CLIP work as an optional phase.
