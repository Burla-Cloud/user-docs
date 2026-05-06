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
remote_parallel_map(process, [...], image="osgeo/gdal:latest")
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

Creates a pipeline like:

<figure><img src=".gitbook/assets/data-pipeline-4-high-quality (1).gif" alt=""><figcaption></figcaption></figure>

### Monitor progress in the dashboard:

Cancel bad runs, filter logs to watch individual inputs, or monitor output files in the UI.

<figure><img src=".gitbook/assets/area2-radius60-247-251-252.gif" alt=""><figcaption></figcaption></figure>

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

### Features:

{% columns %}
{% column width="50%" %}
**📦 Automatic Package Sync**

Burla automatically (and very quickly) clones your Python packages on every remote machine where code is executed.

**🐋 Custom Containers**

Easily run code in any Docker container.\
Public or private, just paste an image URI in the settings, then hit start!
{% endcolumn %}

{% column width="50%" %}
**📂 Network Filesystem**

Need to get big data into/out of the cluster? Burla automatically mounts a cloud storage bucket to a folder in every container.

**⚙️ Variable Hardware Per-Function**

The `func_cpu` and `func_ram` args make it possible to assign big hardware to some functions, and less to others.
{% endcolumn %}
{% endcolumns %}

***

## Pricing:

Free for Hobbyists. Compute prices the same as Google Cloud.

{% columns %}
{% column %}
#### Self-Hosted: (in your cloud)
{% endcolumn %}

{% column %}
#### Managed: (in Burla's cloud)
{% endcolumn %}
{% endcolumns %}

| ✔ $100/month per Enterprise User.  | ✔ $100/month per Enterprise User.                                             |
| ---------------------------------- | ----------------------------------------------------------------------------- |
| ✅ Free for all non-commercial use. | ✅ 100% Identical pricing to [Google Cloud](https://cloud.google.com/pricing). |
|                                    | ✅ $500 in free credits for qualified users.                                   |

### Run your first 1000-CPU job in 2 minutes:

No setup required. Sign in, open the Colab, and follow along. It's only 4 steps!

1. [Sign in](https://login.burla.dev/) using your Google or Microsoft account.
2. Run the quickstart in this Google Colab notebook:

{% embed url="https://colab.research.google.com/drive/1msf0EWJA2wdH4QG5wPX2BncSEr5uVufv?usp=sharing" %}





***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
