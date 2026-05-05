# All examples

Real Burla workloads for ML, large-scale data processing, production jobs, and scientific computing.

### Basic Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Process 1000's of files</strong></td><td>Run one function call per file, write per-file outputs, and combine the results.</td><td><a href="demo-blogs/process-thousands-of-files-quickly.md">process-thousands-of-files-quickly.md</a></td><td><a href=".gitbook/assets/more-examples/one-parquet-file-per-worker.png">one-parquet-file-per-worker.png</a></td></tr>
<tr><td><strong>Process one giant file</strong></td><td>Split a large file into chunks, process chunks in parallel, and reduce the outputs.</td><td><a href="demo-blogs/process-one-giant-file-quickly.md">process-one-giant-file-quickly.md</a></td><td><a href=".gitbook/assets/more-examples/571m-amazon-reviews.png">571m-amazon-reviews.png</a></td></tr>
<tr><td><strong>Terabyte-Scale ETL</strong></td><td>Split database rows into ID ranges, process each range in parallel, and combine the results.</td><td><a href="demo-blogs/process-data-in-your-database-quickly.md">process-data-in-your-database-quickly.md</a></td><td><a href=".gitbook/assets/more-examples/s3-to-postgres-etl.png">s3-to-postgres-etl.png</a></td></tr>
</tbody>
</table>

### ML & Search Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Embed 50K Wikipedia articles</strong></td><td>Embed 50,000 Wikipedia articles with a CUDA image, CPU download stage, GPU embedding stage, and shared vector artifacts.</td><td><a href="demo-blogs/gpu-embedding-demo.md">gpu-embedding-demo.md</a></td><td><a href=".gitbook/assets/more-examples/gpu-embedding-demo.png">gpu-embedding-demo.png</a></td></tr>
<tr><td><strong>Tune XGBoost on 1,000 CPUs</strong></td><td>Train 36 XGBoost models across 1,000 CPUs and pick the best flight-delay model.</td><td><a href="examples/parallel-hyperparameter-tuning.md">parallel-hyperparameter-tuning.md</a></td><td><a href=".gitbook/assets/more-examples/pandas-apply-parallel.png">pandas-apply-parallel.png</a></td></tr>
<tr><td><strong>Run batch LLM inference</strong></td><td>Load a Hugging Face model once per worker and score Parquet batches without building an endpoint.</td><td><a href="demo-blogs/ml-inference-batch.md">ml-inference-batch.md</a></td><td><a href=".gitbook/assets/more-examples/ml-inference-batch.png">ml-inference-batch.png</a></td></tr>
<tr><td><strong>Embed 2.7M arXiv papers</strong></td><td>Cluster 2.7M abstracts and find isolated papers by running the embedding job at corpus scale.</td><td><a href="demo-blogs/arxiv-fossils.md">arxiv-fossils.md</a></td><td><a href=".gitbook/assets/more-examples/arxiv-fossils.png">arxiv-fossils.png</a></td></tr>
<tr><td><strong>Search 192K artworks with CLIP</strong></td><td>Fetch and embed Open Access museum images, then use FAISS to find visual matches without labels.</td><td><a href="demo-blogs/met-weirdest-art.md">met-weirdest-art.md</a></td><td><a href=".gitbook/assets/more-examples/met-weirdest-art.png">met-weirdest-art.png</a></td></tr>
<tr><td><strong>Rank 50 million Airbnbs</strong></td><td>Run listings, photos, CLIP, YOLOv8, reviews, and bootstrap confidence intervals across the public corpus.</td><td><a href="demo-blogs/airbnb-burla.md">airbnb-burla.md</a></td><td><a href=".gitbook/assets/more-examples/airbnb-burla.png">airbnb-burla.png</a></td></tr>
</tbody>
</table>

