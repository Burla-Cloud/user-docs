# The rainiest day in NOAA's station archive

NOAA GHCN-Daily is a simple dataset at an awkward size. Each year is a gzipped CSV of station observations, and the full history reaches back to the 1700s. The question sounds easy: what is the largest single-day rainfall record, and which countries are wettest by decade?

The demo answers that by scanning one year file per worker. Each map task streams `YYYY.csv.gz` from NOAA, filters PRCP rows, keeps the year's top rainfall observations, aggregates per-country stats, and writes a JSON part file. A reduce task merges parts, builds leaderboards, writes decade CSVs, and renders a Leaflet map.

## What The Job Does

The compromised version processes the last few years or a single country. That is fine for the parser and map style. It cannot answer a historical "ever" question, and it misses country-decade comparisons because station coverage changes over time.

The real version submits every year from `GHCN_START_YEAR` through `GHCN_END_YEAR`. Workers read from `https://www.ncei.noaa.gov/pub/data/ghcn/daily/by_year/{year}.csv.gz`, discard quality-flagged observations, convert tenths of millimeters to millimeters, and keep top records in a heap. They also aggregate total precipitation and observation counts by country code.

Station and country metadata are staged once to `/workspace/shared/ghcn/meta` when bundled snapshots exist. That avoids every reduce run hitting NOAA for the same station lookup.

## The Code Shape

The worker is intentionally narrow: stream one year, emit one part.

```python
def process_year(year: int) -> str:
    rows_seen = 0
    prcp_valid = 0
    heap = []
    country = {}

    for row in _stream_year_rows(year):
        rows_seen += 1
        if len(row) < 4 or row[2] != "PRCP":
            continue
        qflag = row[5] if len(row) > 5 else ""
        if qflag:
            continue
        raw = row[3]
        if not raw or raw == "-9999":
            continue
        prcp_mm = int(raw) / 10.0
        sid = row[0]

        item = (prcp_mm, sid, row[1], mflag, sflag, obs_time)
        if len(heap) < TOP_PER_YEAR:
            heapq.heappush(heap, item)
        elif prcp_mm > heap[0][0]:
            heapq.heapreplace(heap, item)

    return _write_part(year, part)
```

The driver uses two Burla calls: one for the map, one for the reduce.

```python
part_paths = remote_parallel_map(
    process_year,
    years,
    func_cpu=1,
    func_ram=4,
)

results_dir = remote_parallel_map(
    reduce_years,
    [list(part_paths)],
    func_cpu=8,
    func_ram=32,
)
```

The reduce writes `top_500.csv`, `top_by_station.csv`, `country_decade_stats.csv`, decade markdown summaries, `run_summary.json`, and `map.html`.

## Why It Matters

Climate datasets are often a pile of plain files. The bottleneck is reading them all, not inventing new algorithms. This demo shows a practical pattern for that case: one file per worker, small JSON part files, one reduce task with enough RAM.

Burla does not change the math. It turns a long serial scan into a short distributed scan while leaving the source code as normal Python: `gzip`, `csv`, `heapq`, `folium`, and `remote_parallel_map`.
