---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Welcome

<div align="center">

<img src=".gitbook/assets/banner.png" alt="" width="1000">

</div>

Burla is a library for running python functions on (lots of) computers in the cloud.

### Quickstart:

1. To install, run: `pip install burla`
2. To create an account/login, run: `burla login`
3. Click the "start cluster" button at [cluster.burla.dev](https://cluster.burla.dev)
4. Once booted, try the following basic example:

```python
from burla import remote_parallel_map

my_inputs = list(range(100))

def my_function(my_input):
    print(f"processing input #{my_input}")
    return my_input * 2
    
results = remote_parallel_map(my_function, my_inputs)

print(f"return values: {list(results)}")
```

### What is Burla:

Burla is kind of like AWS Lambda, except it:

* deploys code in seconds
* is invoked like a normal local python function
* lets you use custom hardware you can change on the fly / per request
* lets you run code in custom docker/OCI containers
* has no limit on runtime (lambda has a 15min limit)
* is open-source, and designed to be self-hosted

To use Burla you must have a cluster running that the client knows about.\
Currently, out library is hardcoded to only call our free public cluster ([cluster.burla.dev](https://cluster.burla.dev)).

Currently, our public cluster is configured to run 16, 32 cpu vms.\
Burla clusters are multi-tenant/ can run many jobs from separate users.\
Nodes in a burla cluster are single-tenant/ your job will never be on the same machine as another job.

### Components / How it works:

Burla's major components are split across 4 separate GitHub repositories.

1. [Burla](https://github.com/burla-cloud/burla)\
   The python package (the client).
2. [main\_service](https://github.com/burla-cloud/main\_service)\
   Service representing a single cluster, manages nodes, routes requests to node\_services.
3. [node\_service](https://github.com/burla-cloud/node\_service)\
   Service running on each node, manages containers, routes requests to container\_services.
4. [container\_service](https://github.com/burla-cloud/container\_service)\
   Service running inside each container, executes user submitted functions.



