---
description: Why cluster-first distributed systems are the wrong abstraction for dynamic pipelines.
layout:
  width: default
  title:
    visible: true
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
  tags:
    visible: true
---

# Stop Designing the Cluster

Most distributed systems have the same bug: they make you design the cluster before you can describe the work.

Before you know what the pipeline will discover, you choose node types. Before you know which stages deserve GPUs, you size worker pools. Before you know whether a branch will matter, you decide which Docker image has to carry every dependency you might need and how much capacity to keep warm just in case.

We have normalized this so thoroughly that it feels inevitable.

It isn't. It is an abstraction leak.

## We are still writing two programs

Most distributed systems make you write two programs.

The first is the one you actually care about: the pipeline, the model, the transformation, the simulation, the scoring job, the business logic.

The second is the one no one talks about directly: the infrastructure plan. Which machines exist. Which workers get GPUs. Which images get preloaded. How aggressively the cluster scales. How much idle capacity to keep around. Which kinds of nested work are safe. Which branches are too awkward to express naturally because the cluster will not tolerate them.

This second program is scattered across cluster config, autoscaler settings, image strategy, scheduler hints, and a thousand quiet design decisions people stop noticing after a while.

That is the real tax.

And the problem is not just that this tax is annoying. The problem is that it changes how people write software.

When infrastructure is awkward, programmers stop writing the system that best matches the problem. They start writing the system the cluster is willing to run.

They flatten workflows that should branch. They avoid nested parallelism. They batch unlike stages together just to stay inside one environment. They keep heterogeneous capacity warm because changing course mid-run is too expensive. They ship giant images with everything baked in because switching environments later is painful.

The software becomes polite to the infrastructure instead of faithful to the problem.

That is a bad bargain.

## Dynamic pipelines expose the leak

This becomes obvious the moment the workload is genuinely dynamic.

Imagine a pipeline that starts by parsing millions of documents on cheap CPU workers. Most of them are easy. But 3% look suspicious, so only that subset goes through a heavier model on H100s. That model flags a smaller subset that now needs OCR in a different container with a different dependency stack. Some of those results fan out into per-page work. Others stop there. Later, everything fans back in for aggregation and post-processing on lightweight machines.

What exactly was the cluster for that program?

If you answer that question up front, you are guessing.

You are designing infrastructure for branches the program may never take. You are provisioning around worst-case possibilities instead of actual execution. You are pretending the shape of the computation is known before the computation has happened.

That is backwards.

The pipeline is the thing learning about the work. The infrastructure should be allowed to follow it.

Once the workload becomes truly dynamic, the cluster-first model starts to look brittle. Not because it is impossible to make it work, but because the developer is doing work the runtime should be doing.

A remote function should be allowed to discover more work and launch more work.

A later stage should be allowed to ask for different hardware.

A different branch should be allowed to run in a different environment.

None of that should feel exotic. That is what dynamic software does.

## The runtime should turn intent into infrastructure

The right abstraction is much simpler.

A distributed program should be able to say:

- run this function over these inputs
- give each invocation the CPU, RAM, and GPU it needs
- if the next stage needs a different container, use a different container
- if a remote step discovers more work, let it fan out again
- if the amount of parallelism changes, let the runtime adapt
- when the work is done, tear the infrastructure back down

Everything below that line should happen automatically.

The machines should appear because the computation requires them.

The container should be selected because the computation requires it.

The level of parallelism should change because the computation requires it.

And when the workload narrows, the infrastructure should narrow with it.

This is not the weak version of the idea, where there is an autoscaler somewhere in the background. It is the stronger version: infrastructure is no longer the thing the developer is expected to plan first.

That does not mean low-level control disappears. Experts will always want escape hatches. The lower layers still matter, the way memory allocation still matters and routing still matters and storage engines still matter.

But mature abstractions change where people start.

Most developers do not think about memory pages when they allocate an object. Most developers do not think about packets when they make an HTTP request. They operate at the level of intent, and the system handles the machinery underneath.

Distributed computing should move up the stack in the same way.

We do not need better cluster YAML nearly as much as we need fewer reasons for developers to think in cluster YAML at all.

## What this unlocks

Once you stop forcing the pipeline to fit a predesigned cluster, a much larger class of software becomes practical.

You can:

- change hardware class mid-program, from lots of cheap CPU workers to H100s and back again
- switch Docker images between stages because the environment belongs to the step, not to a fixed cluster identity
- let remote functions trigger more remote work, so nested fan-out becomes normal instead of dangerous
- pay for expensive infrastructure only when the data actually justifies it
- let the runtime react to the real shape of the workload instead of a guess made before execution

Bad abstractions do not just waste time. They shrink the set of ideas teams are willing to try.

When every dynamic branch implies more infrastructure choreography, people quietly stop building dynamic systems.

Better abstractions do the opposite. They expand the space of programs that are reasonable to write.

That is the real prize.

## The bet behind Burla

This is the bet behind Burla.

Burla is opinionated on purpose. It starts from the work, not the cluster.

The core primitive is [`remote_parallel_map`](API-Reference.md): give it a function, a set of inputs, and the requirements for that stage. CPU. RAM. GPU if needed. A container image if needed. Burla figures out the machines, brings up what it needs, runs the work, and gets out of the way.

A simple version looks like this:

```python
from burla import remote_parallel_map

chunks = remote_parallel_map(
    preprocess,
    raw_files,
    func_cpu=1,
    func_ram=2,
    grow=True,
)

embeddings = remote_parallel_map(
    embed,
    chunks,
    func_gpu="H100_80G",
    image="gcr.io/acme/pytorch-2.3-cu12",
    grow=True,
)

remote_parallel_map(
    write_results,
    embeddings,
    func_cpu=1,
    func_ram=4,
)
```

That code is doing something deeper than saving a few lines of configuration.

The first stage is saying: use cheap CPU workers.

The second stage is saying: now switch to H100s and a different container.

The third stage is saying: now go back to lightweight machines.

The developer never had to pre-design a permanent cluster that could satisfy all three stages simultaneously. The pipeline decided the infrastructure as it went.

And it gets better from there.

A Burla function can itself call `remote_parallel_map` again. So if a remote step discovers that one input should branch, recurse, or explode into ten thousand more tasks, it can do that directly. Nested fan-out is not treated like a scheduling hazard. It is just more computation.

That matters because a lot of valuable systems are not static. They branch. They learn. They escalate some paths and skip others. They change environments midstream. They discover that a small subset deserves expensive treatment. They discover more work inside the work.

The infrastructure should be able to follow that.

## The cluster should be an implementation detail

Much of cloud tooling is still stuck halfway up the abstraction ladder.

We replaced racks with instances. Then instances with containers. Then we added schedulers, managed clusters, and autoscalers. All of that was real progress.

But too much of the developer experience still assumes that you should be thinking about worker pools, warm capacity, node types, placement strategy, and image policy before you can cleanly express what the program wants to do.

That is better than bare metal.

It is not the end state.

The end state is not infrastructure, but more automated.

The end state is that, for a large class of workloads, infrastructure recedes from the foreground. The developer describes the work. The developer describes the requirements of each step. The runtime turns that into machines, images, GPUs, parallelism, and teardown.

Not because developers no longer care about performance or control.

Because the abstraction has finally moved high enough.

The cluster was never supposed to be the thing you were designing.

It was supposed to be the thing the runtime figured out after you described the work.
