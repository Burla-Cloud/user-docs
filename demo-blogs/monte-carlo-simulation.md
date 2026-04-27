# Run the Monte Carlo you actually meant to run

The compromised Monte Carlo is familiar: fewer paths, wider confidence intervals, and a note in the notebook that says "scale this later." The real experiment is the billion-path run, with the same payoff function, the same random seed scheme, and an error bar small enough to trust. The question you asked is not the experiment you ran if the sample count changed because your laptop was tired.

This demo prices an option with 1,000,000,000 simulated paths split into 2,000 independent chunks. Each worker returns sums, not raw paths, so the reduce step is tiny.

## what we built

The input planner is deliberately boring. Pick the total path count, pick a chunk count, and pass `(chunk_id, n, params)` to the worker. The chunk id becomes part of the RNG seed, so reruns are reproducible without shared state.

```python
TOTAL = 1_000_000_000
N_CHUNKS = 2_000
PER_CHUNK = TOTAL // N_CHUNKS

params = {"S0": 100.0, "K": 95.0, "T": 1.0, "r": 0.01, "sigma": 0.3}
tasks = [(i, PER_CHUNK, params) for i in range(N_CHUNKS)]
```

The worker is normal NumPy. It simulates terminal prices, computes discounted payoff, and returns enough statistics to combine results exactly.

```python
def run_chunk(chunk_id: int, n: int, p: dict) -> dict:
    import numpy as np

    rng = np.random.default_rng(seed=42 + chunk_id)
    z = rng.standard_normal(n)
    st = p["S0"] * np.exp((p["r"] - 0.5 * p["sigma"] ** 2) * p["T"] + p["sigma"] * np.sqrt(p["T"]) * z)
    payoff = np.maximum(st - p["K"], 0.0) * np.exp(-p["r"] * p["T"])
    return {"n": n, "sum": float(payoff.sum()), "sum_sq": float((payoff ** 2).sum())}
```

## how the pipeline works

Burla runs the 2,000 chunks as independent jobs. There is no queue service to stand up and no MPI shape to force onto an embarrassingly parallel simulation.

```python
from burla import remote_parallel_map

results = remote_parallel_map(run_chunk, tasks, func_cpu=1, func_ram=2, grow=True)

total_n = sum(r["n"] for r in results)
mean = sum(r["sum"] for r in results) / total_n
var = (sum(r["sum_sq"] for r in results) / total_n) - mean ** 2
se = math.sqrt(var / total_n)
```

## why this demo is interesting

Monte Carlo is a clean example because the infrastructure changes the statistical object. A 10-million-path run is not a noisy preview of a billion-path run when you are making decisions from tail behavior or standard error. It changes which estimates you are willing to trust. The nice part is that the code does not become distributed-systems code. The random seed rule and the sufficient statistics are the only design choices that matter.

A reader can use the same pattern for any simulation where trials are independent: price paths, bootstrap samples, parameter sweeps, particle simulations, or independent chains. The important habit is to reduce on the worker. Return `sum`, `sum_sq`, counts, quantile sketches, or top-K events. Do not ship every simulated path back to the client.

## how to build your version

Start by making the unit of work independent. For pricing, that means one seed and one path count. For posterior sampling, it might be one chain. For physics, one parameter point. Return compact sufficient statistics instead of arrays. Then call `remote_parallel_map` with enough chunks to keep the workers busy and aggregate on the client.

## why Burla fits

This is exactly the case where scheduler plumbing is worse than the math. Burla removes Slurm scripts, Batch job definitions, worker fleets, and shared filesystem setup. The worker function is the experiment. The input list is the queue.

## what the small run misses

A compromised 10-million-path run can tell you the code executes. It cannot tell you the tail estimate you actually care about. The real billion-path run gives you the price and the error bar; the smaller run mostly tells you why you need the larger one.