### Data Processing Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Query 2.4TB Parquet in 76s</strong></td><td>Run a DuckDB query over 1,000 Parquet files on 10,000 CPUs and combine the results.</td><td><a href="examples/process-2.4tb-of-parquet-files-in-76s.md">process-2.4tb-of-parquet-files-in-76s.md</a></td><td><a href=".gitbook/assets/more-examples/parquet-parallel.png">parquet-parallel.png</a></td></tr>
<tr><td><strong>Distill 571M Amazon reviews</strong></td><td>Read 275 GB of JSONL with HTTP Range requests, deterministic scoring, and heap-based reducers.</td><td><a href="demo-blogs/amazon-review-distiller.md">amazon-review-distiller.md</a></td><td><a href=".gitbook/assets/more-examples/amazon-review-distiller.png">amazon-review-distiller.png</a></td></tr>
<tr><td><strong>Scan 2.76B NYC taxi trips</strong></td><td>Scan 2.76B taxi and FHV trips to find ghost, emergent, and recovered city zones.</td><td><a href="demo-blogs/nyc-ghost-neighborhoods.md">nyc-ghost-neighborhoods.md</a></td><td><a href=".gitbook/assets/more-examples/nyc-ghost-neighborhoods.png">nyc-ghost-neighborhoods.png</a></td></tr>
<tr><td><strong>Geocode 9.49M Flickr photos</strong></td><td>Reverse-geocode public photos and build country signatures from user-written tags.</td><td><a href="demo-blogs/world-photo-index.md">world-photo-index.md</a></td><td><a href=".gitbook/assets/more-examples/world-photo-index.png">world-photo-index.png</a></td></tr>
<tr><td><strong>Summarize 1M GitHub READMEs</strong></td><td>Export README Parquet from BigQuery, shard deterministic summarizers, and reduce category stats.</td><td><a href="demo-blogs/github-repo-summarizer.md">github-repo-summarizer.md</a></td><td><a href=".gitbook/assets/more-examples/github-repo-summarizer.png">github-repo-summarizer.png</a></td></tr>
<tr><td><strong>Audit 5,000 Parquet files</strong></td><td>Compute per-file QA stats without starting Spark for a simple file-parallel job.</td><td><a href="demo-blogs/parquet-parallel.md">parquet-parallel.md</a></td><td><a href=".gitbook/assets/more-examples/parquet-parallel.png">parquet-parallel.png</a></td></tr>
<tr><td><strong>Parallelize pandas apply</strong></td><td>Partition a Parquet dataset and run ordinary pandas code on each worker.</td><td><a href="demo-blogs/pandas-apply-parallel.md">pandas-apply-parallel.md</a></td><td><a href=".gitbook/assets/more-examples/pandas-apply-parallel.png">pandas-apply-parallel.png</a></td></tr>
</tbody>
</table>

### Production Job Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>ETL 10K S3 files to Postgres</strong></td><td>Transform 10,000 gzipped JSON files while protecting Postgres with <code>max_parallelism</code>.</td><td><a href="demo-blogs/python-etl-no-airflow.md">python-etl-no-airflow.md</a></td><td><a href=".gitbook/assets/more-examples/python-etl-no-airflow.png">python-etl-no-airflow.png</a></td></tr>
<tr><td><strong>Resize 1M images</strong></td><td>Chunk image keys, resize with Pillow, and stream progress as workers write outputs.</td><td><a href="demo-blogs/image-dataset-resize.md">image-dataset-resize.md</a></td><td><a href=".gitbook/assets/more-examples/image-dataset-resize.png">image-dataset-resize.png</a></td></tr>
<tr><td><strong>Run a 2M-user API backfill</strong></td><td>Backfill user profiles while keeping provider limits explicit in chunk size, sleeps, and <code>max_parallelism</code>.</td><td><a href="demo-blogs/rate-limited-api-requests.md">rate-limited-api-requests.md</a></td><td><a href=".gitbook/assets/more-examples/rate-limited-api-requests.png">rate-limited-api-requests.png</a></td></tr>
<tr><td><strong>Scrape 1M web pages</strong></td><td>Scrape static HTML with polite pacing, retries, error rows, and a global concurrency cap.</td><td><a href="demo-blogs/parallel-web-scraping.md">parallel-web-scraping.md</a></td><td><a href=".gitbook/assets/more-examples/parallel-web-scraping.png">parallel-web-scraping.png</a></td></tr>
<tr><td><strong>Run 1B option simulations</strong></td><td>Split the run into independent chunks, then reduce sums and squared sums locally.</td><td><a href="demo-blogs/monte-carlo-simulation.md">monte-carlo-simulation.md</a></td><td><a href=".gitbook/assets/more-examples/monte-carlo-simulation.png">monte-carlo-simulation.png</a></td></tr>
</tbody>
</table>

### Science & Geo Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>1,000-CPU Genomics Pipeline</strong></td><td>Convert 360 sequencing samples with custom bioinformatics tools, then merge final files.</td><td><a href="examples/multi-stage-genomic-pipeline.md">multi-stage-genomic-pipeline.md</a></td><td><a href=".gitbook/assets/more-examples/bioinformatics-alignment.png">bioinformatics-alignment.png</a></td></tr>
<tr><td><strong>Align every FASTQ sample</strong></td><td>Run <code>bwa</code> and <code>samtools</code> in a custom image with one FASTQ pair per worker.</td><td><a href="demo-blogs/bioinformatics-alignment.md">bioinformatics-alignment.md</a></td><td><a href=".gitbook/assets/more-examples/bioinformatics-alignment.png">bioinformatics-alignment.png</a></td></tr>
<tr><td><strong>Find NOAA's rainiest day</strong></td><td>Stream every yearly GHCN-Daily CSV, keep top heaps, and reduce station-level extremes.</td><td><a href="demo-blogs/ghcn-rainiest-day.md">ghcn-rainiest-day.md</a></td><td><a href=".gitbook/assets/more-examples/ghcn-rainiest-day.png">ghcn-rainiest-day.png</a></td></tr>
<tr><td><strong>NDVI for 2K Sentinel tiles</strong></td><td>Compute NDVI one Sentinel tile at a time with <code>rasterio</code> and shared outputs.</td><td><a href="demo-blogs/gdal-raster-processing.md">gdal-raster-processing.md</a></td><td><a href=".gitbook/assets/more-examples/gdal-raster-processing.png">gdal-raster-processing.png</a></td></tr>
</tbody>
</table>
