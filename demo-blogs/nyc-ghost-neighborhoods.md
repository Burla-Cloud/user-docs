# Scan every NYC taxi month

In this example we:

* Download every public TLC monthly Parquet file from 2011 through 2024.
* Count pickups for yellow, green, FHV, and HVFHS trips.
* Score all 264 taxi zones by their own time series.

I wanted to find ghost neighborhoods, but only after including the ride-share data. Otherwise you are mostly measuring the limits of yellow cab coverage.

### Step 1: Make one task per month file

The public data is already split by taxi type, year, and month. That becomes the input list.

```python
BASE = "https://d37ci6vzurychx.cloudfront.net/trip-data"

def monthly_url(taxi_type: str, year: int, month: int) -> str:
    return f"{BASE}/{taxi_type}_tripdata_{year}-{month:02d}.parquet"

jobs = [(taxi_type, year, month) for taxi_type in TAXI_TYPES for year, month in months]
```

### Step 2: Count pickups on the worker

Each worker downloads one Parquet file and returns pickup counts by zone. It does not send raw trips back to the client.

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

### Step 3: Build the time series

The client reduces the monthly counts into a zone-by-month matrix and classifies the shape of each zone.

```python
from burla import remote_parallel_map

month_results = remote_parallel_map(process_month, jobs, func_cpu=1, func_ram=4, grow=True)

matrix = build_zone_month_matrix(month_results)
classified = classify_zones(matrix, zone_lookup)
write_report(classified)
```

### What's the point?

Mobility data is full of traps. Yellow cabs, green cabs, app-based FHVs, and high-volume FHVs do not appear in the public data at the same time. If you scan one feed, you can mistake a reporting change for a neighborhood change.

This version keeps the question honest. Count every month, keep the output small, and do the interpretation after the scan. Then you can change the ghost definition without redownloading 2.76 billion trips.
