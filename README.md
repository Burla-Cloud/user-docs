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

# Welcome

### Scale Python across 1,000's of computers using one line of code.

Burla is an open-source Python package with only **one function**: `remote_parallel_map`\
Here's an example:

<figure><img src=".gitbook/assets/CleanShot 2026-01-18 at 15.07.24.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/final_terminal_with_header_rounded.gif" alt=""><figcaption></figcaption></figure>

<p align="center">This realtime example runs <code>my_function</code> on 1,000 separate computers in one second!</p>

### Enable anyone to process terabytes of data in minutes, not days.

Burla is simple enough for anyone to learn, yet extremely scalable, and flexible:

* **Scalable:** See our [demo](examples/process-2.4tb-of-parquet-files-in-76s.md) where we process 2.4TB in 76s using 10,000 CPUs!
* **Flexible:** Runs any code, inside any Docker container, on any hardware like GPU's or TPU's.

Easily monitor long-running workloads, or manage compute resources in the dashboard.

<figure><img src=".gitbook/assets/new_platform_demo.gif" alt=""><figcaption></figcaption></figure>

### How it works:

With Burla, **running code in the cloud feels the same as coding on your laptop:**

```python
return_values = remote_parallel_map(my_function, my_inputs)
```

When functions are run using `remote_parallel_map`:

* Anything they print appears in your local terminal, and inside Burla's dashboard.
* Any exceptions in your code are thrown locally.
* Your local Python environment is quickly cloned on all remote workers.
* Code begins running in under one second! Even with millions of inputs or thousands of machines.

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

The `func_cpu` and `func_ram` args make it possible to assign more hardware to some functions, and less to others.
{% endcolumn %}
{% endcolumns %}

### Convert any workload into a scalable data-pipeline:

Have a workload that takes forever to run?

By injecting many `remote_parallel_map` calls into your code, data-science teams have created programs that handle more data, and finish running in minutes instead of days.

The network filesystem at `./shared` makes it trivial to process your data stored in cloud storage.

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
