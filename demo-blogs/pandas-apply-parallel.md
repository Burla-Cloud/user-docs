# Parallel pandas apply without rewriting the transform

`pandas.apply` is often where a practical data workflow slows down. The transform is already written, but it is row-oriented, string-heavy, and hard to push into SQL or Arrow expressions without changing behavior.

This demo keeps the row function. It reads a Parquet dataset from S3, partitions unique users into 1,200 chunks, runs a pandas transform per chunk on Burla workers, concatenates the returned DataFrames, and writes `enriched.parquet`.

## What The Job Does

The compromised version samples a few users and rewrites the transform as vectorized pandas. That is sometimes worth doing. It is also easy to introduce subtle differences when the existing business logic uses regexes, user-agent strings, null handling, and bucket labels.

The real version keeps the row-level enrichment and scales around it. It discovers all `user_id` values with `pyarrow.dataset`, distributes user ID slices across 1,200 tasks, and lets each worker filter the S3 Parquet dataset to its users before calling `df.apply(enrich, axis=1)`.

Each worker gets 2 CPUs and 8 GB RAM. That is larger than the simplest map jobs because each task materializes a filtered DataFrame.

## The Code Shape

The worker resets pandas string inference because pandas 3 can default to Arrow-backed strings, while downstream `.apply` and `.values` paths in this demo expect NumPy-backed behavior.

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
        return pd.Series({
            "utm_source": src.group(1) if src else None,
            "url_len": len(row["url"] or ""),
            "is_mobile": "Mobile" in (row["user_agent"] or ""),
            "revenue_bucket": "high" if row["revenue"] > 100 else "low",
        })

    enriched = df.apply(enrich, axis=1)
    return pd.concat([df, enriched], axis=1)
```

The dispatch is a single call:

```python
frames = remote_parallel_map(
    apply_on_chunk,
    chunks,
    func_cpu=2,
    func_ram=8,
    grow=True,
)

final = pd.concat(frames, ignore_index=True)
final.to_parquet("enriched.parquet")
```

## Why It Matters

The lesson is not that `apply` is ideal. The lesson is that migration pressure should match the job. If the transform is correct and the problem is wall clock, running the same function over many filtered chunks is often cheaper than rewriting and revalidating the logic.

Burla gives that code a larger execution surface while keeping the source readable to a pandas user. The tradeoff is clear: return DataFrames to the driver only when the result size is manageable.
