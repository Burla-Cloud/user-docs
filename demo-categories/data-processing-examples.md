# Data Processing Examples

Large-scale data processing examples for files, corpora, table scans, and ordinary Python data code.

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Query 2.4TB Parquet in 76s</strong></td><td>Run a DuckDB query over 1,000 Parquet files on 10,000 CPUs and combine the results.</td><td><a href="../examples/process-2.4tb-of-parquet-files-in-76s.md">process-2.4tb-of-parquet-files-in-76s.md</a></td><td><a href="../.gitbook/assets/more-examples/parquet-parallel.png">parquet-parallel.png</a></td></tr>
<tr><td><strong>Distill 571M Amazon reviews</strong></td><td>Read 275 GB of JSONL with HTTP Range requests, deterministic scoring, and heap-based reducers.</td><td><a href="../demo-blogs/amazon-review-distiller.md">amazon-review-distiller.md</a></td><td><a href="../.gitbook/assets/more-examples/amazon-review-distiller.png">amazon-review-distiller.png</a></td></tr>
<tr><td><strong>Scan 2.76B NYC taxi trips</strong></td><td>Scan 2.76B taxi and FHV trips to find ghost, emergent, and recovered city zones.</td><td><a href="../demo-blogs/nyc-ghost-neighborhoods.md">nyc-ghost-neighborhoods.md</a></td><td><a href="../.gitbook/assets/more-examples/nyc-ghost-neighborhoods.png">nyc-ghost-neighborhoods.png</a></td></tr>
<tr><td><strong>Geocode 9.49M Flickr photos</strong></td><td>Reverse-geocode public photos and build country signatures from user-written tags.</td><td><a href="../demo-blogs/world-photo-index.md">world-photo-index.md</a></td><td><a href="../.gitbook/assets/more-examples/world-photo-index.png">world-photo-index.png</a></td></tr>
<tr><td><strong>Summarize 1M GitHub READMEs</strong></td><td>Export README Parquet from BigQuery, shard deterministic summarizers, and reduce category stats.</td><td><a href="../demo-blogs/github-repo-summarizer.md">github-repo-summarizer.md</a></td><td><a href="../.gitbook/assets/more-examples/github-repo-summarizer.png">github-repo-summarizer.png</a></td></tr>
<tr><td><strong>Audit 5,000 Parquet files</strong></td><td>Compute per-file QA stats without starting Spark for a simple file-parallel job.</td><td><a href="../demo-blogs/parquet-parallel.md">parquet-parallel.md</a></td><td><a href="../.gitbook/assets/more-examples/parquet-parallel.png">parquet-parallel.png</a></td></tr>
<tr><td><strong>Parallelize pandas apply</strong></td><td>Partition a Parquet dataset and run ordinary pandas code on each worker.</td><td><a href="../demo-blogs/pandas-apply-parallel.md">pandas-apply-parallel.md</a></td><td><a href="../.gitbook/assets/more-examples/pandas-apply-parallel.png">pandas-apply-parallel.png</a></td></tr>
</tbody>
</table>

## See Other Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Basic Examples</strong></td><td>Starter patterns for files, giant inputs, and database range jobs.</td><td><a href="basic-examples.md">basic-examples.md</a></td><td><a href="../.gitbook/assets/more-examples/one-parquet-file-per-worker.png">one-parquet-file-per-worker.png</a></td></tr>
<tr><td><strong>ML & Search Examples</strong></td><td>GPU embeddings, batch inference, vector search, and multimodal analysis.</td><td><a href="ml-embeddings-and-search.md">ml-embeddings-and-search.md</a></td><td><a href="../.gitbook/assets/more-examples/gpu-embedding-demo.png">gpu-embedding-demo.png</a></td></tr>
<tr><td><strong>Production Job Examples</strong></td><td>ETL, image processing, API backfills, web scraping, and simulations.</td><td><a href="production-data-jobs.md">production-data-jobs.md</a></td><td><a href="../.gitbook/assets/more-examples/python-etl-no-airflow.png">python-etl-no-airflow.png</a></td></tr>
<tr><td><strong>Science & Geo Examples</strong></td><td>Bioinformatics, raster processing, and scientific data scans.</td><td><a href="scientific-and-geospatial-work.md">scientific-and-geospatial-work.md</a></td><td><a href="../.gitbook/assets/more-examples/bioinformatics-alignment.png">bioinformatics-alignment.png</a></td></tr>
</tbody>
</table>
