# Production Data Workflows

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
