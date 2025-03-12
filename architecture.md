---
description: >-
  This section is intentionally kept brief because Burla is very new and
  constantly changing.
---

# Architecture

### Components

Burla has 4 major components: &#x20;

1.  **The Client**

    (the python package)
2.  **The "Main Service"**

    Service representing a single cluster, manages nodes, routes requests to node\_services.\
    Hosts the cluster dashboard.
3.  **The "Node Service"**

    Service running on each node, manages containers, routes requests to worker\_services.
4.  **The "Worker Service"**

    Service running inside each container, executes user submitted functions.

Each component has its own folder in the [Burla GitHub repo](https://github.com/Burla-Cloud/burla)'s top-level directory.\
When running the cluster kind of looks like this:

<figure><img src=".gitbook/assets/Screenshot 2025-01-03 at 12.24.04 PM.png" alt=""><figcaption></figcaption></figure>

Burla clusters are multi-tenant/ can run many jobs from separate users simultaneously.\
Nodes in a burla cluster are single-tenant/ separate jobs never share VM's.

### When the cluster starts:

A cluster-config document is created in Google Cloud Firestore that defines:

* The number & types of VMs to have running/waiting for requests.
* What containers to leave running, ready to serve requests, inside each VM.

### When `remote_parallel_map` is called:

The client concurrently:

* Begins uploading inputs into a message queue (currently uses google cloud firestore).
* Tells the [`main_service`](architecture.md#components) that a job is starting.

The [`main_service`](architecture.md#components) then tells the correct number of compatible [`node_services`](architecture.md#components), to start work on a particular job. Each node service then, in parallel, forwards this request to the correct number of compatible [`worker_services`](architecture.md#components). And, at the same time, kills any containers that are incompatible/ not needed for the current request.

As soon as the [`worker_services`](architecture.md#components) have been assigned a job, two things happen:

* Workers start popping/processing inputs from the input queue.
* The client receives a response from the [`main_service`](architecture.md#components) telling it that the workers have started.

Next, the client immediately begins listening to two other message queues:

* one streaming log messages (stdout/stderr) from workers back to the client.
* another streaming objects returned by the user's function back to the client.

The client listens to these streams until it has received one object for every input the user provided.\
As the client receives objects it yields them from `remote_parallel_map` which is a generator.

If the job takes a while the client sends health-check pings to the [`main_service`](architecture.md#components), which propagates them to [`node_services`](architecture.md#components), which propagate them to all [`worker_services`](architecture.md#components). These pings make sure that all workers are still working on the correct job and none of the services have crashed.

While the job is running every individual node is watching both the results/outputs stream, and it's own set of workers. If a user function error is detected (on any node), or all the workers on the current node finish their work, the node reboots itself, starting a new set of containers to keep warm as defined in the cluster spec.

### Any Questions?

[Schedule a call with us!](http://cal.com/jakez/burla)\
We're always happy to talk face to face.
