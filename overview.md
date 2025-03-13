# Overview

### `burla.remote_parallel_map`:

Run an arbitrary python function on many remote computers at the same time.\
[Click here for API-Documentation.](API-Reference.md)

#### Basic use:

`remote_parallel_map` requires two arguments:

```python
# Arg 1: Any python function.
def my_function(my_input):
    print(my_input)
    return my_input * 2

# Arg 2: List of inputs for `my_function`.
my_inputs = [1, 2, 3]
```

Then `remote_parallel_map` can be called like:

```python
from burla import remote_parallel_map

outputs = remote_parallel_map(my_function, my_inputs)

print(list(outputs))
```

When run, `remote_parallel_map` will call `my_function`, on every object in `my_inputs`, at the same time, each on a separate CPU in the cloud (the max #CPUs depends on your cluster settings).

In under 1 second, the three function calls are made, all at the same time:\
`my_function(1)` , `my_function(2)`, `my_function(3)`

Stdout produced on the remote machines is streamed back to the client (your machine).\
The return values of each function are also collected and sent back to the client.\
The following displays in the client's terminal:

```bash
1
2
3
[2, 4, 6]
```

In the above example, each function call would have been made inside a separate container, each with their own isolated filesystem.

#### Other arguments:

`remote_parallel_map` has a few optional arguments, see [API-Reference](API-Reference.md) for the full API-doc.

```python
remote_parallel_map(
  function_,
  inputs,
  func_cpu=1,
  func_ram=4,
  spinner=True,
  generator=False,
  max_parallelism=None,
  api_key=None,
)
```

The **`func_cpu`** and **`func_ram`** arguments can be used to assign more resources to each individual function call, up to 32 CPUs and 128G of RAM per function call.

**`max_parallelism`** can be used to limit the number of function calls running at the same time.\
By default, the cluster will execute as many parallel functions as possible given the resources it has. Our free public Burla cluster is configured to run 256 CPUs, allowing up to 256 function calls at the same time, however this can be increased if self-hosting.

**`spinner`** can be used to turn off the spinner, which also displays status messages from the cluster, like the state of the current job.

**`api_key`** exists so users can call `remote_parallel_map` inside deployment environments where `burla login` cannot be run. To get an API key send me an email: [jake@burla.dev](mailto:jake@burla.dev).

#### Limitations:

This list also serves as kind of a roadmap/to-do list.\
If one limitation in particular is currently blocking you, send me an email! (jake@burla.dev).\
Or even better, [schedule a quick meeting](http://cal.com/jakez/burla)!

* Maximum size of all inputs combined can be no larger than 84.8MB
* Maximum number of parallel function calls: 256
* Maximum number of inputs: 5,000,000
* Maximum number of CPUs per function call: 32
* Maximum RAM per function call: 128G
* no GPU support yet (If you ask me we can add this quickly!).

### Any Questions?

[Schedule a call with us!](http://cal.com/jakez/burla)\
We're always happy to talk face to face.



