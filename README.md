---
layout:
  width: default
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: false
  pagination:
    visible: false
  metadata:
    visible: false
  tags:
    visible: true
---

# Home

## Scale Python across 1000 CPUs or GPUs in 1 second.

Burla is a Python package for abstracting hardware, it makes cloud computing fast and simple.\
Scale vector embeddings, inference, preprocessing, build dynamic AI/ML pipelines, and more.

Burla only has one function:

```py
from burla import remote_parallel_map

my_inputs = list(range(1000))

def my_function(x):
    print(f"[#{x}] running on separate computer")

remote_parallel_map(my_function, my_inputs)
```

This runs `my_function` on 1000 vms in less than one second:

<figure><img src=".gitbook/assets/hell_cut_extended_no-zsh.gif" alt=""><figcaption></figcaption></figure>

## A better way to build scalable AI/ML data-pipelines.

Burla can change containers, hardware, or fan out to thousands of machines mid-workload.\
This makes it possible to create dynamic pipelines that decide hardware and scale at runtime.

Burla can scale up to 10,000 CPUs in a single function call, thousands of GPUs, or any container.\
Pipelines built with Burla are simpler, more maintainable, faster, and much easier to develop!

This code:

```python
remote_parallel_map(process, [...], image="rocker/geospatial:latest")
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

Creates a pipeline like:

<figure><img src=".gitbook/assets/data-pipeline-4-high-quality (1).gif" alt=""><figcaption></figcaption></figure>

## Double resource utilization with dynamic hardware.

Ever had a pipeline crash after running for 6 hours? or sit at 10% CPU for most of it's run?\
Resource needs change during your workload. With Burla, available hardware can change with it.

```python
remote_parallel_map(..., func_cpu="dynamic")
remote_parallel_map(..., func_ram="dynamic")
remote_parallel_map(..., grow=True)
```

By automatically adjusting CPU/RAM available to each task while running, Burla can massively improve utilization, and eliminate out of memory Errors or silent RAM-swap slowdowns.

Dynamic Hardware often more than doubles resource efficiency. Read [our blog](dynamic-hardware.md) to learn how it works.

## How it works:

With Burla, running code in the cloud feels the same as running code locally.

```python
return_values = remote_parallel_map(my_function, my_inputs)
```

When a Python function is run using `remote_parallel_map`:

* Anything it prints appears locally (and inside the dashboard).
* Any exceptions are thrown locally.
* Any packages or local modules are (very quickly) cloned on all remote machines.
* Code starts running in under one second! Even with millions of inputs or thousands of machines.

### Monitor progress in the dashboard:

Cancel bad runs, filter logs to watch individual inputs, or monitor output files live in the Filesystem UI.

<figure><img src=".gitbook/assets/area2-radius60-247-251-252.gif" alt=""><figcaption></figcaption></figure>

## Pricing:

Prices not based on consumption. Enterprise seats start at $100/month.\
Burla is free for Hobbyists and Researchers.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Run your first 1000-CPU job in 2 minutes:

Zero setup required. Sign in, open the Colab, and follow along. It's only 4 steps!

1. [Sign in](https://login.burla.dev/) using your Google or Microsoft account.
2. Run the quickstart in this Google Colab notebook:

{% embed url="https://colab.research.google.com/drive/1msf0EWJA2wdH4QG5wPX2BncSEr5uVufv?usp=sharing" %}

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
