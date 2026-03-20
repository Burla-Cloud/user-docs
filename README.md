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

### Enable anyone to process terabytes of data in minutes, not days.

Burla is simple enough for anyone to learn, yet extremely scalable, and flexible.

* **Scalable:** See our [demo](examples/process-2.4tb-of-parquet-files-in-76s.md) where we process 2.4TB in 76s using 10,000 CPUs!
* **Flexible:** Runs any code, inside any Docker container, on any hardware like GPU's or TPU's.

Easily monitor long-running workloads, or manage compute resources in the dashboard.

<figure><img src=".gitbook/assets/new_platform_demo.gif" alt=""><figcaption></figcaption></figure>

### How it works:

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

### Convert any workload into a scalable data-pipeline:

Have a workload that takes forever to run?

By injecting many `remote_parallel_map` calls into their code, Data-Scientists, ML-Engineers, and Analysts have created programs that handle terabytes of data, and finish running in minutes.

The network filesystem at `./shared` makes it trivial to process your data stored in a cloud storage.

```python
from burla import remote_parallel_map

# Run `process_file` on many small machines
results = remote_parallel_map(process_file, files)

# Combine results on one big machine
result = remote_parallel_map(combine_results, [results], func_cpu=64)
```

<p align="center">The above example demonstrates a basic map-reduce operation.</p>

### Burla only takes 2 minutes to try!

<a href="https://login.burla.dev/" class="button primary">Try Burla for free</a>

1. ☝️ Sign in using your Google or Microsoft account.
2. Click the **`⏻ Start`** button to boot some computers.
3. Scale Python over 1,000 CPU's in [this Google Colab notebook](https://colab.research.google.com/drive/1bR8Gpa85gqJi7_9uKdcJDX9_WG0tuVmG?usp=sharing)!

Quick reminder: Burla is open-source and easy to self-host. [Click here](https://docs.burla.dev/get-started#quickstart-self-hosted) to deploy Burla in your Cloud.

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
