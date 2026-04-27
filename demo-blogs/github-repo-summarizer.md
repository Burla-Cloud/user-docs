# Summarizing a million GitHub READMEs

GitHub's public BigQuery dataset is big enough that even finding one README per repo becomes a data job. This demo exports more than a million READMEs to Parquet, uploads that Parquet to Burla's shared filesystem, and fans out deterministic README summarizers across hundreds of workers.

There is no LLM in the summarizer. It extracts the H1 title, first prose paragraph, primary language, install style, code-fence count, badges, category scores, and token counts. That makes the output cheap, repeatable, and easy to reduce.

## What The Job Does

The compromised version uses BigQuery sample tables. The repo notes explain why that fails: `sample_files` and `sample_contents` only yield about 16,000 matched READMEs because the samples are not correlated. It is enough for a demo UI, but not for a claim about GitHub at scale.

The real version scans the full `github_repos.files` and `github_repos.contents` tables, picks one README per repo, joins primary language from `github_repos.languages`, and streams Arrow batches into `samples/readmes.parquet`. The estimated scan is about 3 TB. That is an explicit cost choice in `prepare.py`.

Then `scale.py` uploads the Parquet to `/workspace/shared/grs/readmes.parquet`, partitions rows into 600 stripes, and runs one worker per stripe. Each worker streams row groups, processes every nth row, and writes one JSON shard to `/workspace/shared/grs/shards`.

## The Code Shape

The worker avoids loading the full Parquet in memory. It iterates Arrow batches and selects rows by modulo.

```python
def summarize_shard(shard_idx: int, n_shards: int) -> dict:
    import pyarrow.parquet as pq

    pf = pq.ParquetFile(PARQUET_PATH)
    global_idx = 0
    for batch in pf.iter_batches(
        batch_size=4000,
        columns=["repo_name", "lang", "path", "size", "content"],
    ):
        for j in range(batch.num_rows):
            g = global_idx + j
            if (g % n_shards) != shard_idx:
                continue
            s = summarize_row(
                repo_list[j] or "",
                lang_list[j] or "",
                path_list[j] or "",
                int(size_list[j] or 0),
                content_list[j] or "",
            )
            rows.append(s)
        global_idx += batch.num_rows
```

The fan-out is direct:

```python
jobs = [(i, args.shards) for i in range(args.shards)]
results = remote_parallel_map(
    summarize_shard,
    jobs,
    func_cpu=args.func_cpu,
    func_ram=args.func_ram,
    grow=True,
    max_parallelism=args.parallelism,
    spinner=True,
)
```

The summarizer itself is old-fashioned text processing: regexes for install commands, category word weights, README badges, code fences, and token frequency.

## Why It Matters

This demo is useful because it respects the boring bottlenecks: BigQuery scan cost, Parquet export size, cluster upload, row-group iteration, and bounded worker memory.

For data engineers, it is a clean pattern for "run the same cheap function over a very wide table." Burla handles the worker pool and the shared file path. The job logic remains a deterministic Python function over one README at a time.
