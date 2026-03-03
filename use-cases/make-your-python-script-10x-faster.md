---
description: A guide for turning slow Python loops into fast parallel cloud jobs.
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

# Make your Python script 10x faster.

If your script does the same work one item at a time, it is often much faster to run many items at once in the cloud.

This guide shows a simple way to convert a normal Python loop into a parallel Burla job.

## Before you start

Make sure you have already:

1. installed Burla: `pip install burla`
2. connected your machine: `burla login`
3. started your cluster in the Burla dashboard

## Step 1: Find the part of your script that repeats

A good first target is work that looks like this:

```python
from time import sleep

items = [1, 2, 3]
results = []

for item in items:
    sleep(1)
    results.append(item * item)

print(results)
```

This loop is slow because it waits for each item to finish before starting the next one.

## Step 2: Move repeated work into one function

Put the per-item work into a function that takes one input and returns one output.

```python
from time import sleep


def square_number(number):
    sleep(1)
    return number * number
```

## Step 3: Run the function in parallel with Burla

Pass your function and a list of inputs to `remote_parallel_map`.

```python
from burla import remote_parallel_map
from time import sleep


def square_number(number):
    sleep(1)
    return number * number


numbers = [1, 2, 3]
results = remote_parallel_map(square_number, numbers)
print(results)
```

Burla runs one function call per input on separate cloud machines.

## Step 4: Check that your function is safe to parallelize

Your function should:

- use only its input to do work
- return its own result
- avoid shared local state between calls

This is safe:

```python
def process_record(record):
    return {"id": record["id"], "length": len(record["text"])}
```

This is not safe:

```python
total = 0


def process_record(record):
    global total
    total += len(record["text"])
    return total
```

## Step 5: Measure your speedup

Use a small timer so you can compare local and Burla runs.

```python
from time import perf_counter, sleep
from burla import remote_parallel_map


def square_number(number):
    sleep(1)
    return number * number


numbers = [1, 2, 3]

start_time = perf_counter()
local_results = [square_number(number) for number in numbers]
local_elapsed_seconds = perf_counter() - start_time

start_time = perf_counter()
remote_results = remote_parallel_map(square_number, numbers)
remote_elapsed_seconds = perf_counter() - start_time

print(local_elapsed_seconds)
print(remote_elapsed_seconds)
print(local_results == remote_results)
```

For independent work items, Burla is often much faster than a local loop.

## What to do next

Once this pattern works, apply the same function-plus-input-list shape to your real script.

If your script works with many files, continue with [Process thousands of files quickly.](process-thousands-of-files-quickly.md)
