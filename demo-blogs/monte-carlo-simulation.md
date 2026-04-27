# A billion-path Monte Carlo in plain Python

Monte Carlo is the cleanest possible parallel job: split random draws into independent chunks, run the same function everywhere, then combine sums. That makes it a good Burla demo because there is no data-engineering distraction.

This repo prices a simple European call option with one billion simulated paths. It divides the work into 2,000 chunks, seeds each chunk independently, computes payoff sums and squared sums, then reduces them into a mean and standard error.

## What The Job Does

The compromised version runs a few million paths on a laptop. That is enough to test the payoff formula and the random seed logic. It is not enough if you care about a tight standard error or want to sweep parameters.

The real version sets `TOTAL = 1_000_000_000`, `N_CHUNKS = 2_000`, and `PER_CHUNK = TOTAL // N_CHUNKS`. Each task carries a `chunk_id`, a path count, and the option parameters. Workers generate standard normals with NumPy, compute terminal stock prices, discount payoffs, and return only three numbers: `n`, `sum`, and `sum_sq`.

The driver adds those partial sums and computes the final standard error. There is no shared storage and no large return payload.

## The Code Shape

The remote function is exactly the statistical unit of work.

```python
def run_chunk(chunk_id: int, n: int, p: dict) -> dict:
    import numpy as np

    rng = np.random.default_rng(seed=42 + chunk_id)
    Z = rng.standard_normal(n)
    ST = p["S0"] * np.exp(
        (p["r"] - 0.5 * p["sigma"] ** 2) * p["T"]
        + p["sigma"] * np.sqrt(p["T"]) * Z
    )
    payoff = np.maximum(ST - p["K"], 0.0) * np.exp(-p["r"] * p["T"])
    return {
        "chunk_id": chunk_id,
        "n": n,
        "sum": float(payoff.sum()),
        "sum_sq": float((payoff ** 2).sum()),
    }
```

The map call sends 2,000 chunks:

```python
tasks = [(i, PER_CHUNK, params) for i in range(N_CHUNKS)]
results = remote_parallel_map(
    run_chunk,
    tasks,
    func_cpu=1,
    func_ram=2,
    grow=True,
)
```

The reduce is local arithmetic:

```python
total_n = sum(r["n"] for r in results)
mean = sum(r["sum"] for r in results) / total_n
var = (sum(r["sum_sq"] for r in results) / total_n) - mean ** 2
se = math.sqrt(var / total_n)
```

## Why It Matters

This is the baseline shape for embarrassingly parallel numerical work. If the job can be expressed as "return sufficient statistics per chunk," the distributed part stays simple and the reduce stays exact.

Burla's role is to start enough Python workers that a billion-path run behaves like a short script. The simulation remains NumPy, not MPI, Slurm, or a custom job definition.
