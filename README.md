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

### Run any Python function on 1000 computers in 1 second.

Burla is the simplest way to scale python, it's has one function: `remote_parallel_map`\
It's open-source, works with GPU's, custom docker containers, and up to 10,000 CPU's at once.

<figure><img src=".gitbook/assets/main_demo.gif" alt=""><figcaption></figcaption></figure>

### A data-platform any team can learn in minutes:

Scale machine learning systems, or other research efforts without weeks of onboarding or setup.\
Burla's open-source web platform makes it simple to monitor long running pipelines or training runs.

<figure><img src=".gitbook/assets/new_platform_demo.gif" alt=""><figcaption></figcaption></figure>

### **How it works:**

Burla only has one function:

```python
from burla import remote_parallel_map

my_inputs = [1, 2, 3]

def my_function(my_input):
    print("I'm running on my own separate computer in the cloud!")
    return my_input
    
return_values = remote_parallel_map(my_function, my_inputs)
```

With Burla, running code in the cloud feels the same as coding locally:

* Anything you print appears in your local terminal.
* Exceptions thrown in your code are thrown on your local machine.
* Your local python packages are automatically synchronized with the cluster.
* Responses are pretty quick, you can call a million simple functions in a couple seconds!

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

Need to get big data into/out of the cluster? Burla automatically mounts a cloud storage bucket to your working directory.

#### ⚙️ Variable Hardware Per-Function

The `func_cpu` and `func_ram` args make it possible to assign more hardware to some functions, and less to others, unlocking new ways to simplify pipelines and architecture.
{% endcolumn %}
{% endcolumns %}

### Easily create pipelines without special syntax.

Nest `remote_parallel_map` calls fan code in/out over thousands of machines.\
Example: Process every record of many files in parallel, then combine results on one big machine.

```python
from burla import remote_parallel_map

def process_record(record):
    # Pretend this does some math per-record!
    return result

def process_file(filepath):
    # load records from disk (network storage)
    results = remote_parallel_map(process_record, records)
    # save results back to disk (network storage)

def combine_results(result_filepaths):
    # load results from disk (network storage), then combine!
    
result_filepaths = remote_parallel_map(process_file, filepaths)
remote_parallel_map(combine_results, [result_filepaths], func_ram=256)
```

### Watch our Demo:

{% embed url="https://www.youtube.com/watch?v=9d22y_kWjyE" %}

### &#x20;Get started now:

{% embed url="https://docs.burla.dev/signup" %}







***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
