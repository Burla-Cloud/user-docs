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

## Scale Python across 1000 computers in 1 second.            Using one line of code.

Burla is a package with only **one function**.  Here's how it works:

```py
from burla import remote_parallel_map

my_inputs = list(range(1000))

def my_function(x):
    print(f"I'm running on my own separate computer in the cloud! #{x}")

remote_parallel_map(my_function, my_inputs)
```

**This runs `my_function` on 1000 vm's in the cloud, in 1 second:**

<figure><img src=".gitbook/assets/final_terminal_with_header_rounded.gif" alt=""><figcaption></figcaption></figure>

<h4 align="center">☝️ <a href="https://colab.research.google.com/drive/1msf0EWJA2wdH4QG5wPX2BncSEr5uVufv?usp=sharing">Try this example now in Google Colab</a> 🔗</h4>

## Easily build scalable Data-Pipelines.

Define any hardware at runtime, Burla scales up to 10,000 CPUs in a single function call.\
Load data in parallel from the cloud storage bucket mounted to every worker: `/workspace/shared`

```python
remote_parallel_map(process, ...)
remote_parallel_map(aggregate, ..., func_cpu=64)
remote_parallel_map(predict, ..., func_gpu="A100")
```

This creates a pipeline like:

<figure><img src=".gitbook/assets/output-onlinegiftools.gif" alt=""><figcaption></figcaption></figure>

### Monitor progress in the dashboard:

Cancel bad runs, filter logs to watch individual inputs, or watch output files appear in the storage UI.

<figure><img src=".gitbook/assets/new_platform_demo.gif" alt=""><figcaption></figcaption></figure>

## How it works:

With Burla, **running code in the cloud feels the same as coding on your laptop:**

```python
return_values = remote_parallel_map(my_function, my_inputs)
```

When functions are run with `remote_parallel_map`:

* Anything they print appears locally (and inside Burla's dashboard).
* Any exceptions are thrown locally.
* Any packages or local modules they use are (very quickly) cloned on remote machines.
* Code starts running in under one second! Even with millions of inputs or thousands of machines.

### Features:

{% columns %}
{% column width="50%" %}
**📦 Automatic Package Sync**

Burla automatically (and very quickly) clones your Python packages in every remote machine where your code runs.

**🐋 Custom Containers**

Easily run code in any Docker container.\
Public or private, just paste an image URI in the settings, then hit start!
{% endcolumn %}

{% column width="50%" %}
**📂 Network Filesystem**

Need to get big data into/out of the cluster? Burla automatically mounts a cloud storage bucket to `./shared` in every container.

**⚙️ Variable Hardware Per-Function**

The `func_cpu` and `func_ram` args make it possible to assign big hardware to some functions, and less to others.
{% endcolumn %}
{% endcolumns %}

### It only takes 2 minutes to try!

1. [Sign in](https://login.burla.dev/) using your Google or Microsoft account.
2. Follow our 3-step quickstart on the homepage!

Burla is **open-source** and easy to self-host. [Click here](https://docs.burla.dev/get-started#quickstart-self-hosted) to deploy Burla in your Cloud instead.

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
