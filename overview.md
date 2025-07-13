# Overview

### Burla is a python package with only one function:

* Anything you print appears your local terminal.
* Exceptions thrown in your code are thrown on your local machine.
* Responses are pretty quick, you can call a million simple functions in a couple seconds.

With Burla, running code in the cloud feels the same as coding locally:

This code runs: `my_function(1)`, `my_function(2)`, `my_function(3)` in parallel, each in a separate container, and on a separate cpu, in the cloud.

```python
from burla import remote_parallel_map


def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```

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
