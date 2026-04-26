---
description: Why dynamic hardware should be a normal part of ML and data programs.
---

# The Experiment You Don't Run

<figure><img src=".gitbook/assets/blog-hero-experiment.jpg" alt="A lone scientist works at a small desk under a single lamp, dwarfed by rows of unused supercomputers stretching to the horizon."><figcaption></figcaption></figure>

Most ML and data work starts as a question. Then a different question barges in.

You open a notebook to find out whether something works. A minute later you are choosing an instance type, guessing memory, checking whether the Docker image has CUDA, wondering if Spark is overkill, and asking how painful this will be to rerun.

This feels normal because we do it all the time.

The cloud gave us more computers than we know how to use. But most tools still ask us to pick one before the work has shown us what it needs.

That changes the experiment.

## Infrastructure has veto power

Say you need to build search over company documents: PDFs, contracts, invoices, support exports, customer uploads, and a few old scans nobody wants to touch.

The clean plan is boring:

1. Parse documents.
2. Chunk text.
3. Embed chunks.
4. Build an index.
5. Run evals.

The real job appears after the first pass. Some PDFs are scanned. Some have tables that matter. Some are 900 pages. Some are corrupt. Some are in the wrong language. Some need OCR. Some need a vision model. Some should be skipped. Some deserve a slow parser only if a cheap parser thinks they matter.

The program you want is a decision tree.

Start cheap and wide. Inspect every file. Send normal PDFs through the fast path. Send scans to OCR. Send table-heavy documents to a different container. Fan chunks out to GPUs for embeddings. Run a broad eval grid. Rerun low-confidence cases. Build the index from what survived.

That is the obvious experiment.

But it is often not the experiment people run.

They sample 500 documents. Skip OCR for now. Use one embedding model. Hand-pick 20 eval questions. Promise to come back later. The prototype works, sort of, because it was shaped around what was easy to run.

This is the cost of infrastructure friction: it gives your curiosity a budget before the experiment starts.

## Hardware should be control flow

Good experiments escalate: start cheap, keep the interesting cases, spend expensive hardware only where the data earns it.

This is how good scientists spend attention. Our tools often ask for the spending plan before there is evidence.

So the question shrinks.

Not "what is the real experiment?"

"What experiment fits in the machine I already picked?"

Hardware should be part of the program's control flow. If a branch needs OCR, run it in an OCR container. If another needs CUDA, run it on GPUs. If the next step is a million independent files, fan out across a thousand machines for a few minutes. When the work becomes aggregation, fan back in.

Not a platform migration. Not queue wiring. Not a YAML ceremony.

Code.

```python
profiles = map(inspect_file, files, cpu=2)

parsed = map(parse_pdf, normal_pdfs, image="python:3.12", cpu=4)

ocr_text = map(run_ocr, scanned_pdfs, image="ocr-stack", cpu=16)

embeddings = map(embed, chunks, image="pytorch-cuda", gpu="A100")

scores = map(evaluate, eval_grid, cpu=8)
```

I do not care much about the exact syntax. I care where the decision lives.

The person writing the pipeline knows why one step needs GDAL, why another needs a GPU, why bad files should be isolated, and why only the winners deserve the expensive pass. That logic belongs in the program.

## What becomes possible

The useful new thing is writing the ambitious version first.

For model search, do a cascade. Train 10,000 cheap models on small samples. Keep 1,000. Retrain those on more data. Keep 100. Add expensive features. Keep 10. Run calibration and slice analysis. Early stages want cheap CPU parallelism. Later stages may want memory or GPUs. The static-cluster version feels like a pipeline project. The dynamic version feels like a loop.

For RAG evals, run the full matrix: chunk sizes, embedding models, retrieval depths, rerankers, prompts, model versions, judges, datasets. Cheap filters first. Expensive judges later. Evaluate the actual space, not the tiny version that fits in your patience.

For geospatial work, stop building one cursed Docker image with every dependency known to humanity. Tile satellite scenes in an `osgeo/gdal` container. Run segmentation on GPUs. Polygonize on CPUs. Aggregate by region. Rerun cloudy or low-confidence tiles with a slower path.

These are the programs people would write if infrastructure stopped interrupting.

## Adapters are not enough

Docker makes environments portable. Queues make functions distributable. Spark makes dataframes run across clusters. Workflow engines make scripts schedulable. Hosted notebooks put the laptop somewhere bigger.

These tools are useful. Some are great. But they still make the user think in infrastructure nouns: clusters, workers, queues, images, DAGs, schedulers, node types.

That is fine if the job is infrastructure. It is not fine if the job is figuring out which documents need OCR, whether a reranker improves retrieval, why a fraud model fails on one customer segment, or which features are worth computing at all.

Someone still has to build the infrastructure. The next step is letting more ML and data people stay with the problem longer.

Functions. Inputs. Outputs. Hardware requirements. Containers where they matter. Logs. Exceptions. Fan out. Fan in.

That covers a lot of work.

## Self-hosting is not a footnote

One constraint decides whether any of this matters: the data often cannot move.

Healthcare data, financial data, customer documents, internal logs, proprietary datasets, giant buckets already sitting in GCP. These are not edge cases. This is the work.

The best developer experience in the world is useless if the data has to leave the customer's cloud account. The magic has to happen where the bucket already is.

The interesting thing is cloud compute that feels local while running inside your own cloud. Developers can fan out, switch hardware, switch containers, and stream logs back. The organization keeps IAM, audit logs, cost controls, and network boundaries.

## This is why the cloud still feels early

The cloud already won at the hardware layer. Nobody needs to be convinced that a thousand machines can exist.

The gap is that using them still feels too deliberate. For ML and data work, the cloud often feels like a bigger version of the old machine model. Pick the machine. Enter the machine. Run the code. Hope you picked right.

But the work is not static. A script should start on your laptop, inspect the data, route weird cases, grab GPUs, switch containers, fan out, fan in, escalate promising branches, and shut everything down when it is done.

That changes which workflows are worth attempting.

The next big improvement in cloud compute will not be bigger machines. The machines already got big.

It will be infrastructure losing its veto over curiosity.

## Addendum

Burla is our attempt at this. The main function is:

```python
from burla import remote_parallel_map

results = remote_parallel_map(my_function, inputs)
```

Your function runs across remote machines. Prints and exceptions come back locally. Different calls can use different CPUs, GPUs, and Docker containers.

A pipeline can look like this:

```python
parsed = remote_parallel_map(
    parse_file,
    files,
    image="python:3.12",
    func_cpu=4,
)

embeddings = remote_parallel_map(
    embed,
    parsed,
    image="pytorch/pytorch:2.5.1-cuda12.4-cudnn9-runtime",
    func_gpu="A100",
)

index_parts = remote_parallel_map(
    build_index,
    embeddings,
    func_cpu=32,
)
```

The self-hosted version installs into your own GCP project, so data and compute stay in your cloud.

We have demos, like [processing 2.4TB of Parquet files on 10,000 CPUs in 76 seconds](examples/process-2.4tb-of-parquet-files-in-76s.md). But the benchmark is not the point.

The point is the experiment you run when infrastructure stops saying no first.
