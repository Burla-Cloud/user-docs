---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: false
  pagination:
    visible: true
---

# Welcome

{% embed url="https://www.youtube.com/watch?v=1HQkTL-7_VY" %}

#### Sign Up Now:

{% @formspree/formspree %}

#### Burla is an open-source, batch-processing platform for Python developers.

* It can deploy a simple python function to 10,000VM's in about 2 seconds (see our [demo](https://www.youtube.com/watch?v=1HQkTL-7_VY)).
* Code can run in any custom docker container, on any machine type, for any length of time.
* It comes with a dashboard to monitor long running batch jobs, and view logs.
* Burla can be installed with [one command](installation.md).

#### Basic example:

Burla is a python package with only [one function](API-Reference.md#burla.remote_parallel_map). Here's an example:

```python
from burla import remote_parallel_map


def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```

This code runs: `my_function(1)`, `my_function(2)`, `my_function(3)` in parallel, each in a separate container, and on a separate cpu, in the cloud.

With Burla, running code in the cloud feels the same as coding locally. This means:

* Errors thrown in your code appear on your local machine.
* Anything you print appears your local terminal.
* Responses are pretty quick (you can call a million simple functions in a couple seconds).

For more info on `remote_parallel_map` see our [overview](overview.md#burla.remote_parallel_map) page, or the [API docs](API-Reference.md).









***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.

