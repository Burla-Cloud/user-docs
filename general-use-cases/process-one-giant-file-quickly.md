---
description: A beginner-friendly guide for splitting one large file into chunks and processing chunks in parallel.
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

# Process one giant file quickly.

When one file is too large for fast single-machine processing, use this pattern:

1. split the file into chunk files
2. process chunk files in parallel
3. combine chunk results into one final result

This keeps the logic simple and gives you parallel speed.

## Before you start

Make sure you have already:

1. installed Burla: `pip install burla`
2. connected your machine: `burla login`
3. started your cluster in the Burla dashboard

## Step 1: Split one big file into chunk files

This example takes one giant text file and writes smaller chunk files with 50,000 lines each.

```python
from pathlib import Path


def create_chunk_files(
    input_file_path="/workspace/shared/giant/input.txt",
    output_directory_path="/workspace/shared/giant/chunks",
    lines_per_chunk=50000,
):
    output_directory = Path(output_directory_path)
    output_directory.mkdir(parents=True, exist_ok=True)
    lines = Path(input_file_path).read_text().splitlines()
    chunk_paths = []
    for index in range(0, len(lines), lines_per_chunk):
        chunk_path = output_directory / f"chunk-{index // lines_per_chunk}.txt"
        chunk_path.write_text("\n".join(lines[index:index + lines_per_chunk]) + "\n")
        chunk_paths.append(str(chunk_path))
    return chunk_paths


chunk_file_paths = create_chunk_files()
```

## Step 2: Process chunks in parallel

Now run one function call per chunk file.

```python
from pathlib import Path
from burla import remote_parallel_map


def count_lines_in_chunk(chunk_file_path):
    line_count = len(Path(chunk_file_path).read_text().splitlines())
    result_path = Path("/workspace/shared/giant/chunk-results") / f"{Path(chunk_file_path).stem}.txt"
    result_path.parent.mkdir(parents=True, exist_ok=True)
    result_path.write_text(f"{line_count}\n")
    return str(result_path)


chunk_result_paths = remote_parallel_map(count_lines_in_chunk, chunk_file_paths)
```

## Step 3: Combine chunk results into one final result

After all chunks finish, combine the per-chunk outputs.

```python
from pathlib import Path

total_line_count = sum(int(Path(path).read_text().strip()) for path in chunk_result_paths)

final_result_path = Path("/workspace/shared/giant/final/line-count.txt")
final_result_path.parent.mkdir(parents=True, exist_ok=True)
final_result_path.write_text(f"total_line_count={total_line_count}\n")

print(final_result_path)
```

## Step 4: Start small, then scale

Test the full workflow with a small file first.

When that works, run the same workflow on your giant file.

## What to do next

If your data is already in a database, continue with [Process data in your database quickly.](process-data-in-your-database-quickly.md)
