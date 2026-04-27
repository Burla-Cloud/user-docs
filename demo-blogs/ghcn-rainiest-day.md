# Find the rainiest day by scanning every NOAA year file

The compromised weather analysis downloads modern years or one region and calls the top storm good. The real run scans every GHCN-Daily year file from 1750 through today, keeps daily precipitation records, and joins station metadata after the reduce. The question changes if old years, odd stations, or quality flags disappear.

This demo processed 3,177,336,585 rows and found 1,750.0 mm at Koumac, New Caledonia on 1976-01-17.

## what we built

One worker streams one `YYYY.csv.gz` from NOAA. It keeps a top-100 heap for that year and aggregates country totals.

```python
def _stream_year_rows(year: int):
    import csv, gzip, io, requests

    url = f"https://www.ncei.noaa.gov/pub/data/ghcn/daily/by_year/{year}.csv.gz"
    with requests.get(url, stream=True, timeout=300, headers={"User-Agent": "ghcn-rainiest-day/1.0"}) as resp:
        resp.raise_for_status()
        with gzip.GzipFile(fileobj=resp.raw) as gz:
            text = io.TextIOWrapper(gz, encoding="utf-8", errors="replace", newline="")
            yield from csv.reader(text)
```

The precipitation filter is simple and visible. That matters because extreme rainfall records are sensitive to flags and units.

```python
def process_year(year: int) -> str:
    import heapq
    heap, country = [], {}
    for row in _stream_year_rows(year):
        if len(row) < 4 or row[2] != "PRCP":
            continue
        if len(row) > 5 and row[5]:
            continue
        if not row[3] or row[3] == "-9999":
            continue
        prcp_mm = int(row[3]) / 10.0
        item = (prcp_mm, row[0], row[1])
        if len(heap) < 100:
            heapq.heappush(heap, item)
        elif prcp_mm > heap[0][0]:
            heapq.heapreplace(heap, item)
```

## how the pipeline works

The map writes `/workspace/shared/ghcn/parts/{year}.json`. The reduce worker merges yearly heaps into a global top 500, joins stations, computes country-decade stats, and renders the map.

```python
years = list(range(start_year, end_year + 1))
part_paths = remote_parallel_map(process_year, years, func_cpu=1, func_ram=2, grow=True)
[result_path] = remote_parallel_map(reduce_years, [part_paths], func_cpu=8, func_ram=32, grow=True)
```

## why this demo is interesting

Extreme-weather questions punish shortcuts. A gridded product can be better for climate averages, but it will smear the point measurement you need for single-day records. A modern-only station subset can be cleaner, but it removes older records and hides network changes. The real archive is messy, and that mess is part of the result.

The map/reduce shape fits the data source exactly. NOAA already publishes one compressed CSV per year. Each map worker streams one file once, keeps a small heap, and writes a compact part. The reducer can then change ranking rules, station dedupe, or decade filters without downloading three billion rows again.

## how to build your version

Make the public shard the unit of work: one year, one station file, one day partition. Keep the filtering rule in code, not in prose. Write map outputs to shared storage so you can rerun the reduce with different ranking logic.

## practical notes from the build

The reducer keeps two views because extremes are tricky. The raw top 500 shows every record, including station clusters. The station-deduped view asks a different question: what is each station's wettest day? Keeping both views avoids hiding repeated Koumac records while still giving a cleaner geographic leaderboard.

For a similar climate scan, write down the unit conversion and quality filters in code. Here `PRCP` is tenths of millimeters, empty quality flag is required, and `MDPR` multi-day totals are excluded. Those details decide the result.

## why Burla fits

Burla removes the year-file queue, worker fleet, and shared artifact wiring. It also keeps the reducer as Python, which is handy when the output is a leaderboard plus a map rather than a table scan.


## ways to adapt it

The same plan works for temperature records, wind gusts, river gauges, air-quality monitors, or any public archive split by time. Put the per-file filter in the worker, keep a heap or compact aggregate, and write parts to shared storage. Then rerun the reduce for alternate definitions: station-deduped, country-ranked, decade-ranked, or flagged-only views.

## what the modern subset misses

The real run shows Koumac clustering, Maui windward stations, and early-data artifacts. The compromised modern subset misses both the record and the caveats that tell you how much to trust it.
