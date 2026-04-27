# Scan every Parquet shard instead of trusting a sample

In this example we:

* List thousands of S3 Parquet files.
* Run one PyArrow worker per file.
* Return a per-file report with rows, bytes, users, revenue, and null rates.

This is a dataset audit. If you read ten files and hope the other 4,990 look the same, you have changed the question.

### Step 1: List the files

Object listing is cheap, so the client builds the work queue.

```python
import boto3

s3 = boto3.client("s3")
response = s3.list_objects_v2(Bucket="my-events-bucket", Prefix="events/2025/")
parquet_keys = [obj["Key"] for obj in response["Contents"] if obj["Key"].endswith(".parquet")]
while response.get("IsTruncated"):
    response = s3.list_objects_v2(
        Bucket="my-events-bucket",
        Prefix="events/2025/",
        ContinuationToken=response["NextContinuationToken"],
    )
    parquet_keys += [obj["Key"] for obj in response["Contents"] if obj["Key"].endswith(".parquet")]
```

### Step 2: Inspect one file per worker

The worker opens one object and returns a small dict. You can swap this for a schema check, transform, or rewrite.

```python
def scan_parquet_file(key: str) -> dict:
    import boto3
    import pyarrow.parquet as pq

    s3 = boto3.client("s3")
    obj = s3.get_object(Bucket="my-events-bucket", Key=key)
    table = pq.read_table(obj["Body"])
    return {
        "key": key,
        "rows": table.num_rows,
        "bytes": obj["ContentLength"],
        "distinct_users": table.column("user_id").combine_chunks().unique().length(),
        "revenue_sum": float(table.column("revenue").to_pandas().sum()),
    }
```

### Step 3: Build the report

Each file becomes one result row.

```python
from burla import remote_parallel_map

stats = remote_parallel_map(scan_parquet_file, parquet_keys, func_cpu=1, func_ram=4, grow=True)
report = pd.DataFrame(stats)
report.to_csv("parquet_scan_report.csv", index=False)
```

### What's the point?

Parquet datasets often fail by partition. One day has a bad writer. One customer shard has null ids. One backfill wrote timestamps in seconds while the rest wrote milliseconds.

Spark is great for many table scans, but a per-file audit is usually easier as normal Python. I want one row per object because that is the shape I need for triage.
