# Overview

#### Burla is the simplest way to orchestrate parallel python

* Deploy a simple function to 1000 computers in 1 second (see our [demo](https://www.youtube.com/watch?v=1HQkTL-7_VY)).
* Code runs in any docker container, on any machine type, for any length of time.
* It comes with a dashboard to monitor long running jobs, and view logs.
* Burla can be installed with [one command](installation.md).

#### How it works:

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

This code runs: `my_function(1)`, `my_function(2)`, `my_function(3)` in parallel, each in a separate container, and on a separate cpu, in the cloud.

#### Other arguments:

`remote_parallel_map` has a few optional arguments, see [API-Reference](API-Reference.md) for the full API-doc.

```python
remote_parallel_map(
  function_,
  inputs,
  func_cpu=1,
  func_ram=4,
  background=False,
  spinner=True,
  generator=False,
  max_parallelism=None,
)
```

The **`func_cpu`** and **`func_ram`** arguments can be used to assign more resources to each individual function call, up to max amount your machine type supports.

**`background`** makes `remote_parallel_map` exit after having uploaded the function and inputs. After which the job will continue to run independantly on the cluster in the background.

**`spinner`** can be used to turn off the spinner, which also displays status messages from the cluster, like the state of the current job.

**`generator`** makes `remote_parallel_map` return a python `generator` object instead of a `list` containing return values. This generator yields return values as they are produced instead of all at once.

**`max_parallelism`** can be used to limit the number of function calls running at the same time.\
By default, the cluster will execute as many parallel functions as possible given available resources.\
This can be useful to avoid overloading external systems like databases or other web services.





***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk!
