---
description: With <30 lines of Python.
---

# Process 2.4TB in Parquet Files in 48s

In this example we:

* Generate 1,000 billion-row Parquet files (2.4TB) and store them in Google Cloud Storage.
* Run a DuckDB query on each file in parallel using a cluster with 10,000 CPUs.
* Combine resulting data locally.

### What is the Trillion row Challenge?

An extension of the [billion row challenge](https://github.com/gunnarmorling/1brc), the goal of the trillion row challenge is to compute the min, max, and mean temperature per location, for 413 unique locations, from data stored as a collection of parquet files in blob storage. Data looks like this (but with 1,000,000,000,000 rows):

<figure><img src="../.gitbook/assets/CleanShot 2025-11-25 at 12.59.38.png" alt=""><figcaption></figcaption></figure>

### Cluster settings:

For this challenge we use used a 125 node cluster with 80 cpus and 320G ram per node.\
Underneath these are N4-standard-80 machines. The cluster settings look like this:

<figure><img src="../.gitbook/assets/CleanShot 2025-11-25 at 16.55.00.png" alt=""><figcaption></figcaption></figure>

This cluster took 1min 37s to boot.

### Generating the dataset:

We chose to split the 1T row dataset into 1,000 parquet files.\
With 10k cpus, this means we only need to download one file per 10-cpu worker.

Once generated, parquet files are written to the \`./shared\` folder. Anything placed in this folder is synchronized with a Google Cloud Storage bucket using GCSFuse. The dashboard has a built in file manager where you can monitor, upload, and download files from the cluster.

Below, each call to `generate_parquet` is done in a `python:3.12` docker container (see settings ↑), this can be any docker container as long as your VM's service account is authorized to pull it.\
Python packages your code uses that aren't in the container are detected and installed quickly at runtime. The code below took 3s to install pandas, numpy, and pyarrow in every container.

```python
import pyarrow
import numpy as np
import pandas as pd
from burla import remote_parallel_map

TOTAL_ROWS = 1_000_000_000_000
ROWS_PER_FILE = 1_000_000_000

lookup_df = pd.read_csv("avg_temp_per_station.csv")
file_nums = range(TOTAL_ROWS // ROWS_PER_FILE)

def generate_parquet(file_num: int):
    rng = np.random.default_rng(seed=file_num)

    # array of station names
    station_indices = rng.integers(0, len(lookup_df) - 1, ROWS_PER_FILE)
    station_names = lookup_df.station.to_numpy()[station_indices]

    # array of station temps, aligned by index with `station_names`
    station_temps = rng.normal(0, 10, ROWS_PER_FILE)
    station_temps += lookup_df.mean_temp.to_numpy()[station_indices]
    station_temps = station_temps.round(1)

    # save to parquet in cluster filesystem
    df = pd.DataFrame({"station": station_names, "temp": station_temps})
    df.to_parquet(f"shared/1TRC/{file_num}.parquet", engine="pyarrow")

remote_parallel_map(generate_parquet, file_nums, func_ram=64)
```

This code completed in 6m 39s, and the final dataset is 2.4TB.\
After running files appear under the "Filesystem" tab (underneath this is a GCS Bucket).

<figure><img src="../.gitbook/assets/CleanShot 2025-11-26 at 16.08.20.png" alt=""><figcaption></figcaption></figure>

### Running the challenge!

This code runs a simple DuckDB query on all 1,000 Parquet files at the same time.\
Each query returns a pandas dataframe with the min/mean/max per station within that file.\
DataFrames are concatenated, and aggregated locally to produce a final min/mean/max temperature per weather station.

Like the data generation code above, every call to `station_stats` runs in a `python:3.12` docker container, and any missing python packages are quickly installed at runtime. Parquet files pulled from the \`./shared\` folder are actually being downloaded from Google Cloud Storage using GCSFuse.

To avoid using any cached files/data the cluster was rebooted before running the code below.

```python
import duckdb
import pandas as pd
from burla import remote_parallel_map

TOTAL_ROWS = 1_000_000_000_000
ROWS_PER_FILE = 1_000_000_000
file_nums = range(TOTAL_ROWS // ROWS_PER_FILE)

def station_stats(file_num: int):
    query = """
    SELECT
        station,
        MIN(temp) AS min_temp,
        AVG(temp) AS mean_temp,
        MAX(temp) AS max_temp
    FROM read_parquet(?)
    GROUP BY station
    ORDER BY station
    """
    con = duckdb.connect(database=":memory:")
    con.execute("PRAGMA threads=10")
    return con.execute(query, (f"shared/1TRC/{file_num}.parquet",)).df()

dataframes = remote_parallel_map(station_stats, file_nums, func_cpu=10)
df = pd.concat(dataframes).groupby("station").agg(
    min_value=("min_temp", "min"),
    mean_value=("mean_temp", "mean"),
    max_value=("max_temp", "max"),
)
print(df)
```

This only took 48s to finish, which I believe is technically a new record!\
However, as I talk more about below, I don't actually think runtime is what really matters most.\
I also believe there's a clear path to a <5s runtime using this same hardware, which would be fun to explore :)

Output:

```bash
✔ Done! Ran 1000 inputs through `station_stats` (1000/1000 completed)         
              name  min_value  mean_value  max_value
0             Abha      -46.9   18.000151       84.2
1          Abidjan      -34.1   26.000040       89.9
2           Abéché      -30.9   29.400059       91.5
3            Accra      -36.1   26.399808       88.7
4      Addis Ababa      -46.5   15.999770       81.7
..             ...        ...         ...        ...
407       Yinchuan      -54.2    9.000354       75.5
408         Zagreb      -52.6   10.700003       75.1
409  Zanzibar City      -36.0   25.999645       91.1
410         Ürümqi      -53.8    7.399437       69.3
411          İzmir      -40.8   17.900016       80.9

[412 rows x 4 columns]
```

### How expensive was this?

For this demo we used spot instances. These are extra machines rented at a discount that might be taken back ("preempted") at anytime when Google needs them. This isn't an issue for us because Burla jobs continue working even if some nodes are preempted while in the middle of a job.

Our cluster used 125 N4-standard-80 machines, took 1m37s to boot, and was shut down 1min later.\
To be safe let's assume the billable runtime per node was 3min, with 125 nodes this is 6.25 hours.

At spot pricing N4-standard-80 machines cost $1.22/hour meaning this job cost about $7.63.

### Can this be faster? What's the point?

48s is a respectable time, but the real question I'm trying to answer is:\
**If I were in the office on a busy day, and needed to process a bunch of stuff, what would I do?**\
**How long would it take? and how expensive would it be?**

Let's be honest, this isn't the most compute-efficient solution in the world. Most of the time is spent downloading data, GCSFuse is easy to use, but it isn't maxing out the network capacity of the VM.\
\
But if I were in the office, and you asked me to get you the min/mean/max per station, assuming I'd never heard of this challenge before, I'd have an answer for you probably <5 minutes later, and for less than $10. In my opinion this is the real result, and I think it's an impressive one!

### Bonus: I think this is possible in <5s (without cheating)

Using a cluster this size, I think a time of <5s is possible (including data-download from GCS).

Here's how I would do this in <5s:

Definitely! But I think hyperoptimizing violates the spirit of&#x20;





