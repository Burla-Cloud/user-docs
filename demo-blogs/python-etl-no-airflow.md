# Run the file-drop ETL before it becomes a platform project

The compromised ETL loops through yesterday's files from one machine, then someone opens an Airflow ticket because it is slow. The real job is simpler: transform every file in the drop, write to Postgres, and cap database concurrency. If you changed the job into an orchestration project, you changed the experiment.

This demo processes 10,000 gzipped JSON files from S3 and loads cleaned rows into Postgres.

## what we built

The client lists the daily prefix and builds one input per file.

```python
import boto3

BUCKET = "my-events-bucket"
DATE = "2025-04-19"

keys = []
for page in boto3.client("s3").get_paginator("list_objects_v2").paginate(Bucket=BUCKET, Prefix=f"raw/{DATE}/"):
    keys += [obj["Key"] for obj in page.get("Contents", []) if obj["Key"].endswith(".json.gz")]
```

The worker handles extract, transform, and load for one object. It uses `execute_values` so each file becomes one batched insert.

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

## how the pipeline works

The database is protected by `max_parallelism`. Results stream back so the terminal can show progress.

```python
from burla import remote_parallel_map

done = 0
for r in remote_parallel_map(
    etl_one_file, keys, func_cpu=1, func_ram=2,
    max_parallelism=1000, generator=True, grow=True,
):
    done += 1
    if done % 100 == 0:
        print(done, r["rows_out"])
```

## why this demo is interesting

Airflow is useful when a pipeline has durable schedules, dependencies, owners, and reruns. A lot of ETL work is smaller: a daily file drop, a backfill, or a migration that needs parallelism more than a platform. The compromised version either runs too slowly on one box or grows into a DAG system before the data question is answered.

The real experiment is sink-aware. Transforming 10,000 files in parallel is easy; loading them without flattening Postgres is the part that matters. That is why `max_parallelism` belongs in the walkthrough. It turns the database connection pool into a first-class constraint instead of an afterthought.

## how to build your version

Make one worker own one file or one partition. Keep the load idempotent with `ON CONFLICT`, merge keys, or output partitions. Pick the concurrency from the sink: Postgres connection pool, warehouse write slots, or API quota.

## why Burla fits

Burla removes the scheduler, metadata database, DAG deploy path, Batch queue, and worker fleet. A daily cron or CI job can run the Python file and still fan out across cloud machines.

## what the local loop misses

The compromised loop tests transform correctness. The real run tests sink pressure, duplicate behavior, and every malformed file in the drop. That is where ETL breaks.
