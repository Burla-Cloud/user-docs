---
description: A beginner-friendly pattern for processing many files in parallel.
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

# Process thousands of files quickly.

If you have a lot of files, the fastest pattern is usually:

1. give each file to one function call
2. run many function calls in parallel
3. save each output to `/workspace/shared`

This keeps your script simple and makes it easy to scale up.

## Before you start

Make sure you have already:

1. installed Burla: `pip install burla`
2. connected your machine: `burla login`
3. started your cluster in the Burla dashboard

If you are new to the shared filesystem, read [Read and Write GCS Files](../common-patterns/read-and-write-gcs-files.md).

## Step 1: Build a list of file paths

Start with a list of input files.

```python
from pathlib import Path

input_directory = Path("/workspace/shared/logs/raw")
input_file_paths = sorted(str(path) for path in input_directory.glob("*.txt"))

print(f"Found {len(input_file_paths)} files")
print(input_file_paths[:3])
```

Each path in the list becomes one parallel function call.

## Step 2: Write one file-processing function

This function reads one input file and writes one output file.

```python
from pathlib import Path


def count_error_lines(input_file_path):
    input_path = Path(input_file_path)
    output_directory = Path("/workspace/shared/logs/processed")
    output_directory.mkdir(parents=True, exist_ok=True)

    output_path = output_directory / f"{input_path.stem}-summary.txt"
    error_count = 0

    for line in input_path.read_text().splitlines():
        if "ERROR" in line:
            error_count += 1

    output_path.write_text(f"error_count={error_count}\n")
    return str(output_path)
```

## Step 3: Run all files in parallel

Use `remote_parallel_map` with your function and file path list.

```python
from burla import remote_parallel_map

processed_file_paths = remote_parallel_map(count_error_lines, input_file_paths)
print(f"Created {len(processed_file_paths)} output files")
print(processed_file_paths[:3])
```

## Step 4: Combine the per-file outputs into one report

You can keep this final combine step simple.

```python
from pathlib import Path

total_error_count = 0

for processed_file_path in processed_file_paths:
    line = Path(processed_file_path).read_text().strip()
    error_count = int(line.split("=")[1])
    total_error_count += error_count

final_report_path = Path("/workspace/shared/logs/final/error-report.txt")
final_report_path.parent.mkdir(parents=True, exist_ok=True)
final_report_path.write_text(f"total_error_count={total_error_count}\n")

print(final_report_path)
```

## Step 5: Scale up safely

Before you run thousands of files, test with a small subset first.

```python
small_test_paths = input_file_paths[:20]
test_outputs = remote_parallel_map(count_error_lines, small_test_paths)
print(f"Test outputs: {len(test_outputs)}")
```

When the small test works, run the full list.

## What to do next

If you have one very large file instead of many small files, continue with [Process one giant file quickly.](process-one-giant-file-quickly.md)
