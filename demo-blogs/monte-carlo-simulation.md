# Run the Monte Carlo you actually meant to run

In this example we:

* Simulate 1,000,000,000 option price paths.
* Split the run into 2,000 independent chunks.
* Return sums and squared sums, then compute the final price and error bar locally.

A smaller run is not the same experiment if the whole point is the error bar.

### Step 1: Plan independent chunks

The chunk id becomes part of the random seed, so reruns are reproducible without shared state.

```python
TOTAL = 1_000_000_000
N_CHUNKS = 2_000
PER_CHUNK = TOTAL // N_CHUNKS

params = {"S0": 100.0, "K": 95.0, "T": 1.0, "r": 0.01, "sigma": 0.3}
tasks = [(i, PER_CHUNK, params) for i in range(N_CHUNKS)]
```

### Step 2: Simulate on the worker

The worker is normal NumPy. It returns enough statistics to combine results exactly.

```python
def run_chunk(chunk_id: int, n: int, p: dict) -> dict:
    import numpy as np

    rng = np.random.default_rng(seed=42 + chunk_id)
    z = rng.standard_normal(n)
    st = p["S0"] * np.exp((p["r"] - 0.5 * p["sigma"] ** 2) * p["T"] + p["sigma"] * np.sqrt(p["T"]) * z)
    payoff = np.maximum(st - p["K"], 0.0) * np.exp(-p["r"] * p["T"])
    return {"n": n, "sum": float(payoff.sum()), "sum_sq": float((payoff ** 2).sum())}
```

### Step 3: Reduce locally

The result list is tiny because the workers do not return raw paths.

```python
from burla import remote_parallel_map

results = remote_parallel_map(run_chunk, tasks, func_cpu=1, func_ram=2, grow=True)

total_n = sum(r["n"] for r in results)
mean = sum(r["sum"] for r in results) / total_n
var = (sum(r["sum_sq"] for r in results) / total_n) - mean ** 2
se = math.sqrt(var / total_n)
```

### What's the point?

Monte Carlo is nice because the distributed-systems part should be almost nonexistent. Pick independent chunks, seed them cleanly, return sufficient statistics.

Do not ship every simulated path back to the client. That would turn a clean simulation into an output-size problem. Return `sum`, `sum_sq`, counts, quantile sketches, or top events. The worker function is the experiment. The input list is the queue.
