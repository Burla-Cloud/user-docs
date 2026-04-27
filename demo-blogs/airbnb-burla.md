# Airbnb at full public-data scale

Airbnb data is easy to sample and hard to exhaust. Inside Airbnb publishes listings and reviews for major cities, but the interesting questions need all of it: photos, review text, city metadata, and enough compute to avoid turning the project into a month of batch jobs.

This demo runs across 1,097,241 listings in 116 cities, scrapes 1,406,718 photo URLs, CLIP-scores 1,243,339 images, sends 48,122 candidate images through YOLOv8 on A100s, and scores 50,686,612 reviews through a three-tier funnel ending with Claude Haiku on the top 10,000. The point is not that Airbnb is special. The point is that a messy public-data question often becomes a systems problem before it becomes a statistics problem.

## What The Job Does

The compromised version is a city or two, a few thousand listings, and maybe one visual feature. That version is useful for debugging. It also lies by omission: different cities have different photo habits, review norms, and scraping failure modes.

The real version keeps the whole shape of the dataset. It validates every live Inside Airbnb city, cleans each city's listings, scrapes public listing pages, falls back to listing-data photos where public pages hit Datadome, scores every image with CLIP, runs YOLOv8 on selected high-signal images, scores review text, then tests five hypotheses with bootstrap confidence intervals.

Each stage writes Parquet to Burla's shared GCS-backed filesystem under `/workspace/shared`, then later stages read those Parquets. That matters because the dataset is too large to shuttle through the driver after every step.

## The Code Shape

The GPU image stage shows the pattern. First, one large CPU worker chooses the top image candidates from the CPU-scored Parquet. Then A100 workers run YOLOv8 over batches.

```python
from burla import remote_parallel_map

[picked] = remote_parallel_map(
    select_top_k_images,
    [TopKImagesArgs(
        images_cpu_path=f"{SHARED_ROOT}/images_cpu.parquet",
        top_n_per_axis=TOP_N_PER_AXIS,
    )],
    func_cpu=8,
    func_ram=64,
    max_parallelism=1,
    grow=True,
    spinner=True,
)

results = remote_parallel_map(
    gpu_detect_image_batch,
    batches,
    func_cpu=8,
    func_ram=64,
    func_gpu="A100_40G",
    max_parallelism=n_workers,
    grow=True,
    spinner=True,
)
```

The review stage is more like a search funnel. It ingests `reviews.csv.gz` files by city, merges them into `reviews_raw.parquet`, rechunks by row group, runs a cheap heuristic over every review, embeds the top 200,000 with `sentence-transformers/all-MiniLM-L6-v2`, clusters with MiniBatchKMeans, and sends the top 10,000 to Claude.

That structure is honest about cost. A model call over 50 million reviews would be wasteful. A regex pass over everything and a model pass over the weird tail is the right split.

## Why It Matters

Most data science demos stop right before the part that hurts: heterogeneous input files, image downloads, GPU model weight placement, text funnels, partial scrape failure, and checkpointed intermediate data. This one includes those parts.

Burla is mostly boring here, which is the useful bit. The project is normal Python stages and worker functions. The cluster exists so that a million-listing, 50-million-review run can be treated like one local pipeline with a lot of workers behind it.
