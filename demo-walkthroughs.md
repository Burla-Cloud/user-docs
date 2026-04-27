---
description: Real Burla examples for ML, data science, and production data work.
---

# Other Examples

Use these when you want to see Burla beyond the three main examples: GPU embedding jobs, batch inference, public-data scans, file-parallel ETL, scientific pipelines, and API workloads with real external limits.

Each example is built around a production constraint: how inputs are partitioned, which container and hardware profile runs each stage, how results are reduced, and which constraint the toy version would miss.

<figure><img src=".gitbook/assets/vector-field-icon (2).svg" alt="A field of arrows representing many independent tasks running in parallel." width="45%"><figcaption></figcaption></figure>

{% hint style="info" %}
Start with the workload shape. If the work is one file per task, look at Parquet, GHCN, images, or ETL. If the job changes hardware mid-pipeline, look at Airbnb or GPU embeddings. If the bottleneck is an external system, look at APIs, scraping, or Postgres.
{% endhint %}

## ML, embeddings, and search

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>GPU embeddings on A100s</strong></td>
      <td>Embed 50,000 Wikipedia articles with a CUDA image, CPU download stage, GPU embedding stage, and shared vector artifacts.</td>
      <td><a href="demo-blogs/gpu-embedding-demo.md">gpu-embedding-demo.md</a></td>
    </tr>
    <tr>
      <td><strong>Batch inference without serving</strong></td>
      <td>Load a Hugging Face model once per worker and score Parquet batches without building an endpoint.</td>
      <td><a href="demo-blogs/ml-inference-batch.md">ml-inference-batch.md</a></td>
    </tr>
    <tr>
      <td><strong>Embed the whole arXiv</strong></td>
      <td>Cluster 2.7M abstracts and find isolated papers by running the embedding job at corpus scale.</td>
      <td><a href="demo-blogs/arxiv-fossils.md">arxiv-fossils.md</a></td>
    </tr>
    <tr>
      <td><strong>Label-free visual search over the Met</strong></td>
      <td>Fetch and embed Open Access museum images, then use FAISS to find visual matches without labels.</td>
      <td><a href="demo-blogs/met-weirdest-art.md">met-weirdest-art.md</a></td>
    </tr>
    <tr>
      <td><strong>Multimodal Airbnb analysis</strong></td>
      <td>Run listings, photos, CLIP, YOLOv8, reviews, and bootstrap confidence intervals across the public corpus.</td>
      <td><a href="demo-blogs/airbnb-burla.md">airbnb-burla.md</a></td>
    </tr>
  </tbody>
</table>

## Full-corpus analysis

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>571M Amazon reviews</strong></td>
      <td>Read 275 GB of JSONL with HTTP Range requests, deterministic scoring, and heap-based reducers.</td>
      <td><a href="demo-blogs/amazon-review-distiller.md">amazon-review-distiller.md</a></td>
    </tr>
    <tr>
      <td><strong>NYC taxi history</strong></td>
      <td>Scan 2.76B taxi and FHV trips to find ghost, emergent, and recovered city zones.</td>
      <td><a href="demo-blogs/nyc-ghost-neighborhoods.md">nyc-ghost-neighborhoods.md</a></td>
    </tr>
    <tr>
      <td><strong>9.49M Flickr photos</strong></td>
      <td>Reverse-geocode public photos and build country signatures from user-written tags.</td>
      <td><a href="demo-blogs/world-photo-index.md">world-photo-index.md</a></td>
    </tr>
    <tr>
      <td><strong>NOAA rain extremes</strong></td>
      <td>Stream every yearly GHCN-Daily CSV, keep top heaps, and reduce station-level extremes.</td>
      <td><a href="demo-blogs/ghcn-rainiest-day.md">ghcn-rainiest-day.md</a></td>
    </tr>
    <tr>
      <td><strong>One million GitHub READMEs</strong></td>
      <td>Export README Parquet from BigQuery, shard deterministic summarizers, and reduce category stats.</td>
      <td><a href="demo-blogs/github-repo-summarizer.md">github-repo-summarizer.md</a></td>
    </tr>
  </tbody>
</table>

## Production data jobs

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>S3 to Postgres ETL</strong></td>
      <td>Transform 10,000 gzipped JSON files while protecting Postgres with <code>max_parallelism</code>.</td>
      <td><a href="demo-blogs/python-etl-no-airflow.md">python-etl-no-airflow.md</a></td>
    </tr>
    <tr>
      <td><strong>Millions of image resizes</strong></td>
      <td>Chunk image keys, resize with Pillow, and stream progress as workers write outputs.</td>
      <td><a href="demo-blogs/image-dataset-resize.md">image-dataset-resize.md</a></td>
    </tr>
    <tr>
      <td><strong>One Parquet file per worker</strong></td>
      <td>Compute per-file QA stats without starting Spark for a simple file-parallel job.</td>
      <td><a href="demo-blogs/parquet-parallel.md">parquet-parallel.md</a></td>
    </tr>
    <tr>
      <td><strong>Pandas apply in parallel</strong></td>
      <td>Partition a Parquet dataset and run ordinary pandas code on each worker.</td>
      <td><a href="demo-blogs/pandas-apply-parallel.md">pandas-apply-parallel.md</a></td>
    </tr>
    <tr>
      <td><strong>Rate-limited API jobs</strong></td>
      <td>Make millions of calls while encoding provider limits in chunk size, sleeps, and <code>max_parallelism</code>.</td>
      <td><a href="demo-blogs/rate-limited-api-requests.md">rate-limited-api-requests.md</a></td>
    </tr>
    <tr>
      <td><strong>Bounded web scraping</strong></td>
      <td>Scrape static HTML with retries, backoff, and a global concurrency cap.</td>
      <td><a href="demo-blogs/parallel-web-scraping.md">parallel-web-scraping.md</a></td>
    </tr>
  </tbody>
</table>

## Scientific and geospatial work

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Genome alignment</strong></td>
      <td>Run <code>bwa</code> and <code>samtools</code> in a custom image with one FASTQ pair per worker.</td>
      <td><a href="demo-blogs/bioinformatics-alignment.md">bioinformatics-alignment.md</a></td>
    </tr>
    <tr>
      <td><strong>GDAL raster processing</strong></td>
      <td>Compute NDVI one Sentinel tile at a time with <code>rasterio</code> and shared outputs.</td>
      <td><a href="demo-blogs/gdal-raster-processing.md">gdal-raster-processing.md</a></td>
    </tr>
    <tr>
      <td><strong>Billion-path Monte Carlo</strong></td>
      <td>Return sums and squared sums from independent chunks, then reduce locally.</td>
      <td><a href="demo-blogs/monte-carlo-simulation.md">monte-carlo-simulation.md</a></td>
    </tr>
  </tbody>
</table>

## How to read these

Copy the partitioning strategy, not the dataset. The useful part is usually how inputs are split, which code runs inside the worker, what hardware is requested, and where the reduction happens.

If a toy version would skip the tail, remove CUDA, sample away bad files, or hide sink pressure, it is not the same experiment. These examples show the version you would trust before making a product or research decision.
