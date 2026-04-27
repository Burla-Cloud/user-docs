# Keep the pandas apply, scale the dataset

The compromised version rewrites the transform for Spark, drops the annoying Python bits, and then answers a simpler question. The real version runs the row function you already trust across the full Parquet dataset. If your regex, API lookup, or weird row rule changed, the experiment changed too.

This demo splits a large event table by user id, runs normal `df.apply(..., axis=1)` on each chunk, and concatenates the results.

## what we built

We use PyArrow to discover a cheap partition key, then stripe user ids across 1,200 chunks. That gives each worker a slice it can filter from the Parquet dataset.

```python
import pyarrow.dataset as ds

DATASET = "s3://my-bucket/events/"
dataset = ds.dataset(DATASET, format="parquet")
all_users = dataset.to_table(columns=["user_id"]).column("user_id").unique().to_pylist()
chunks = [all_users[i::1200] for i in range(1200)]
```

The worker reads only its users and runs plain pandas. There is no UDF dialect here.

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

## how the pipeline works

Burla treats each user-id chunk as one input. The final reduce is ordinary pandas concatenation.

```python
from burla import remote_parallel_map

frames = remote_parallel_map(apply_on_chunk, chunks, func_cpu=2, func_ram=8, grow=True)
final = pd.concat(frames, ignore_index=True)
final.to_parquet("enriched.parquet")
```

## why this demo is interesting

The useful part is that the row function survives intact. Many teams have a pandas transform that encodes years of tiny product rules: regexes, string cleanup, hand-built buckets, odd exceptions from old data. Rewriting that into Spark SQL or a vectorized expression may be possible, but it is also a new implementation with new bugs.

The infrastructure choice changes the experiment when the rewrite drops behavior. Burla lets you keep the original Python and change the amount of hardware instead. That is especially handy for one-off feature backfills, data migrations, and notebook-born transforms that need to run once over a real dataset.

## how to build your version

Choose a partition key that keeps each chunk under worker memory: user id, account id, date bucket, or a hash. Make the worker read from the source of truth, run the existing `apply`, and return a DataFrame or write a shard to storage. If the output is large, have workers write Parquet shards and merge later.

## why Burla fits

Burla removes the Ray or Dask cluster setup and the Spark rewrite. You keep the pandas code, ask for CPU and RAM per worker, and let the input chunks act as the queue. That matters for row functions full of Python objects, regexes, `json.loads`, and small external calls.

## what the rewrite misses

A compromised rewrite often drops the part that made the transform useful. The real run keeps the messy row function and applies it everywhere, so the output answers the same question you asked in the notebook.
