# Scanning thousands of Parquet files

Many analytics jobs are simple per-file scans hiding under a large file count. You have daily partitions, user shards, or event logs in S3. For each file you need row counts, byte sizes, distinct users, revenue sums, null rates, or schema checks.

This demo lists Parquet files under an S3 prefix, sends one file key per worker, reads each file with PyArrow, returns a small stats dict, and writes `parquet_scan_report.csv`.

## What The Job Does

The compromised version reads a hundred files locally and estimates the rest. That can catch obvious schema problems, but it will miss the one bad shard, the weird null pocket, or the partition whose revenue sum is off.

The real version scans every file. It paginates `s3.list_objects_v2` under `events/2025/`, keeps keys ending in `.parquet`, and sends the list to Burla. Each worker calls `s3.get_object`, passes the body to `pq.read_table`, computes a few stats, and returns one dict.

The interesting constraint is result size. Returning one small dict per file is fine. Returning file contents would be a mistake. The remote workers do the scan, and the driver only builds a pandas report from stats.

## The Code Shape

The worker is intentionally one file wide.

```python
def scan_parquet_file(key: str) -> dict:
    import boto3
    import pyarrow.parquet as pq

    s3 = boto3.client("s3")
    obj = s3.get_object(Bucket=BUCKET, Key=key)
    table = pq.read_table(obj["Body"])

    return {
        "key": key,
        "rows": table.num_rows,
        "bytes": obj["ContentLength"],
        "distinct_users": table.column("user_id").combine_chunks().unique().length(),
        "revenue_sum": float(table.column("revenue").to_pandas().sum()),
        "null_user_rate": table.column("user_id").null_count / max(table.num_rows, 1),
    }
```

The Burla call maps that over every key:

```python
stats = remote_parallel_map(
    scan_parquet_file,
    parquet_keys,
    func_cpu=1,
    func_ram=4,
    grow=True,
)

df = pd.DataFrame(stats)
df.to_csv("parquet_scan_report.csv", index=False)
```

## Why It Matters

This is one of the cleanest uses of distributed compute: each file is independent, work per file is bounded, and the result per file is tiny. The code does not need a distributed DataFrame API.

Burla is useful because it turns a slow serial loop into a map over object keys. The demo keeps the storage nouns visible: S3 bucket, prefix, Parquet key, PyArrow table, CSV report.
