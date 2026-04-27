# Run the file-drop ETL before it becomes a platform project

In this example we:

* List 10,000 gzipped JSON files from S3.
* Transform each file in a Burla worker.
* Load cleaned rows into Postgres while capping database concurrency.

This is the kind of job that turns into an Airflow ticket when the actual problem is just: yesterday's files are too slow on one machine.

### Step 1: List the files

The client lists the daily prefix and builds one input per file.

```python
import boto3

BUCKET = "my-events-bucket"
DATE = "2025-04-19"

keys = []
for page in boto3.client("s3").get_paginator("list_objects_v2").paginate(Bucket=BUCKET, Prefix=f"raw/{DATE}/"):
    keys += [obj["Key"] for obj in page.get("Contents", []) if obj["Key"].endswith(".json.gz")]
```

### Step 2: Transform and insert one file

The worker owns extract, transform, and load for one object. `execute_values` keeps each file as one batched insert.

```python
def etl_one_file(key: str) -> dict:
    import gzip, json, os, boto3, psycopg2
    from psycopg2.extras import execute_values

    body = boto3.client("s3").get_object(Bucket="my-events-bucket", Key=key)["Body"].read()
    rows_in = [json.loads(line) for line in gzip.decompress(body).splitlines() if line]
    rows_out = [
        (r["event_id"], r["user_id"], r["event_type"], r["ts"], float(r.get("amount") or 0))
        for r in rows_in
        if r.get("event_type") in ("click", "purchase", "signup")
    ]
    conn = psycopg2.connect(os.environ["DATABASE_URL"])
    with conn, conn.cursor() as cur:
        execute_values(cur, "INSERT INTO events VALUES %s ON CONFLICT DO NOTHING", rows_out, page_size=1000)
    conn.close()
    return {"key": key, "rows_in": len(rows_in), "rows_out": len(rows_out)}
```

### Step 3: Protect Postgres

The database is the constraint, so `max_parallelism` is the important line.

```python
from burla import remote_parallel_map

done = 0
for r in remote_parallel_map(
    etl_one_file,
    keys,
    func_cpu=1,
    func_ram=2,
    max_parallelism=1000,
    generator=True,
    grow=True,
):
    done += 1
    if done % 100 == 0:
        print(done, r["rows_out"])
```

### What's the point?

Transforming 10,000 files in parallel is easy. Loading them without flattening Postgres is the part that matters.

That is why I like this shape. The Python stays boring, the insert stays idempotent, and the sink gets a real concurrency cap. You can put this behind cron or CI without adopting a workflow platform for one file drop.
