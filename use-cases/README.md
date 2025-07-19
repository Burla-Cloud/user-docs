---
layout:
  width: default
  title:
    visible: false
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: false
---

# Use Cases

## One Function, Endless Possibility

`remote_parallel_map` is the **simplest possible abstraction** to run code on lots of computers!

```python
from burla import remote_parallel_map

my_inputs = [1, 2, 3]

def my_function(my_input):
    print("I'm running on my own separate computer in the cloud!")
    return my_input
    
return_values = remote_parallel_map(my_function, my_inputs)
```

That means it can simplify and accelerate any solution that runs code on many machines, like:

<table data-view="cards"><thead><tr><th align="center"></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td align="center">Save Big on Batch AI Inference</td><td><a href="../.gitbook/assets/Screenshot 2025-07-13 at 5.42.56 PM.png">Screenshot 2025-07-13 at 5.42.56 PM.png</a></td><td><a href="./">.</a></td></tr><tr><td align="center">Orchestrate Data Pipelines</td><td><a href="../.gitbook/assets/Screenshot 2025-07-16 at 11.41.14 AM.png">Screenshot 2025-07-16 at 11.41.14 AM.png</a></td><td></td></tr><tr><td align="center">Scale Computational Bio</td><td><a href="../.gitbook/assets/Screenshot 2025-07-13 at 5.42.46 PM.png">Screenshot 2025-07-13 at 5.42.46 PM.png</a></td><td><a href="./">.</a></td></tr><tr><td align="center">Develop in Remote Environments</td><td><a href="../.gitbook/assets/Screenshot 2025-07-16 at 12.27.14 PM.png">Screenshot 2025-07-16 at 12.27.14 PM.png</a></td><td></td></tr><tr><td align="center">Prepare Terabytes of Data</td><td><a href="../.gitbook/assets/Screenshot 2025-07-13 at 5.40.15 PM.png">Screenshot 2025-07-13 at 5.40.15 PM.png</a></td><td><a href="./">.</a></td></tr><tr><td align="center">Simplify AI Agent Development</td><td><a href="../.gitbook/assets/Screenshot 2025-07-18 at 1.06.55 AM.png">Screenshot 2025-07-18 at 1.06.55 AM.png</a></td><td></td></tr></tbody></table>

&#x20;

#### Try it now with just two commands:

(**Currently Google Cloud only!**)

1. `pip install burla`
2. `burla install`

See our Getting Started guide for more info:

{% embed url="https://docs.burla.dev/getting-started#quickstart" %}

&#x20;&#x20;

&#x20;

***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
