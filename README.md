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

Burla manages infrastructure inside your cloud, doubles efficiency, and let's you focus on code.\
Scale vector embeddings, inference, preprocessing, build dynamic AI/ML pipelines, and more.

Burla only has one function:

```py
from burla import remote_parallel_map

my_inputs = list(range(1000))

def my_function(x):
    print(f"[#{x}] running on separate computer")

remote_parallel_map(my_function, my_inputs)
```

This example runs `my_function` on 1000 VMs in less than one second:

<figure><img src=".gitbook/assets/hell_cut_extended_no-zsh.gif" alt=""><figcaption></figcaption></figure>

## Stop wasting time. Infrastructure can manage itself.

Define the hardware or container you need next to the code that needs it.

Easily change hardware, docker containers, or fan out to thousands of machines mid-workload.\
Burla can scale up to 10,000 CPUs in a single function call, thousands of GPUs, or any container.

This code:

```python
remote_parallel_map(process, [...], image="rocker/geospatial:latest")
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

Creates a pipeline like:

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## Resource needs change.                                           Hardware should change with it!

Burla can adapt hardware assigned to each function call live while the program is running.\
This frequently more than doubles compute efficiency, and eliminates memory errors.

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

```python
remote_parallel_map(..., func_ram="dynamic", func_cpu="dynamic")
```

Read [our blog post](blog/dynamic-hardware.md) to learn more about dynamic hardware.

## Remote development, local feel.

Running code in the cloud shouldn't feel any different than running code locally.

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
