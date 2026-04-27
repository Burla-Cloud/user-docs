# Scan every Parquet shard instead of trusting a sample

The compromised version reads ten Parquet files, squints at the summary stats, and hopes the other 4,990 files look the same. The real version scans every shard and finds the broken partition, the weird schema, the null burst, or the revenue spike. If you asked for a dataset audit, sampling quietly changed the experiment.

This demo scans thousands of S3 Parquet files with one worker per file. Each worker returns a small report: rows, bytes, distinct users, revenue, and null rate.

## what we built

First we list the files. This stays on the client because object listing is cheap and gives us the work queue.

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

The worker opens one object and uses PyArrow to inspect it. You can swap the body for a transform, a schema check, or a rewrite.

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

## how the pipeline works

The Burla call is the dispatcher. Each key is one input. Results come back as Python dicts and become a DataFrame report.

```python
from burla import remote_parallel_map

stats = remote_parallel_map(scan_parquet_file, parquet_keys, func_cpu=1, func_ram=4, grow=True)
report = pd.DataFrame(stats)
report.to_csv("parquet_scan_report.csv", index=False)
```

## why this demo is interesting

Parquet datasets often fail by partition, not by schema alone. One day has a bad writer version. One customer shard has null ids. One backfill wrote timestamps in seconds while the rest wrote milliseconds. A compromised sample almost always misses that because the bug is sparse. The real scan gives you one row per file, which is exactly the shape a data engineer needs for triage.

This also keeps the audit close to the storage layout. You are not asking Spark to build one logical table and hide file identity. You are asking a sharper question: what is true about each object in the bucket? That is a better fit for PyArrow plus a map over keys.

## how to build your version

Pick the file-level invariant you care about: row counts, min/max timestamps, primary-key uniqueness, compression, schema drift, or per-file aggregates. Keep the worker pure: input key in, report dict out. If each file is large, give the worker more RAM. If each file is small, batch several keys per worker.

## why Burla fits

Spark can scan Parquet, but a per-file audit is often easier as Python. AWS Batch makes you define a job, container, queue, and retry policy before you can answer the question. Burla turns the S3 keys into a live queue and runs the exact PyArrow code you would have written in a notebook.

## what the small run misses

The bad file is usually not in the first ten. The real scan catches the one partition with a null `user_id` explosion. The compromised scan gives you a clean-looking report and leaves the broken day in production.
