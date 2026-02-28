---
description: An example for running one function call on a bigger machine with func_cpu and func_ram.
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

# Run code on one big cloud machine.

Sometimes you do not want thousands of small machines.
You want one bigger machine, with more CPUs and more RAM, to run one heavy step.

In this example we run a single function call on a bigger machine by passing `func_cpu` and `func_ram`.

## Before you start

Make sure you have already:

1. installed Burla: `pip install burla`
2. connected your machine: `burla login`
3. started your cluster in the Burla dashboard

## Step 1: Define a function you want to run on a bigger machine

This example reads a file and returns a small summary.

```python
from pathlib import Path

from burla import remote_parallel_map


def summarize_text_file(file_path):
    text = Path(file_path).read_text()
    return {"file_path": file_path, "character_count": len(text)}
```

## Step 2: Run it once, but with more CPU and RAM

`remote_parallel_map` always takes a list of inputs.
To run your function once, pass a list with one item.

`func_cpu` is the number of CPUs, and `func_ram` is RAM in **GB**.

```python
file_path = "/workspace/shared/big-text-file.txt"

results = remote_parallel_map(
    summarize_text_file,
    [file_path],
    func_cpu=8,
    func_ram=64,
)

print(results[0])
```

This runs `summarize_text_file(file_path)` on one remote machine that has up to 8 CPUs and 64GB of RAM available for that function call.
