---
description: In fact, it's much better if you don't.
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

# You should never need to estimate how much CPU or RAM you need.

This might sound crazy but it's true. The reason why has nothing to do with our ability to guess.

The real reason is that modern cloud schedulers with access to live resource use information can vertically scale available CPU and RAM per workload in realtime, massively boosting efficiency, and eliminating memory errors.

This violates one of the most basic assumptions of modern cloud computing: before you run a workload, you need to describe the hardware it should run on. Not only is this unnecessary, it forces you into a static resource box, where more useful work frequently _could_ happen but doesn't. How often does your workload use 100% CPU or RAM? [Estimates](https://www.datacenterdynamics.com/en/news/only-13-of-provisioned-cpus-and-20-of-memory-utilized-in-cloud-computing-report/) put average utilization below 25%.

Let's clarify with a simple example. A data science team wants to parse 10,000 PDFs. Some are short. Some are enormous. Some are malformed. Some require much more memory, and many require very little. The job is clear: parse the PDFs. But the infrastructure interface asks a different question: how much CPU and RAM should each worker get? How many machines should I use? Should I process all the small ones separately? or run a separate cleaning step to avoid wasting resources?

We shouldn't need to think about these things. Not because it's annoying (which it is), but because even if you guess perfectly, the large PDF's will still sit at 10% CPU for an hour before quickly spiking to 100%, then back to 10%, and that extra space could have been put to use processing small PDFs.

The traditional interface is:

> Run this code on this hardware.

An ideal interface should be closer to:

> Run this code.

The system should figure out the hardware while the job is running.

## Problems with guessing

When you choose fixed resources upfront, there are only three possible outcomes.

You can set CPU and RAM too low. Then the job fails with out-of-memory errors, or it becomes mysteriously slow because memory spills to disk. The second case is often worse than the first. A crash is visible. Swap is quiet. A pipeline that should take minutes can become a pipeline that takes hours, and nothing in the user's code necessarily looks wrong.

You can set CPU and RAM too high. Then the job runs, but the cluster is full of reserved capacity that nobody is using. You are paying for machines that are sitting partly empty.

Or you can make a guess that is right on average. This still loses, because the average task is not the task currently running.

In the PDF example, a fixed worker size has to serve tiny PDFs and giant PDFs. If you size every worker for the giant PDFs, the tiny PDFs waste memory. If you size every worker for the tiny PDFs, the giant PDFs fail or crawl. If you pick a compromise, both problems remain: some work is overprovisioned, and some work is underprovisioned.

<figure><img src="../.gitbook/assets/dynamic-hardware-task-distribution.png" alt="A long-tail distribution where most PDFs need little CPU or RAM and a few need much more."><figcaption><p>Most parallel jobs are not made of identical work items. A fixed resource request has to pretend they are.</p></figcaption></figure>

This is not a user error. It is a bad abstraction.

The system is asking the user to predict something that will only become visible during execution.

## How Burla vertically scales CPU and RAM

A running machine cannot magically grow more CPUs or RAM. However, the number of workers running on that machine can change.

Dynamic hardware does not mean that a single computer literally changes size every second. It means that the effective resources available to each unit of work change while the job runs.

Burla workers (docker containers) do not have fixed CPU and RAM limits inside a node. If a node has many workers, they share the node's resources. If some workers are removed, the remaining workers have more CPU and RAM available to them. If more workers are added, the available resources per worker decrease.

So Burla can control the hardware available to each task by controlling concurrency.

<figure><img src="../.gitbook/assets/dynamic-hardware-worker-adjustment.png" alt="A server tray with worker blocks inside it, where smaller workers leave and a larger worker keeps running with more space."><figcaption><p>Burla does not resize a running computer. It resizes the amount of work competing for that computer.</p></figcaption></figure>

Burla starts aggressively, with one worker per available CPU. If CPU and RAM utilization is low, Burla adds workers. Those workers pull more inputs from the queue and start doing useful work.

When the node saturates, Burla removes workers. For CPU, saturation means the whole node is (almost) out of available CPU capacity, not that a single core is busy. For memory, this means RAM is exhausted and the system would otherwise start spilling to swap. The input being processed by a removed worker goes back onto the queue, where another worker can pick it up later.

This is how Burla adjusts CPU and RAM available to each task during runtime.

## Why killing work is not waste

At first, this sounds inefficient. A worker may run for a while, get killed, and have its input retried later. Isn't that wasted work?

Because the worker was running in capacity that otherwise would have been idle, this is not waste. If it finishes before the node becomes saturated, the job has made progress. If it does not finish, the input returns to the queue. Retry overhead is tiny, and the work item is assumed to be idempotent: it can run more than once without corrupting the result.

