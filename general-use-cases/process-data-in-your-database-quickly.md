---
description: A beginner-friendly way to process database rows in parallel by splitting work into ID ranges.
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: false
---

# Process data in your database quickly.

If your table has millions of rows, one long query loop is usually slow.

A simple faster pattern is:

1. split rows into many ID ranges
2. process each range in parallel
3. combine range results

This pattern works best with a numeric column you can split into ranges, such as an indexed `id` column.

## Before you start

Make sure you have already:

1. installed Burla: `pip install burla`
2. connected your machine: `burla login`
3. started your cluster in the Burla dashboard

For this example, also install a PostgreSQL driver:

4. `pip install psycopg2-binary`

## Step 1: Decide your row ranges

Start with ranges that do not overlap.

```python
def build_id_ranges(start_id, end_id, rows_per_range):
    return [
        (range_start_id, min(range_start_id + rows_per_range - 1, end_id))
        for range_start_id in range(start_id, end_id + 1, rows_per_range)
    ]


id_ranges = build_id_ranges(start_id=1, end_id=100_000, rows_per_range=10_000)
```

## Step 2: Write one function that processes one range

Each function call opens its own database connection and handles one ID range.

```python
import psycopg2


def process_id_range(id_range):
    start_id, end_id = id_range

    with psycopg2.connect(
        host="localhost",
        dbname="app",
        user="app",
        password="app",
    ) as connection:
        with connection.cursor() as cursor:
            cursor.execute(
                "SELECT amount FROM orders WHERE id BETWEEN %s AND %s",
                (start_id, end_id),
            )
            amounts = [row[0] for row in cursor.fetchall()]

    return {"row_count": len(amounts), "total_amount": float(sum(amounts))}
```

## Step 3: Run all ranges in parallel

Pass the list of ranges to `remote_parallel_map`.

```python
from burla import remote_parallel_map

range_results = remote_parallel_map(process_id_range, id_ranges)
```

## Step 4: Combine the range results

Now compute one final total from all range outputs.

```python
total_rows = sum(range_result["row_count"] for range_result in range_results)
total_amount = sum(range_result["total_amount"] for range_result in range_results)

print(f"Total rows processed: {total_rows}")
print(f"Total amount: {total_amount}")
```

## Step 5: Run a small test before the full job

Always test first with a small ID window.

```python
from burla import remote_parallel_map

small_test_ranges = build_id_ranges(start_id=1, end_id=5_000, rows_per_range=1_000)
remote_parallel_map(process_id_range, small_test_ranges)
```

After small tests succeed, run your full range list.
