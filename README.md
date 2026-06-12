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
  actions:
    visible: true
---

# Home

## Scale Python to 1,000 VMs in your cloud in 1 second.

Burla is an open-source compute platform for scaling Python applications.\
Run ML-pipelines, preprocessing and more with simpler code, less compute, and instant feedback.

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

&#x20;

## Scalable & efficient pipelines are not straightforward.

Slow deployments, VM reboots, or container rebuilds mean waiting 5-10 minutes with every change.\
Errors are vague, and configs are full of secret tradeoffs. 90% resource utilization is a pipe dream.

<figure><img src=".gitbook/assets/pipeline-problems.png" alt=""><figcaption></figcaption></figure>

&#x20;

## Burla can scale any workload with a single function.

Easily fan Python in/out across thousands of machines with varying sizes, types, and environments.\
Quickly develop pipelines that handle 100+ TB datasets, using simple code anyone can understand.

This code:

```python
remote_parallel_map(process, [...], image="rocker/geospatial:latest")
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

Creates a pipeline like:

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

&#x20;

&#x20;

## With Burla, the same jobs use 50% less compute.&#x20;

Compared to software like Ray, Dask, Airflow, or AWS Batch workloads running on Burla require less total compute, and automatically achieve close to 90% resource utilization for the duration of the job.

This is achieved with adaptive concurrency and horizontal autoscaling. Burla quickly reacts to changes in task resource utilization, and rearranges work during runtime to fill excess capacity.

<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

This system frequently more than doubles compute efficiency, and eliminates out of memory errors.\
[Read our blog](blog/dynamic-hardware.md) to learn how it works.

&#x20;

## How it works:

Running code in the cloud shouldn't feel any different from running code locally.

```python
return_values = remote_parallel_map(my_function, my_inputs)
```

When a Python function is run using `remote_parallel_map`, it runs in the cloud but:

* Anything it prints appears locally (and inside the dashboard).
* Any exceptions are thrown locally.
* Any packages or local modules are (very quickly) cloned on all remote machines.
* Code starts running in under one second! Even with millions of inputs, or thousands of machines.

Burla automatically manages it's own pool of VMs underneath to maximize speed and efficiency.\
You can manually add & remove machines from the pool, or let the platform react live to requests.

&#x20;

## Everything you need to manage Python at scale.

Monitor your analysis, pipeline, or background job from your phone.\
Burla has all the features you need to closely manage logs, output files, and available compute.

<figure><img src=".gitbook/assets/area2-radius60-247-251-252.gif" alt=""><figcaption></figcaption></figure>

&#x20;

## Pricing:

Zero markup on compute.\
Burla is free for any personal, academic, or research use.

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

## How we partner:

We only get paid if agreed-upon improvements in runtime and efficiency are actually achieved.\
Have peace of mind with on-call support from engineers experienced in ML at scale.

<figure><img src=".gitbook/assets/Updated spacing.png" alt=""><figcaption></figcaption></figure>

&#x20;

<h2 align="center"><a href="https://cal.com/jakez/burla?user=jakez">Book a discovery call</a></h2>

<p align="center">To learn how Burla can boost velocity, efficiency, and developer happiness.</p>

&#x20;

&#x20;

***

Questions?\
Email jake@burla.dev, we're always happy to talk.
