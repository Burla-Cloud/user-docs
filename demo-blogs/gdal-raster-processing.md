# Raster processing with GDAL on many workers

Raster jobs are a good test of whether parallel Python can handle real geospatial dependencies. You need GDAL, remote object storage, enough RAM for bands, and output files that preserve profile metadata.

This demo computes NDVI for Sentinel-2 tiles. Each worker reads red and near-infrared JP2 bands from the `sentinel-s2-l2a` requester-pays S3 bucket, computes `(nir - red) / (nir + red)`, writes a compressed tiled GeoTIFF to `my-ndvi-outputs`, and returns mean NDVI plus pixel count.

## What The Job Does

The compromised version computes NDVI for a few local tiles. That is the right first test because raster dependencies are fussy. It does not test the production shape: thousands of tiles, remote JP2 reads, in-memory GeoTIFF creation, and enough workers to finish before the analysis window moves on.

The real version reads `sentinel_tiles.txt`, where each line is a tile ID, and submits all IDs to Burla. Each worker gets 2 CPUs and 8 GB RAM. `rasterio` is imported at module scope so Burla installs the raster stack on workers. The worker uses `MemoryFile` for both input JP2 bytes and the output GeoTIFF, so there is no local file staging beyond memory buffers.

The output is twofold: one GeoTIFF per tile in S3 and a local `ndvi_report.csv` with per-tile summary stats.

## The Code Shape

The worker reads two bands, copies the raster profile, and writes one output object.

```python
def compute_ndvi(tile_id: str) -> dict:
    import boto3
    import numpy as np
    import rasterio
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
    with MemoryFile() as mem:
        with mem.open(**profile) as dst:
            dst.write(ndvi.astype("float32"), 1)
        s3.put_object(Bucket=DST_BUCKET, Key=f"ndvi/{tile_id}.tif", Body=mem.read())
```

The dispatch keeps one tile per task:

```python
results = remote_parallel_map(
    compute_ndvi,
    tile_ids,
    func_cpu=2,
    func_ram=8,
    grow=True,
)
```

## Why It Matters

Geospatial jobs often get pushed into specialized systems because GDAL installs and raster IO are annoying. This demo shows a simpler option for embarrassingly parallel tile work: package the Python dependencies, map over tile IDs, and let each worker write its own artifact.

Burla is useful because the compute unit matches the data unit. One tile in, one GeoTIFF out. The driver only needs a summary CSV.
