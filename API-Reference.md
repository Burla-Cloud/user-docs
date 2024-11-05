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

Run an arbitrary python function on many remote computers at the same time.

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

Run provided `function_` on each item in `inputs` at the same time, each on a separate CPU, up to 256 CPUs (as of 10/11). If more than 256 inputs are provided, inputs are queued and processed sequentially on each worker.

| **Parameters**    |                                                                                                                                                                                         |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**          | **Description**                                                                                                                                                                         |
| `function_`       | <p><code>Callable</code></p><p>Python function. Must have single input argument, eg: <code>function_(inputs[0])</code> does not raise an exception.</p>                                 |
| `inputs`          | <p><code>Iterable[Any]</code></p><p>Iterable of elements passable to <code>function_</code>.</p>                                                                                        |
| `func_cpu`        | <p><code>int</code></p><p>(Optional) Number of CPU's made available to every instance of <code>function_</code>. The maximum possible value is <code>32</code>.</p>                     |
| `func_ram`        | <p><code>int</code></p><p>(Optional) Amount of RAM (GB) made available to every instance of <code>function_</code>. The maximum possible value is <code>128</code>.</p>                 |
| `spinner`         | <p><code>bool</code></p><p>(Optional) Set to <code>False</code> to prevent status indicator/spinner from being displayed.</p>                                                           |
| `max_parallelism` | <p><code>int</code></p><p>(Optional) Maximum number of <code>function_</code> instances allowed to be running at the same time. Defaults to available-cpus / <code>func_cpu</code>.</p> |
| `api_key`         | <p><code>str</code></p><p>(Optional) API key, for use in deployment environments where <code>burla login</code> cannot be run.</p>                                                      |



| **Returns** |                                                                            |
| ----------- | -------------------------------------------------------------------------- |
| **Type**    | **Description**                                                            |
| `Generator` | Generator yielding objects returned by `function_` in no particular order. |







***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
