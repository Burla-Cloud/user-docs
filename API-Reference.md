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

# API-Reference

## API-Reference

### `burla.remote_parallel_map`

Run an arbitrary python function on many remote computers at the same time.\
See our [overview](overview.md) for a more detailed description of how to use `remote_parallel_map.`

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

Run provided `function_` on each item in `inputs` at the same time, each on a separate CPU. If more inputs are provided than there are CPU's, they are queued and processed sequentially on each worker.

If the provided `function_` raises an exception, the exception, including stack trace is reraised on the client machine.

| **Parameters**    |                                                                                                                                                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**          | **Description**                                                                                                                                                                                                   |
| `function_`       | <p><code>Callable</code></p><p>Python function. <strong>Must have single input argument</strong>, eg: <code>function_(inputs[0])</code> does not raise an exception.</p>                                          |
| `inputs`          | <p><code>List[Any]</code></p><p>Iterable of elements passable to <code>function_</code>.</p>                                                                                                                      |
| `func_cpu`        | <p><code>int</code></p><p>(Optional) Number of CPU's made available to every instance of <code>function_</code>. The maximum possible value is determined by your cluster machine type.</p>                       |
| `func_ram`        | <p><code>int</code></p><p>(Optional) Amount of RAM (GB) made available to every instance of <code>function_</code>. The maximum possible value is determined by your cluster machine type.</p>                    |
| `background`      | <p><code>bool</code><br>(Optional) <code>remote_parallel_map</code> will return as soon as your inputs and function have been uploaded. The job will continue to run independently in the background.</p>         |
| `spinner`         | <p><code>bool</code></p><p>(Optional) Set to <code>False</code> to prevent status indicator/spinner from being displayed.</p>                                                                                     |
| `generator`       | <p><code>bool</code><br>(Optional) Set to <code>True</code> to return a <code>Generator</code> instead of a <code>List</code>. The generator will yield outputs as they are produced, instead of all at once.</p> |
| `max_parallelism` | <p><code>int</code></p><p>(Optional) Maximum number of <code>function_</code> instances allowed to be running at the same time. Defaults to #available-cpus divided by <code>func_cpu</code>.</p>                 |



| **Returns**           |                                                                                                                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Type**              | **Description**                                                                                                                                                                 |
| `List` or `Generator` | List of objects returned by `function_` in no particular order. If `Generator=True`, returns generator yielding objects returned by `function_` in the order they are produced. |







***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
