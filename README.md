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

## Scale Python to 1000+ VMs in 1 second.          Automatically achieve 2-5x higher utilization.               &#x20;

Burla is a self-hostable compute platform for processing big data.\
Scale analysis, inference, embeddings, and more with instant feedback, and better utilization.

Burla only has one function:

```py
from burla import remote_parallel_map

my_inputs = list(range(1000))

def my_function(x):
    print(f"[#{x}] running on separate computer")

remote_parallel_map(my_function, my_inputs)
```

This example runs `my_function` on 1,000 VMs in less than one second:

<figure><img src=".gitbook/assets/hell_cut_extended_no-zsh.gif" alt=""><figcaption></figcaption></figure>

## Pipelines are simpler with programmable infrastructure.

With Burla you define infrastructure inside your code. This makes it trivial to create pipelines that fan different functions in/out across thousands of machines with varying sizes, types, and environments.

Quickly develop pipelines that handle 100+ TB datasets, using plain Python anyone can understand.

This code:

```python
remote_parallel_map(process, [...], image="rocker/geospatial:latest")
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

Creates a pipeline like:

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

## Infra that manages itself is over twice as efficient.

Burla vertically scales hardware available to each function call live while the program is running.\
This frequently more than doubles compute efficiency, and eliminates out of memory errors.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```python
remote_parallel_map(..., func_ram="dynamic", func_cpu="dynamic")
```

Read [our blog post](blog/dynamic-hardware.md) to learn more about dynamic hardware.

## Write code like your laptop has 1000 CPUs or GPUs.

Running code in the cloud shouldn't feel any different from running code locally.

```python
return_values = remote_parallel_map(my_function, my_inputs)
```

When a Python function is run using `remote_parallel_map`, it runs in the cloud but:

* Anything it prints appears locally (and inside the dashboard).
* Any exceptions are thrown locally.
* Any packages or local modules are (very quickly) cloned on all remote machines.
* Code starts running in under one second! Even with millions of inputs, or thousands of machines.

Code runs on a pool of VM's that are automatically managed by Burla to maximize efficiency.\
You can manually add & remove machines from the pool, or let the platform react live to requests.&#x20;

## A full platform to scale up and monitor any workload.

Keep an eye on your analysis, pipeline, or background job from your phone.\
Burla has all the features you need to closely monitor logs, output files, and available compute.

<figure><img src=".gitbook/assets/area2-radius60-247-251-252.gif" alt=""><figcaption></figcaption></figure>

## Pricing:

Zero markup on compute.\
Burla is $250/month per enterprise user, and free for everyone else.

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## Run your first 1000-CPU job in 2 minutes:

Zero setup required. Sign in, open the Colab, and follow along. It's only 4 steps!

1. [Sign in](https://login.burla.dev/) using your Google or Microsoft account.
2. Run the quickstart in this Google Colab notebook:

{% embed url="https://colab.research.google.com/drive/1msf0EWJA2wdH4QG5wPX2BncSEr5uVufv?usp=sharing" %}

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
