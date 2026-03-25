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

## Scale Python across 1000 computers in 1 second.

Burla is an open-source cloud platform for Python developers. It only has one function:

```py
from burla import remote_parallel_map

my_inputs = list(range(1000))

def my_function(x):
    print(f"[#{x}] running on separate computer")

remote_parallel_map(my_function, my_inputs)
```

This runs `my_function` on 1000 vm's in the cloud, in < 1 second:

<figure><img src=".gitbook/assets/247-251-252.gif" alt=""><figcaption></figcaption></figure>

## The fastest way to create scalable data pipelines.

Burla scales up to 10,000 CPUs in a single function call, supports GPUs, and custom containers.\
Load data in parallel from cloud storage, then write results in parallel from thousands of VMs at once.

```python
remote_parallel_map(process, [...])
remote_parallel_map(aggregate, [...], func_cpu=64)
remote_parallel_map(predict, [...], func_gpu="A100")
```

This creates a pipeline like:

<figure><img src=".gitbook/assets/data-pipeline-4 (1).gif" alt=""><figcaption></figcaption></figure>

### Monitor progress in the dashboard:

Cancel bad runs, filter logs to watch individual inputs, or monitor output files in the UI.

<figure><img src=".gitbook/assets/area2-rounded-white-r60-exact-size.gif" alt=""><figcaption></figcaption></figure>

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

### Try Burla in less than 2 minutes:

1. [Sign in](https://login.burla.dev/) using your Google or Microsoft account.
2. Follow the 3-step quickstart on the homepage!

Burla is **open-source** and easy to self-host. [Click here](https://docs.burla.dev/get-started#quickstart-self-hosted) to deploy Burla in your cloud instead.





***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
