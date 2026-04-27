# Process every raster tile, not a pretty subset

The compromised geospatial run clips a handful of tiles and produces a convincing map. The real run touches every Sentinel or Landsat tile in the region, where CRS surprises, missing bands, and bad nodata values show up. If you sampled the easy tiles, you ran a smaller experiment.

This demo computes NDVI for 2,000 Sentinel-2 tiles. Each worker reads red and near-infrared bands from S3, writes a compressed GeoTIFF, and returns summary stats.

## what we built

The input is a list of tile ids. The worker builds S3 keys from each id and keeps the GDAL work inside `rasterio`.

```python
SRC_BUCKET = "sentinel-s2-l2a"
DST_BUCKET = "my-ndvi-outputs"

with open("sentinel_tiles.txt") as f:
    tile_ids = [line.strip() for line in f if line.strip()]
```

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

## how the pipeline works

Burla gives each tile two CPUs and enough RAM for the bands.

```python
from burla import remote_parallel_map

results = remote_parallel_map(compute_ndvi, tile_ids, func_cpu=2, func_ram=8, grow=True)
pd.DataFrame(results).to_csv("ndvi_report.csv", index=False)
```

## why this demo is interesting

Raster jobs look simple in a notebook because the first tile usually works. The pain arrives when the full region includes different projections, missing assets, requester-pays buckets, and files that GDAL can open but your downstream model cannot use. The real experiment is the one that touches the ugly tiles.

This pattern is also useful for building ML training data. Swap NDVI for cloud masking, chip extraction, or COG conversion. Each worker owns the source reads and writes outputs beside the data. The final report lets you sort by mean value, pixel count, failures, or suspicious distributions before you train anything.

## how to build your version

Make one task equal one tile, scene, or chip group. Keep source reads and destination writes inside the worker. For reprojection or `gdalwarp`, use a custom image with the exact native packages you need. Return summary metrics so the reduce step can catch failed or suspicious tiles.

## why Burla fits

Geospatial work often gets stuck on packaging and orchestration, not math. Burla removes the AMI, queue, Batch definition, and per-node GDAL install. Your worker can use `rasterio`, PROJ data, requester-pays buckets, and cloud writes directly.

## what the subset misses

The real continent-scale run finds the tiles with missing bands and the regions where the NDVI distribution is wrong. The compromised subset finds a nice image and misses the data-quality problem.
