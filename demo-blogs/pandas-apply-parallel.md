# Keep the pandas apply, scale the dataset

In this example we:

* Split a Parquet event table by user id.
* Run normal `df.apply(..., axis=1)` on each worker.
* Concatenate the enriched chunks into one output dataset.

Sometimes the ugly pandas row function is the business logic. Rewriting it into Spark just to make it run faster can quietly change the answer.

### Step 1: Pick a partition key

Here we stripe user ids across 1,200 chunks. Each worker can filter the Parquet dataset down to its users.

```python
import pyarrow.dataset as ds

DATASET = "s3://my-bucket/events/"
dataset = ds.dataset(DATASET, format="parquet")
all_users = dataset.to_table(columns=["user_id"]).column("user_id").unique().to_pylist()
chunks = [all_users[i::1200] for i in range(1200)]
```

### Step 2: Run plain pandas

The worker reads its slice and applies the same row function you would run locally.

```python
def apply_on_chunk(user_ids: list[str]) -> pd.DataFrame:
    import re
    import pandas as pd
    import pyarrow.dataset as ds

    pd.set_option("future.infer_string", False)
    dataset = ds.dataset("s3://my-bucket/events/", format="parquet")
    df = dataset.filter(ds.field("user_id").isin(user_ids)).to_table().to_pandas()

    utm_re = re.compile(r"utm_source=([^&]+)")
    def enrich(row):
        src = utm_re.search(row["url"] or "")
        return pd.Series({"utm_source": src.group(1) if src else None, "url_len": len(row["url"] or "")})

    return pd.concat([df, df.apply(enrich, axis=1)], axis=1)
```

### Step 3: Concatenate the results

Burla treats each user-id chunk as one input.

```python
from burla import remote_parallel_map

frames = remote_parallel_map(apply_on_chunk, chunks, func_cpu=2, func_ram=8, grow=True)
final = pd.concat(frames, ignore_index=True)
final.to_parquet("enriched.parquet")
```

### What's the point?

A lot of useful transforms are full of regexes, JSON parsing, weird buckets, and old product rules. They are not beautiful. They are just correct.

This lets you keep that code and change the amount of hardware underneath it. For a one-off backfill or migration, that is often the most honest thing to do.
