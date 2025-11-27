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
---

# Welcome

### Run any Python function on 1,000 computers in 1 second.

Burla is the simplest way to scale Python. It only has one function: [`remote_parallel_map`](API-Reference.md#burla.remote_parallel_map)\
It's open-source, works with GPU's, any docker image, and scales up to 10,000 CPU's in a single call.

<figure><img src=".gitbook/assets/main_demo.gif" alt=""><figcaption></figcaption></figure>

### Enable <mark style="color:red;">anyone</mark> to process terabytes of data in <mark style="color:red;">seconds</mark>.

Burla is extremely scalable, extremely flexible, and extremely easy to learn.

* Over 10,000 CPUs in a single function call.\
  See our demo where we process 2.4TB of parquet files in just 48s, using <30 lines of code.
* One function, two required arguments.\
  This makes Burla simple enough for anyone to fully comprehend in minutes.
* Any hardware. Any docker image.\
  Burla can run any linux-compatible program, on any hardware, in parallel.

Our open-source web platform makes it easy to manage data, and monitor long-running pipelines.

<figure><img src=".gitbook/assets/new_platform_demo.gif" alt=""><figcaption></figcaption></figure>

### **How it works:**

Burla only has one function: `remote_parallel_map`  \
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
* Responses are pretty quick, you run a million function calls in a couple seconds!

### Features:

{% columns %}
{% column width="50%" %}
#### 📦 Automatic Package Sync

Burla clusters automatically (and very quickly) install any missing python packages into all containers in the cluster.

#### 🐋 Custom Containers

Easily run code in any linux-based Docker container. Public or private, just paste an image URI in the settings, then hit start!
{% endcolumn %}

{% column width="50%" %}
#### 📂 Network Filesystem

Need to get big data into/out of the cluster? Burla automatically mounts a cloud storage bucket to `./shared` in every container.

#### ⚙️ Variable Hardware Per-Function

The `func_cpu` and `func_ram` args make it possible to assign more hardware to some functions, and less to others, unlocking new ways to simplify pipelines and architecture.
{% endcolumn %}
{% endcolumns %}

### Create massive data-pipelines with plain Python:

Build pipelines that fan in/out over thousands of machines, then aggregate data in one big machine.\
The network filesystem mounted at \`./shared\` makes it easy to pass big data between steps.

```python
from burla import remote_parallel_map

# Run `process_file` on many smaller machines
results = remote_parallel_map(process_file, files)

# Combine results on one large machine
result = remote_parallel_map(combine_results, [results], func_ram=256)
```

### Quick Demo: Hyper-parameter tuning XGBoost with 1,000 CPUs

{% embed url="https://www.youtube.com/watch?v=9d22y_kWjyE" %}

### &#x20;Get started now:

{% embed url="https://docs.burla.dev/signup" %}







***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
