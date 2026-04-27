# Scan every NYC taxi month before naming ghost neighborhoods

The compromised mobility analysis samples yellow cabs or a few recent years and gets a clean story. The real run scans every public TLC monthly Parquet file across yellow, green, FHV, and HVFHS trips from 2011 through 2024. If you leave out ride-share data, you change ghosts into artifacts.

This demo processes 2,758,715,765 trips and scores all 264 TLC taxi zones by their own time series.

## what we built

Each task is one `(taxi_type, year, month)` Parquet file. The worker streams from CloudFront and returns counts by pickup zone.

```python
BASE = "https://d37ci6vzurychx.cloudfront.net/trip-data"

def monthly_url(taxi_type: str, year: int, month: int) -> str:
    return f"{BASE}/{taxi_type}_tripdata_{year}-{month:02d}.parquet"

jobs = [(taxi_type, year, month) for taxi_type in TAXI_TYPES for year, month in months]
```

```python
def process_month(job: tuple[str, int, int]) -> dict:
    import pyarrow.parquet as pq
    import requests, io

    taxi_type, year, month = job
    body = requests.get(monthly_url(taxi_type, year, month), timeout=300).content
    table = pq.read_table(io.BytesIO(body), columns=["PULocationID"])
    counts = table.column("PULocationID").to_pandas().value_counts().to_dict()
    return {"taxi_type": taxi_type, "year": year, "month": month, "counts": {int(k): int(v) for k, v in counts.items()}}
```

## how the pipeline works

Burla maps monthly files, then the client builds a zone-by-month matrix and classifies zones.

```python
from burla import remote_parallel_map

month_results = remote_parallel_map(process_month, jobs, func_cpu=1, func_ram=4, grow=True)

matrix = build_zone_month_matrix(month_results)
classified = classify_zones(matrix, zone_lookup)
write_report(classified)
```

## why this demo is interesting

Mobility data is full of reporting traps. Yellow cabs, green cabs, app-based FHVs, and high-volume FHV services do not enter the public data at the same time or with the same coverage. A compromised run over one feed can mistake a data-source transition for a neighborhood transition.

The real run keeps the time series wide enough to see both effects. It can separate Manhattan yellow-cab corridors that faded from outer-borough ride-share markets that became visible later. The method is intentionally transparent: count pickups by zone and month, then classify the shape of each zone's series.

## how to build your version

Pick the natural public shard: month files, day files, or partition paths. Return counts, not raw rows. Do the interpretive classification after the map so you can change definitions without rescanning the dataset.

## practical notes from the build

The worker returns monthly counts instead of raw trips because the classification needs time series, not individual rides. That keeps the map output small: one dict per monthly file. The client can then build a zone-by-month matrix, smooth recent means, and classify each zone without rereading CloudFront.

For another city, copy the shape rather than the labels. Use whatever public shard exists, return the smallest aggregate that preserves your question, and keep feed coverage caveats close to the classification. Mobility datasets change schema and reporting rules over time; the walkthrough should make those changes visible.

## why Burla fits

Burla removes the need for Spark or a hand-built VM fleet for a scan that is file-parallel. The worker code is PyArrow and requests; the reduce is pandas and a static report.


## ways to adapt it

For another mobility dataset, start with the question before the map. Are you looking for growth, disappearance, recovery, seasonality, or mode shift? That choice decides the aggregate. A bike-share system might need station-by-week counts. A bus dataset might need route-by-hour counts. A delivery dataset might need neighborhood-by-day counts. Burla gives you enough workers to scan the full archive, but the metric still has to match the behavior you care about.

## what the yellow-cab sample misses

The real run shows outer-borough zones becoming visible through FHV and HVFHS data. The compromised yellow-cab subset turns reporting coverage into neighborhood death and misses the second taxi system ride-share created.
