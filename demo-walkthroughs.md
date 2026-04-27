---
description: Walkthroughs of real Burla examples, from public datasets to GPU embedding jobs.
---

# Demo walkthroughs

Most examples get smaller before anyone admits it. The notebook uses the clean files, skips the weird tail, avoids the GPU stage, or samples the corpus until the job fits on one machine. The question stays ambitious in the title, but the experiment quietly changes.

These walkthroughs show the larger version. Each one explains what we built, where the work was split, which code runs on workers, how to build a similar pipeline, and what the smaller experiment would miss.

## discovery projects

These examples are interesting because the answer is not known before the scan. The result depends on letting the whole corpus vote.

* [Test Airbnb hypotheses at public-data scale](demo-blogs/airbnb-burla.md): listings, photos, CLIP, YOLOv8, reviews, and bootstrap confidence intervals across the public Inside Airbnb corpus.
* [Distill 571 million reviews with byte ranges, not vibes](demo-blogs/amazon-review-distiller.md): HTTP Range reads over 275 GB of Amazon review JSONL, with deterministic scoring and heap-based reducers.
* [Map what the world photographed by processing the whole Flickr slice](demo-blogs/world-photo-index.md): reverse-geocode 9.49M public Flickr photos and build country signatures from user-written tags.
* [Scan NYC taxi history without sampling away the city](demo-blogs/nyc-ghost-neighborhoods.md): process 2.76B taxi and FHV trips to find ghost, emergent, and recovered zones.
* [Find extinct ideas by embedding the whole arXiv](demo-blogs/arxiv-fossils.md): embed 2.7M abstracts, cluster topics, and find the loneliest paper in the corpus.
* [Find visual twins in the Met without using museum labels](demo-blogs/met-weirdest-art.md): fetch and embed Met Open Access images, then use FAISS to surface cross-century visual matches.
* [Find the rainiest station day in NOAA's archive](demo-blogs/ghcn-rainiest-day.md): stream every yearly GHCN-Daily CSV, keep top heaps, and reduce station extremes.
* [Summarize a million GitHub READMEs without an LLM](demo-blogs/github-repo-summarizer.md): export README Parquet from BigQuery, shard deterministic summarizers, and reduce category stats.

## build patterns

These are the reusable shapes: one file per worker, one chunk per worker, one API slice per worker, one model batch per worker.

* [Resize millions of images as an S3-in, S3-out job](demo-blogs/image-dataset-resize.md): chunk image keys, resize with Pillow, and stream progress as workers finish.
* [Run genome alignment without building a scheduler](demo-blogs/bioinformatics-alignment.md): use a custom image with `bwa` and `samtools`, one FASTQ pair per worker.
* [Scrape URLs with bounded parallelism](demo-blogs/parallel-web-scraping.md): scrape static HTML with retry, backoff, and a global concurrency cap.
* [Run ETL without making Airflow the center](demo-blogs/python-etl-no-airflow.md): fan out S3 file transforms while protecting Postgres with `max_parallelism`.
* [Process raster tiles with GDAL on many workers](demo-blogs/gdal-raster-processing.md): compute NDVI one Sentinel tile at a time with `rasterio` and shared outputs.
* [Run batch inference without turning it into serving](demo-blogs/ml-inference-batch.md): load a Hugging Face model once per worker and score Parquet batches.
* [Make millions of API calls with a global cap](demo-blogs/rate-limited-api-requests.md): encode provider rate limits in chunk size, per-worker sleep, and `max_parallelism`.
* [Run pandas apply without rewriting the transform](demo-blogs/pandas-apply-parallel.md): partition a Parquet dataset and run ordinary pandas code on each worker.
* [Scan one Parquet file per worker](demo-blogs/parquet-parallel.md): compute per-file stats without starting Spark for a file QA job.
* [Run a billion Monte Carlo paths in plain Python](demo-blogs/monte-carlo-simulation.md): return sums and squared sums from independent chunks, then reduce locally.
* [Put the embedding model on A100s, then ask the search question](demo-blogs/gpu-embedding-demo.md): split CPU download from GPU embedding and query search.

## how to read these

Start with the problem that looks most like yours. Copy the shape, not the dataset. If your data is line-oriented, look at the Amazon walkthrough. If each file is independent, look at Parquet or GHCN. If the work changes hardware mid-pipeline, look at Airbnb or the GPU embedding demo.

The point is to keep the experiment honest. If the real question needs every document, every image, every monthly file, or the actual GPU model, the walkthrough should make that version the easy one to run.
