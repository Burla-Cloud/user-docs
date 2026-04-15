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

Burla is a high-performance parallel processing library for data-teams that iterate quickly.\
Run vector embeddings, inference, or preprocessing inside your cloud with an instant feedback.

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

## The cleanest way to build scalable data-pipelines.

Zero special syntax. Change containers, hardware, or cluster-size automatically mid-workload.

Burla scales up to 10,000 CPUs in a single function call, supports GPUs, and any Docker container.\
Pipelines built with Burla are simpler, more maintainable, faster, and more fun to develop!

```python
remote_parallel_map(process, [...], image="osgeo/gdal:latest")
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

This creates a pipeline like:

<figure><img src=".gitbook/assets/data-pipeline-4-high-quality (1).gif" alt=""><figcaption></figcaption></figure>

### Monitor progress in the dashboard:

Cancel bad runs, filter logs to watch individual inputs, or monitor output files in the UI.

<figure><img src=".gitbook/assets/area2-radius60-247-251-252.gif" alt=""><figcaption></figcaption></figure>

## How it works:

Develop like your laptop has 1,000 CPUs or GPUs. Remote development, local feel.

```python
return_values = remote_parallel_map(my_function, my_inputs)
```

When functions are run with `remote_parallel_map`:

* Anything they print appears locally (and inside the dashboard).
* Any exceptions are thrown locally.
* Any packages or local modules are (very quickly) cloned on remote machines.
* Code starts running in under one second! Even with millions of inputs or thousands of machines.

### Features:

{% columns %}
{% column width="50%" %}
**📦  Automatic Package Sync**

Burla automatically (and very quickly) clones your Python packages on every remote machine where code is executed.

**🐋  Custom Containers**

Easily run code in any Docker container.\
Public or private, just paste an image URI in the settings, then hit start!
{% endcolumn %}

{% column width="50%" %}
**📂  Network Filesystem**

Need to get big data into/out of the cluster? Burla automatically mounts a cloud storage bucket to a folder in every container.

**⚙️  Variable Hardware Per-Function**

The `func_cpu` and `func_ram` args make it possible to assign big hardware to some functions, and less to others.
{% endcolumn %}
{% endcolumns %}

***

## Pricing:

Free for Hobbyists. Compute prices the same as Google Cloud.

{% columns %}
{% column %}
### Self-Hosted: (in your cloud)
{% endcolumn %}

{% column %}
### Managed: (in Burla's cloud)
{% endcolumn %}
{% endcolumns %}

| ✔ $100/month per Enterprise User.  | ✔  $100/month per Enterprise User.                                            |
| ---------------------------------- | ----------------------------------------------------------------------------- |
| ✅ Free for all non-commercial use. | ✅ 100% Identical pricing to [Google Cloud](https://cloud.google.com/pricing). |
|                                    | ✅ $500 in free credits for qualified users.                                   |

### Self-Host Burla today:

Burla can be self-hosted with just [one command](API-Reference.md#burla-install). To learn more see our Getting-Started guide:

{% embed url="https://docs.burla.dev/get-started#quickstart-self-hosted-runs-in-your-cloud" %}

### Try our 1000-CPU Quickstart:

1. [Sign in](https://login.burla.dev/) using your Google or Microsoft account.
2. Run the quickstart in this Google Colab notebook:  ( Takes less than 2 minutes! )

{% embed url="https://colab.research.google.com/drive/1msf0EWJA2wdH4QG5wPX2BncSEr5uVufv?usp=sharing" %}







***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
