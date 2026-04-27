# ETL without making Airflow the center

Some ETL jobs are too simple for a workflow platform and too large for one Python process. This demo reads thousands of gzipped JSON files from S3, filters and reshapes events, and inserts rows into Postgres with `psycopg2`.

It is the kind of daily backfill that often becomes an Airflow DAG because the files are numerous. The code here keeps the work as one Python script and uses Burla only for the parallel file processing.

## What The Job Does

The compromised version loops over a small date prefix locally and writes to a test table. That checks JSON parsing and SQL shape. It does not measure the real pressure on S3, the database, or the worker memory envelope.

The real version lists every `raw/{DATE}/` `.json.gz` object in `my-events-bucket`, sends one S3 key per task, and caps parallelism at 1,000 workers to protect Postgres. Each worker downloads the object, decompresses lines, filters to `click`, `purchase`, and `signup`, uppercases country codes, converts missing amounts to zero, and writes rows with `execute_values`.

The driver uses `generator=True` so it can print progress as each file completes and maintain a running loaded-row count.

## The Code Shape

The worker owns the file and one database transaction.

```python
def etl_one_file(key: str) -> dict:
    import gzip
    import json
    import os
    import boto3
    import psycopg2
    from psycopg2.extras import execute_values

    s3 = boto3.client("s3")
    body = s3.get_object(Bucket="my-events-bucket", Key=key)["Body"].read()
    rows_in = [json.loads(line) for line in gzip.decompress(body).splitlines() if line]

    rows_out = [
        (r["event_id"], r["user_id"], r["event_type"], r["ts"], float(r.get("amount") or 0), r.get("country", "XX").upper())
        for r in rows_in
        if r.get("event_type") in ("click", "purchase", "signup")
    ]

    conn = psycopg2.connect(os.environ["DATABASE_URL"])
    with conn, conn.cursor() as cur:
        execute_values(
            cur,
            "INSERT INTO events (event_id, user_id, event_type, ts, amount, country) VALUES %s ON CONFLICT (event_id) DO NOTHING",
            rows_out,
            page_size=1000,
        )
    conn.close()
    return {"key": key, "rows_in": len(rows_in), "rows_out": len(rows_out)}
```

The map call caps writers:

```python
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
    total_rows += r["rows_out"]
```

## Why It Matters

Airflow is good at schedules and dependencies. It is not required just because a job has 10,000 files. If the dependency graph is "do this function for every object, then report totals," a map is the cleaner model.

Burla gives the script a large worker pool, while the code keeps the real controls in sight: S3 prefix, Postgres connection string, batch insert size, idempotent `ON CONFLICT`, and `max_parallelism` for database protection.
