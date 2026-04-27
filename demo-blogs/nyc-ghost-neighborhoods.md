# Ghost neighborhoods in NYC taxi data

NYC taxi data is one of the best public datasets for watching a city change. It is also large enough to punish lazy analysis. Yellow taxis, green taxis, and high-volume rideshare files cover roughly 3 billion trips across more than a decade of monthly Parquet files.

This demo asks which taxi zones became ghost neighborhoods, which zones were born late, and which zones collapsed then recovered. The output is a zone-by-month matrix, a choropleth SVG, and leaderboards with sparkline histories.

## What The Job Does

The compromised version reads a few recent months or one taxi type. That is enough to debug schemas. It is not enough to separate a pandemic trough from a real long-term shift, and it misses the handoff from yellow taxis to Uber and Lyft.

The real version processes every monthly file in the Hugging Face mirror `DinoPonjevic/NYC_TaxiData_RAW`: yellow from 2011 through 2024, green from 2014 through 2024, and high-volume FHV from 2019 onward. Workers stream one monthly Parquet at a time, project it down to pickup zone counts, and return a compact result. A 500 MB rideshare month with tens of millions of trips becomes at most 263 zone rows before it crosses the network.

The driver then reduces those per-month dicts into a `zone x month` matrix, classifies zones as ghost, cooling, stable, warming, or emergent, and renders a local HTML report.

## The Code Shape

The worker does not return trip rows. It opens the monthly Parquet from Hugging Face, finds the pickup zone column despite schema drift, counts zones in Arrow batches, and returns only counts.

```python
def process_month(task_id: str) -> dict:
    url = _hf_url_for_task(task_id)
    resp = requests.get(url, headers=headers, timeout=300, allow_redirects=True)
    resp.raise_for_status()

    pf = pq.ParquetFile(pa.BufferReader(resp.content))
    zone_col = _find_col(schema_names, PU_ZONE_COL_CANDIDATES)
    counts = defaultdict(int)

    for batch in pf.iter_batches(batch_size=500_000, columns=read_cols):
        zone_np = batch.column(zone_col).to_numpy(zero_copy_only=False)
        zone_int = np.asarray(zone_np, dtype=np.float64)
        valid = ~np.isnan(zone_int) & (zone_int > 0) & (zone_int < 1000)
        uniq, cnts = np.unique(zone_int[valid].astype(np.int32), return_counts=True)
        for zi, ci in zip(uniq.tolist(), cnts.tolist()):
            counts[int(zi)] += int(ci)

    return {"task": task_id, "counts": sorted([[int(k), int(v)] for k, v in counts.items()])}
```

The Burla call is plain:

```python
results = list(remote_parallel_map(
    process_month,
    tasks,
    func_cpu=1,
    func_ram=4,
    **kwargs,
))
```

The reduce phase stays local because the map phase has already collapsed billions of trips to a small matrix.

## Why It Matters

This demo is useful for data engineers because it shows where the real boundary is. The hard part is not drawing a map. It is getting each worker to stream a big Parquet file, handle old column names, aggregate locally, and avoid sending raw rows back.

Burla is the proof that this can stay as a Python file instead of becoming a Spark job. The result is still a normal data artifact: counts, rankings, SVG paths, and HTML. The cluster only changes the wall clock.
