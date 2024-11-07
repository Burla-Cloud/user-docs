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

When run, `remote_parallel_map` will call `my_function`, on every object in `my_inputs`, at the same time, each on a separate CPU in the cloud.

In under 1 second, the three function calls are made simultaneously:\
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
  max_parallelism=None,
  api_key=None,
)
```

The **`func_cpu`** and **`func_ram`** arguments can be used to assign more resources to each individual function call, up to 32 CPUs and 128G of RAM per function call.

**`max_parallelism`** can be used to limit the number of function calls running at the same time.\
By default, the cluster will execute as many parallel functions as possible given the resources it has. Our free public Burla cluster is configured to run 256 CPUs, allowing up to 256 function calls at the same time, however this can be increased if self-hosting.

**`spinner`** can be used to turn off the spinner, which also displays status messages from the cluster, like the state of the current job.

**`api_key`** exists so users can call `remote_parallel_map` inside deployment environments where `burla login` cannot be run. To get an API key send us an email: [jake@burla.dev](mailto:jake@burla.dev).

#### Limitations:

We're actively working to improve / reduce / eliminate the limitations listed below.\
The order in which we do this is determined by you!\
If one limitation in particular is blocking you, tell us! on our [discord](https://discord.gg/TsbCUwBUdy), in a [meeting](http://cal.com/jakez/burla), or over email (jake@burla.dev).

* Maximum size of all inputs combined can be no larger than 84.8MB
* Maximum number of parallel function calls: 256
* Maximum number of inputs: \~5,000,000
* Maximum number of CPUs per function call: 32
* Maximum RAM per function call: 128G
* Average e2e runtime for a function with small inputs: \~1.5s
* uneven/inflexible distribution of inputs between nodes can sometimes cause long runtimes.
* no GPU support (yet).
* unable to specify OCI/Docker containers on a per-request basis.

### Components

Burla's major components are split across 4 separate directories in the [Burla GitHub](https://github.com/burla-cloud/burla) repository. &#x20;

1.  [Burla](https://github.com/Burla-Cloud/burla/tree/main/burla)&#x20;

    The python package (the client).
2.  [main\_service ](https://github.com/Burla-Cloud/burla/tree/main/main\_service)

    Service representing a single cluster, manages nodes, routes requests to node\_services.
3.  [node\_service ](https://github.com/Burla-Cloud/burla/tree/main/node\_service)

    Service running on each node, manages containers, routes requests to container\_services.
4.  [container\_service ](https://github.com/Burla-Cloud/burla/tree/main/container\_service)

    Service running inside each container, executes user submitted functions.

### How does it work?

This section is intentionally kept brief because Burla is still early and constantly changing.

#### When the cluster starts:

A cluster spec exists somewhere (changing constantly) that defines:

* The number & types of VMs to have running/waiting for requests.
* The OCI containers & amount to have ready to serve requests inside each VM.

#### When `remote_parallel_map` is called:

The client concurrently:

* Begins uploading inputs into a distributed message queue.
* Tells the [`main_service`](overview.md#components) that a job is starting.

The [`main_service`](overview.md#components) then tells the correct number of compatible [`node_services`](overview.md#components), to start work on a particular job. Each node service then, in parallel, forwards this request to the correct number of compatible [`container_services`](overview.md#components) (workers). And, at the same time, kills any containers that are incompatible/ not needed for the current request.

As soon as the [`container_services`](overview.md#components) are aware of what job they're working on, two things happen:

* Workers start popping/processing inputs from the input queue.
* The client receives a 200 response from the [`main_service`](overview.md#components) telling it that the workers have started.

Once this happens the client immediately begins listening to two other distributed message queues:

* one streaming log messages (stdout/stderr) from workers back to the client.
* another streaming objects returned by the UDF (user defined function) back to the client.

The client listens to these streams until it has one return value for every input the user provided.\
As the client receives return values it yields them to the user through a generator.

If the job takes a while the client sends health-check pings to the [`main_service`](overview.md#components), which propagates them to [`node_services`](overview.md#components), which propagate them to all [`container_services`](overview.md#components). These pings make sure that all workers are still working on the correct job and none of the services have stopped responding.

While the job is running every individual node is watching the results stream, constantly checking if the number of results is the same as the number of inputs. Once this is true, the node reboots itself, starting all the containers it's supposed to be keeping warm as defined in the cluster spec.

### Any Questions?

[Schedule a call with us!](http://cal.com/jakez/burla)\
We're always happy to talk face to face.



