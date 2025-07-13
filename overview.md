# Overview

### Burla is the simplest way to orchestrate parallel python.

* Deploy a simple function to 1000 computers in 1 second (see our [demo](https://www.youtube.com/watch?v=1HQkTL-7_VY)).
* Code runs in any docker container, on any machine type, for any length of time.
* It comes with a dashboard to monitor long running jobs, and view logs.
* Burla can be installed with [one command](installation.md).

### How it works:

Burla is a python package with only one function:

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on my own separate computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```

With Burla, running code in the cloud feels the same as coding locally:

* Anything you print appears your local terminal.
* Exceptions thrown in your code are thrown on your local machine.
* Responses are pretty quick, you can call a million simple functions in a couple seconds.

Additional arguments **`func_cpu`** and **`func_ram`**  can be used to assign more resources to each individual function call, and the **`background`**  arg can be used to launch a functions as long running background jobs. See our [API-Reference](API-Reference.md) for a full list of all additional arguments.

### Use cases:



***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk!
