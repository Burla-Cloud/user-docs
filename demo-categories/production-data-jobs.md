# Production Job Examples

Examples for file-parallel ETL, image processing, external service limits, and operational backfills.

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>ETL 10K S3 files to Postgres</strong></td><td>Transform 10,000 gzipped JSON files while protecting Postgres with <code>max_parallelism</code>.</td><td><a href="../demo-blogs/python-etl-no-airflow.md">python-etl-no-airflow.md</a></td><td><a href="../.gitbook/assets/more-examples/python-etl-no-airflow.png">python-etl-no-airflow.png</a></td></tr>
<tr><td><strong>Resize 1M images</strong></td><td>Chunk image keys, resize with Pillow, and stream progress as workers write outputs.</td><td><a href="../demo-blogs/image-dataset-resize.md">image-dataset-resize.md</a></td><td><a href="../.gitbook/assets/more-examples/image-dataset-resize.png">image-dataset-resize.png</a></td></tr>
<tr><td><strong>Run a 2M-user API backfill</strong></td><td>Backfill user profiles while keeping provider limits explicit in chunk size, sleeps, and <code>max_parallelism</code>.</td><td><a href="../demo-blogs/rate-limited-api-requests.md">rate-limited-api-requests.md</a></td><td><a href="../.gitbook/assets/more-examples/rate-limited-api-requests.png">rate-limited-api-requests.png</a></td></tr>
<tr><td><strong>Scrape 1M web pages</strong></td><td>Scrape static HTML with polite pacing, retries, error rows, and a global concurrency cap.</td><td><a href="../demo-blogs/parallel-web-scraping.md">parallel-web-scraping.md</a></td><td><a href="../.gitbook/assets/more-examples/parallel-web-scraping.png">parallel-web-scraping.png</a></td></tr>
<tr><td><strong>Run 1B option simulations</strong></td><td>Split the run into independent chunks, then reduce sums and squared sums locally.</td><td><a href="../demo-blogs/monte-carlo-simulation.md">monte-carlo-simulation.md</a></td><td><a href="../.gitbook/assets/more-examples/monte-carlo-simulation.png">monte-carlo-simulation.png</a></td></tr>
</tbody>
</table>

## See Other Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Basic Examples</strong></td><td>Starter patterns for files, giant inputs, and database range jobs.</td><td><a href="basic-examples.md">basic-examples.md</a></td><td><a href="../.gitbook/assets/more-examples/one-parquet-file-per-worker.png">one-parquet-file-per-worker.png</a></td></tr>
<tr><td><strong>ML & Search Examples</strong></td><td>GPU embeddings, batch inference, vector search, and multimodal analysis.</td><td><a href="ml-embeddings-and-search.md">ml-embeddings-and-search.md</a></td><td><a href="../.gitbook/assets/more-examples/gpu-embedding-demo.png">gpu-embedding-demo.png</a></td></tr>
<tr><td><strong>Data Processing Examples</strong></td><td>Large-scale file, corpus, table scan, and Python data workloads.</td><td><a href="data-processing-examples.md">data-processing-examples.md</a></td><td><a href="../.gitbook/assets/more-examples/amazon-review-distiller.png">amazon-review-distiller.png</a></td></tr>
<tr><td><strong>Science & Geo Examples</strong></td><td>Bioinformatics, raster processing, and scientific data scans.</td><td><a href="scientific-and-geospatial-work.md">scientific-and-geospatial-work.md</a></td><td><a href="../.gitbook/assets/more-examples/bioinformatics-alignment.png">bioinformatics-alignment.png</a></td></tr>
</tbody>
</table>
