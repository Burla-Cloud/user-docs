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

<figure><img src=".gitbook/assets/random logos and graphics (6).png" alt=""><figcaption></figcaption></figure>

### Burla is a python package for running functions on remote computers.

With Burla, anyone can scale arbitrary python over thousands of virtual machines in the cloud.\
It's open-source, requires little setup, and is simple enough for even complete beginners.

### Basic example:

Burla is a python package with only one function: `remote_parallel_map`.

Given any python function, and a list of inputs, `remote_parallel_map` calls the given function, on every argument in the list, at the same time, each on a separate virtual machine in the cloud.

Here's an example:

```python
from burla import remote_parallel_map

my_arguments = [1, 2, 3, 4]

def my_function(my_argument: int):
    print(f"Running on remote computer #{my_argument} in the cloud!")
    return my_argument * 2
    
results = remote_parallel_map(my_function, my_arguments)

print(f"return values: {list(results)}")
```

In the above example, each call to `my_function` runs on a separate virtual machine, in parallel.\
With Burla, **running code on remote computers feels the same as running locally**. This means:

* Any errors your function throws will appear on your local machine same as they normally would.&#x20;
* Anything you print appears in your local stdout, just like it normally does.
* Responses are pretty quick (you can run a million simple functions in a couple seconds).

To run the example above simply run:

* Run the following code:
* Start our free demo cluster: [cluster.burla.dev](https://cluster.burla.dev)
* Run `burla login`

[Click here to learn more about remote\_parallel\_map](overview.md#burla.remote_parallel_map).











***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.

