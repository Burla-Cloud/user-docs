# Find the rainiest day by scanning every NOAA year file

In this example we:

* Scan every GHCN-Daily year file from 1750 through today.
* Keep daily precipitation records with clean quality flags.
* Reduce 3,177,336,585 rows into a global rain leaderboard.

The run found 1,750.0 mm at Koumac, New Caledonia on 1976-01-17.

### Step 1: Stream one year per worker

NOAA publishes one compressed CSV per year. That makes the input list obvious.

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

### Step 2: Keep a heap, not the whole file

The worker filters precipitation rows, applies the unit conversion, and keeps only the top records for that year.

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

### Step 3: Reduce the years

The reducer merges yearly heaps, joins station metadata, computes country-decade stats, and renders the map.

```python
years = list(range(start_year, end_year + 1))
part_paths = remote_parallel_map(process_year, years, func_cpu=1, func_ram=2, grow=True)
[result_path] = remote_parallel_map(reduce_years, [part_paths], func_cpu=8, func_ram=32, grow=True)
```

### What's the point?

Extreme weather questions punish shortcuts. A clean modern subset can miss the actual record. A gridded product can be better for averages, but it smears the point measurement you need here.

The important thing is that the filtering rule is code, not prose: `PRCP`, tenths of millimeters, empty quality flag, no missing value. Those details decide the result.
