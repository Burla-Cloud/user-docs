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

Burla is an open-source package with **one function**: \`remote\_parallel\_map\`. Here's an example:

<figure><img src=".gitbook/assets/CleanShot 2026-01-18 at 15.07.24.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/final_terminal_with_header_rounded.gif" alt=""><figcaption></figcaption></figure>

<p align="center">This realtime example runs <code>my_function</code> on 1,000 separate computers in one second!</p>

### Enable anyone to process terabytes of data in minutes, not days.

Burla is simple enough for anyone to learn, yet extremely scalable, and flexible:

* **Scalable:** See our [demo](examples/process-2.4tb-of-parquet-files-in-76s.md) where we process 2.4TB in 76s using 10,000 CPUs!
* **Flexible:** Runs any code, inside any Docker container, on any hardware like GPU's or TPU's.

Our self-hostable dashboard makes it easy to monitor long-running workloads, or manage data.

<figure><img src=".gitbook/assets/new_platform_demo.gif" alt=""><figcaption></figcaption></figure>

### **How it works:**

Burla only has one function: `remote_parallel_map`\
When called, it runs the given function, on every input in the given list, each on a separate computer.

```python
from burla import remote_parallel_map

my_inputs = [1, 2, 3]

def my_function(my_input):
    print("I'm running on my own separate computer in the cloud!")
    return my_input
    
return_values = remote_parallel_map(my_function, my_inputs)
```

Running code in the cloud with Burla feels the same as coding locally:

* Anything you print appears in your local terminal.
* Exceptions thrown in your code are thrown on your local machine.
* Responses are quick, you run a million function calls in a couple seconds!

### Features:

{% columns %}
{% column width="50%" %}
**📦 Automatic Package Sync**

Burla clusters automatically (and very quickly) install any missing python packages into all containers in the cluster.

**🐋 Custom Containers**

Easily run code in any Docker container.\
Public or private, just paste an image URI in the settings, then hit start!
{% endcolumn %}

{% column width="50%" %}
**📂 Network Filesystem**

Need to get big data into/out of the cluster? Burla automatically mounts a cloud storage bucket to `./shared` in every container.

**⚙️ Variable Hardware Per-Function**

The `func_cpu` and `func_ram` args make it possible to assign more hardware to some functions, and less to others, unlocking new ways to simplify pipelines and architecture.
{% endcolumn %}
{% endcolumns %}

### Build scalable data-pipelines using plain Python:

Fan code across thousands of machines, then combine results on one big machine.\
The network filesystem mounted at `./shared` makes it easy to pass big data between steps.

```python
from burla import remote_parallel_map

# Run `process_file` on many small machines
results = remote_parallel_map(process_file, files)

# Combine results on one big machine
result = remote_parallel_map(combine_results, [results], func_ram=256)
```

<p align="center">The above example demonstrates a basic map-reduce operation.</p>

### Demo:

{% embed url="https://www.youtube.com/watch?v=9d22y_kWjyE" %}

### Try it out today:

There are two ways to host Burla:

1. **In your cloud.**\
   Burla is open-source, and can be deployed with one command (currently Google-Cloud only).\
   [Click here](https://docs.burla.dev/get-started#quickstart-self-hosted) to get started with self-hosted Burla.
2. **In our cloud.**\
   First $1,000 in compute spend is free, try it now 👇

{% embed url="https://docs.burla.dev/signup" %}

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
