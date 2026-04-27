# Process every raster tile, not a pretty subset

In this example we:

* Process 2,000 Sentinel-2 tiles.
* Read red and near-infrared bands from S3.
* Compute NDVI and write a report with per-tile stats.

The first tile usually works. The full region is where missing bands, bad nodata values, CRS surprises, and requester-pays mistakes show up.

### Step 1: Make one task per tile

The input is just a list of tile ids.

```python
SRC_BUCKET = "sentinel-s2-l2a"
DST_BUCKET = "my-ndvi-outputs"

with open("sentinel_tiles.txt") as f:
    tile_ids = [line.strip() for line in f if line.strip()]
```

### Step 2: Compute NDVI in the worker

The worker reads both bands, computes NDVI, and returns summary stats.

```python
def compute_ndvi(tile_id: str) -> dict:
    import boto3, numpy as np, rasterio
    from rasterio.io import MemoryFile

    s3 = boto3.client("s3", region_name="eu-central-1")
    def read_band(band: str):
        key = f"tiles/{tile_id}/{band}.jp2"
        body = s3.get_object(Bucket=SRC_BUCKET, Key=key, RequestPayer="requester")["Body"].read()
        with MemoryFile(body) as mem, mem.open() as src:
            return src.read(1).astype("float32"), src.profile

    red, profile = read_band("B04")
    nir, _ = read_band("B08")
    ndvi = (nir - red) / (nir + red + 1e-6)
    profile.update(driver="GTiff", dtype="float32", count=1, compress="DEFLATE", tiled=True)
    return {"tile_id": tile_id, "mean_ndvi": float(ndvi.mean()), "pixels": int(ndvi.size)}
```

### Step 3: Run the tiles

Each tile gets two CPUs and enough RAM for the bands.

```python
from burla import remote_parallel_map

results = remote_parallel_map(compute_ndvi, tile_ids, func_cpu=2, func_ram=8, grow=True)
pd.DataFrame(results).to_csv("ndvi_report.csv", index=False)
```

### What's the point?

A pretty subset can produce a convincing map and still miss the data-quality problem.

For geospatial work, I want one task to own one tile, scene, or chip group. Keep the source reads and output writes inside the worker. Return enough stats that the report can catch suspicious tiles before they quietly enter a model.
