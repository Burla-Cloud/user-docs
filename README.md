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

Burla is the world's simplest cluster compute software.\
It's open-source, works with any hardware, and any docker container.

<figure><img src=".gitbook/assets/main_demo.gif" alt=""><figcaption></figcaption></figure>

### A fully fledged data-platform any team can learn in minutes:

Schedule jobs, create pipelines, scale your machine learning system, or your research.\
Burla is simple enough to learn in minutes, and can be installed with a single command.

<figure><img src=".gitbook/assets/FINAL-lowfr.gif" alt=""><figcaption></figcaption></figure>

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

### Attach big hardware to functions that need it:

Zero config files, just simple arguments like `func_cpu` & `func_ram`.

```python
from xgboost import XGBClassifier

def train_model(hyper_parameters):
    model = XGBClassifier(n_jobs=64, **hyper_parameters)
    model.fit(training_inputs, training_targets)
    
remote_parallel_map(train_model, parameter_grid, func_cpu=64, func_ram=256)
```

### Simple, flexible pipelines:

Nest `remote_parallel_map` calls to build simple, massively parallel pipelines.\
Use `background=True` to fire and forget code, then monitor progress from the dashboard.

```python
from burla import remote_parallel_map

def process_record(record):
    # Pretend this does some math per-record!
    return result

def process_file(file):
    results = remote_parallel_map(process_record, split_into_records(file))
    upload_results(results)

def process_files(files):
    remote_parallel_map(process_file, files, func_ram=16)
    

remote_parallel_map(process_files, [files], background=True)
```

### Run code in any docker image, on any hardware:

Public or private, just paste a link to your image and hit start.\
Any packages you're using that are not in the container are quickly synchronized at runtime.

<figure><img src=".gitbook/assets/settings_demo.gif" alt=""><figcaption></figcaption></figure>

### Get started now:

Enter your email below and we'll reply same day with a free managed instance!\
Compute is on us, if you like it we'll help you get Burla installed in your private cloud.

{% @formspree/formspree fullWidth="false" %}

&#x20;

&#x20;

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
