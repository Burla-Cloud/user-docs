Real Burla workloads for ML, data pipelines, production IO, and scientific computing.

## ML, embeddings, and search

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(178px,1fr));gap:12px;margin:16px 0 30px;">
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/gpu-embedding-demo.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/gpu-embedding-demo.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">GPU embeddings on A100s</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Embed 50,000 Wikipedia articles with a CUDA image, CPU download stage, GPU embedding stage, and shared vector artifacts.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/ml-inference-batch.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/ml-inference-batch.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Batch inference without serving</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Load a Hugging Face model once per worker and score Parquet batches without building an endpoint.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/arxiv-fossils.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/arxiv-fossils.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Embed the whole arXiv</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Cluster 2.7M abstracts and find isolated papers by running the embedding job at corpus scale.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/met-weirdest-art.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/met-weirdest-art.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Label-free visual search over the Met</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Fetch and embed Open Access museum images, then use FAISS to find visual matches without labels.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/airbnb-burla.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/airbnb-burla.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Multimodal Airbnb analysis</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Run listings, photos, CLIP, YOLOv8, reviews, and bootstrap confidence intervals across the public corpus.</span>
  </a>
</div>

## Full-corpus analysis

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(178px,1fr));gap:12px;margin:16px 0 30px;">
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/amazon-review-distiller.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/amazon-review-distiller.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">571M Amazon reviews</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Read 275 GB of JSONL with HTTP Range requests, deterministic scoring, and heap-based reducers.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/nyc-ghost-neighborhoods.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/nyc-ghost-neighborhoods.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">NYC taxi history</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Scan 2.76B taxi and FHV trips to find ghost, emergent, and recovered city zones.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/world-photo-index.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/world-photo-index.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">9.49M Flickr photos</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Reverse-geocode public photos and build country signatures from user-written tags.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/ghcn-rainiest-day.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/ghcn-rainiest-day.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">NOAA rain extremes</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Stream every yearly GHCN-Daily CSV, keep top heaps, and reduce station-level extremes.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/github-repo-summarizer.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/github-repo-summarizer.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">One million GitHub READMEs</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Export README Parquet from BigQuery, shard deterministic summarizers, and reduce category stats.</span>
  </a>
</div>

## Production data jobs

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(178px,1fr));gap:12px;margin:16px 0 30px;">
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/python-etl-no-airflow.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/python-etl-no-airflow.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">S3 to Postgres ETL</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Transform 10,000 gzipped JSON files while protecting Postgres with max_parallelism.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/image-dataset-resize.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/image-dataset-resize.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Millions of image resizes</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Chunk image keys, resize with Pillow, and stream progress as workers write outputs.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/parquet-parallel.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/parquet-parallel.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">One Parquet file per worker</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Compute per-file QA stats without starting Spark for a simple file-parallel job.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/pandas-apply-parallel.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/pandas-apply-parallel.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Pandas apply in parallel</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Partition a Parquet dataset and run ordinary pandas code on each worker.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/rate-limited-api-requests.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/rate-limited-api-requests.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Enrich millions of users through a rate-limited API</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Backfill user profiles while keeping provider limits explicit in chunk size, sleeps, and max_parallelism.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/parallel-web-scraping.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/parallel-web-scraping.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Crawl a million website pages without hiding failures</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Scrape static HTML with polite pacing, retries, error rows, and a global concurrency cap.</span>
  </a>
</div>

## Scientific and geospatial work

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(178px,1fr));gap:12px;margin:16px 0 30px;">
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/bioinformatics-alignment.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/bioinformatics-alignment.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Genome alignment</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Run bwa and samtools in a custom image with one FASTQ pair per worker.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/gdal-raster-processing.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/gdal-raster-processing.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">GDAL raster processing</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Compute NDVI one Sentinel tile at a time with rasterio and shared outputs.</span>
  </a>
  <a href="https://docs.burla.dev/examples/demo-walkthroughs/monte-carlo-simulation.md" style="display:block;border:1px solid #e6edf0;border-radius:8px;padding:12px 13px;text-decoration:none;color:inherit;background:#fff;min-height:142px;">
    <img src=".gitbook/assets/more-examples/monte-carlo-simulation.png" alt="" width="96" height="13" style="display:block;width:96px;height:13px;object-fit:cover;margin-bottom:10px;opacity:0.72;">
    <span style="display:block;font-weight:600;line-height:1.35;margin-bottom:9px;">Billion-path Monte Carlo</span>
    <span style="display:block;font-size:0.92em;line-height:1.45;color:#354247;">Return sums and squared sums from independent chunks, then reduce locally.</span>
  </a>
</div>
