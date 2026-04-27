Real Burla workloads for ML, data pipelines, production IO, and scientific computing.

## ML, embeddings, and search

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>GPU embeddings on A100s</strong></td>
      <td>Embed 50,000 Wikipedia articles with a CUDA image, CPU download stage, GPU embedding stage, and shared vector artifacts.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/gpu-embedding-demo">gpu-embedding-demo.md</a></td>
      <td><a href=".gitbook/assets/more-examples/gpu-embedding-demo.png">gpu-embedding-demo.png</a></td>
    </tr>
    <tr>
      <td><strong>Batch inference without serving</strong></td>
      <td>Load a Hugging Face model once per worker and score Parquet batches without building an endpoint.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/ml-inference-batch">ml-inference-batch.md</a></td>
      <td><a href=".gitbook/assets/more-examples/ml-inference-batch.png">ml-inference-batch.png</a></td>
    </tr>
    <tr>
      <td><strong>Embed the whole arXiv</strong></td>
      <td>Cluster 2.7M abstracts and find isolated papers by running the embedding job at corpus scale.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/arxiv-fossils">arxiv-fossils.md</a></td>
      <td><a href=".gitbook/assets/more-examples/arxiv-fossils.png">arxiv-fossils.png</a></td>
    </tr>
    <tr>
      <td><strong>Label-free visual search over the Met</strong></td>
      <td>Fetch and embed Open Access museum images, then use FAISS to find visual matches without labels.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/met-weirdest-art">met-weirdest-art.md</a></td>
      <td><a href=".gitbook/assets/more-examples/met-weirdest-art.png">met-weirdest-art.png</a></td>
    </tr>
    <tr>
      <td><strong>Multimodal Airbnb analysis</strong></td>
      <td>Run listings, photos, CLIP, YOLOv8, reviews, and bootstrap confidence intervals across the public corpus.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/airbnb-burla">airbnb-burla.md</a></td>
      <td><a href=".gitbook/assets/more-examples/airbnb-burla.png">airbnb-burla.png</a></td>
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
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>571M Amazon reviews</strong></td>
      <td>Read 275 GB of JSONL with HTTP Range requests, deterministic scoring, and heap-based reducers.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/amazon-review-distiller">amazon-review-distiller.md</a></td>
      <td><a href=".gitbook/assets/more-examples/amazon-review-distiller.png">amazon-review-distiller.png</a></td>
    </tr>
    <tr>
      <td><strong>NYC taxi history</strong></td>
      <td>Scan 2.76B taxi and FHV trips to find ghost, emergent, and recovered city zones.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/nyc-ghost-neighborhoods">nyc-ghost-neighborhoods.md</a></td>
      <td><a href=".gitbook/assets/more-examples/nyc-ghost-neighborhoods.png">nyc-ghost-neighborhoods.png</a></td>
    </tr>
    <tr>
      <td><strong>9.49M Flickr photos</strong></td>
      <td>Reverse-geocode public photos and build country signatures from user-written tags.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/world-photo-index">world-photo-index.md</a></td>
      <td><a href=".gitbook/assets/more-examples/world-photo-index.png">world-photo-index.png</a></td>
    </tr>
    <tr>
      <td><strong>NOAA rain extremes</strong></td>
      <td>Stream every yearly GHCN-Daily CSV, keep top heaps, and reduce station-level extremes.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/ghcn-rainiest-day">ghcn-rainiest-day.md</a></td>
      <td><a href=".gitbook/assets/more-examples/ghcn-rainiest-day.png">ghcn-rainiest-day.png</a></td>
    </tr>
    <tr>
      <td><strong>One million GitHub READMEs</strong></td>
      <td>Export README Parquet from BigQuery, shard deterministic summarizers, and reduce category stats.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/github-repo-summarizer">github-repo-summarizer.md</a></td>
      <td><a href=".gitbook/assets/more-examples/github-repo-summarizer.png">github-repo-summarizer.png</a></td>
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
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>S3 to Postgres ETL</strong></td>
      <td>Transform 10,000 gzipped JSON files while protecting Postgres with <code>max_parallelism</code>.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/python-etl-no-airflow">python-etl-no-airflow.md</a></td>
      <td><a href=".gitbook/assets/more-examples/python-etl-no-airflow.png">python-etl-no-airflow.png</a></td>
    </tr>
    <tr>
      <td><strong>Millions of image resizes</strong></td>
      <td>Chunk image keys, resize with Pillow, and stream progress as workers write outputs.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/image-dataset-resize">image-dataset-resize.md</a></td>
      <td><a href=".gitbook/assets/more-examples/image-dataset-resize.png">image-dataset-resize.png</a></td>
    </tr>
    <tr>
      <td><strong>One Parquet file per worker</strong></td>
      <td>Compute per-file QA stats without starting Spark for a simple file-parallel job.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/parquet-parallel">parquet-parallel.md</a></td>
      <td><a href=".gitbook/assets/more-examples/parquet-parallel.png">parquet-parallel.png</a></td>
    </tr>
    <tr>
      <td><strong>Pandas apply in parallel</strong></td>
      <td>Partition a Parquet dataset and run ordinary pandas code on each worker.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/pandas-apply-parallel">pandas-apply-parallel.md</a></td>
      <td><a href=".gitbook/assets/more-examples/pandas-apply-parallel.png">pandas-apply-parallel.png</a></td>
    </tr>
    <tr>
      <td><strong>Enrich millions of users through a rate-limited API</strong></td>
      <td>Backfill user profiles while keeping provider limits explicit in chunk size, sleeps, and <code>max_parallelism</code>.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/rate-limited-api-requests">rate-limited-api-requests.md</a></td>
      <td><a href=".gitbook/assets/more-examples/rate-limited-api-requests.png">rate-limited-api-requests.png</a></td>
    </tr>
    <tr>
      <td><strong>Crawl a million website pages without hiding failures</strong></td>
      <td>Scrape static HTML with polite pacing, retries, error rows, and a global concurrency cap.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/parallel-web-scraping">parallel-web-scraping.md</a></td>
      <td><a href=".gitbook/assets/more-examples/parallel-web-scraping.png">parallel-web-scraping.png</a></td>
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
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Genome alignment</strong></td>
      <td>Run <code>bwa</code> and <code>samtools</code> in a custom image with one FASTQ pair per worker.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/bioinformatics-alignment">bioinformatics-alignment.md</a></td>
      <td><a href=".gitbook/assets/more-examples/bioinformatics-alignment.png">bioinformatics-alignment.png</a></td>
    </tr>
    <tr>
      <td><strong>GDAL raster processing</strong></td>
      <td>Compute NDVI one Sentinel tile at a time with <code>rasterio</code> and shared outputs.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/gdal-raster-processing">gdal-raster-processing.md</a></td>
      <td><a href=".gitbook/assets/more-examples/gdal-raster-processing.png">gdal-raster-processing.png</a></td>
    </tr>
    <tr>
      <td><strong>Billion-path Monte Carlo</strong></td>
      <td>Return sums and squared sums from independent chunks, then reduce locally.</td>
      <td><a href="https://docs.burla.dev/examples/demo-walkthroughs/monte-carlo-simulation">monte-carlo-simulation.md</a></td>
      <td><a href=".gitbook/assets/more-examples/monte-carlo-simulation.png">monte-carlo-simulation.png</a></td>
    </tr>
  </tbody>
</table>