The alternative is not "run the task perfectly." The alternative is to leave that CPU or RAM unused.

Dynamic hardware uses extra capacity speculatively. Some speculative work completes. Some is evicted and retried. But because it runs only while the node has room, it does not reduce the progress of any work already using that machine.

A static scheduler has to reserve for uncertainty. It must choose a fixed worker count, a fixed worker size, or fixed resource requests before it knows what the tasks will actually do. That leaves unused capacity whenever the guess is conservative.

Dynamic Hardware can reproduce the conservative behavior when the machine is full, but it can also use idle capacity while it exists. If the extra work finishes, utilization improves. If the extra work is evicted, the system has lost very little, because the input simply returns to the queue.

Against a human guess, or a static scheduler, this is a much better position to be in.

## Vertical scaling requires horizontal scaling.

Adjusting worker count inside a node can vertically scale available resources per task.\
The cluster itself must also grow and shrink during runtime to make this actually useful.

With `grow=True`, if a job is no longer running at its maximum useful parallelism, Burla can add more nodes while the job is running. Those new nodes join the job, start pulling inputs from the queue, and do more work.

This matters because evicting workers on one node should not mean the whole job slows down. If a few large PDFs force some nodes to reduce their worker count, Burla can compensate by adding more machines. The job keeps moving, and the user does not have to decide in advance how many machines the workload needs.

<figure><img src="../.gitbook/assets/dynamic-hardware-grow-cluster.png" alt="A cluster grows by adding a new node that begins pulling document work from a queue."><figcaption><p>Dynamic Hardware is not only worker tuning inside one node. It is runtime infrastructure adaptation for the whole job.</p></figcaption></figure>

This is why "Dynamic Hardware" is a useful name even though the mechanism is really concurrency control.

From the user's perspective, the amount of hardware available to the workload is changing. A worker may effectively get more of a node when other workers are removed. The job may get more nodes when more parallelism is needed. The user does not manage those decisions directly.

They describe the work.

Burla adapts the infrastructure.

## The constraint

There is one important constraint: work items must be idempotent.

If a worker dies while parsing a PDF, that PDF may be parsed again later. The program must work correctly even if the same input is attempted more than once.

For many data-parallel workloads, this is a reasonable requirement. Parsing documents, embedding text, transforming files, running inference, or computing independent outputs is usually already structured this way. It is less appropriate for code that mutates shared external state without safeguards.

Dynamic Hardware gets its efficiency by treating unfinished work as retryable. If retrying a unit of work is dangerous, the workload needs a different execution model.

But when work items are idempotent, this is exactly the right trade.

## The next step is moving work without killing it

The long-term version of Dynamic Hardware should not need to kill workers at all.

Today, Burla runs workers in containers. The roadmap is to run workers inside VMs, then move the entire running VM to another machine when a node gets crowded. The worker, its memory, and any processes it started would keep running. The overloaded node would get more space. The migrated work would not be retried, because it would not have stopped.

This is called live migration, and it's already in production at scale: [Google Cloud uses live migration](https://docs.cloud.google.com/compute/docs/instances/live-migration-process) to move running VMs during host maintenance without interrupting the workload, rebooting the instance, or changing the VM's application state from the guest's point of view.

Using live migration for dynamic hardware would remove the idempotency requirement from this kind of rescheduling. Instead of killing the smallest workers around a large one, Burla could move them to machines with more room. If no good destination exists, Burla already has mechanisms for adding nodes during runtime, and can move the running workers there.

This is the strongest version of the idea: users do not specify CPU or RAM, Burla keeps every node close to 100% CPU or RAM utilization, no work is killed or thrown out, and the system rearranges hardware underneath while your workload is running.

## The interface should be the work

The deeper point is not that Burla chooses better defaults.

Better defaults would still accept the old premise: that the user's job is to think about hardware, and the system's job is to make that slightly less painful.

The old interface asks the user to translate intent into infrastructure:

> I want to parse these PDFs, so I need to decide how many machines to start, how much RAM each worker needs, how much CPU to allocate, and how much parallelism is safe.

Dynamic Hardware removes that translation step.

The user says:

> Parse these PDFs.

Burla watches the workload as it actually behaves. It fills idle CPU and RAM with useful work. It backs off when a node saturates. It gives large tasks more room by removing smaller ones. It adds nodes when the job needs more parallelism and removes them when they aren't needed.

The result is not merely easier. It is structurally more efficient than asking a person to guess.

A person choosing fixed resources upfront has less information than the system observing the job at runtime. The person sees the code and maybe a few examples. The system sees the actual CPU pressure, actual memory pressure, actual task distribution, and the actual long tail as it happens.

We shouldn't specify:

> Run X code on Y hardware.

We should just say:

> Run X code.

This is a better abstraction.
