# ML & Search Examples

Examples for GPU embeddings, batch inference, vector search, and multimodal analysis.

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Embed 50K Wikipedia articles</strong></td><td>Embed 50,000 Wikipedia articles with a CUDA image, CPU download stage, GPU embedding stage, and shared vector artifacts.</td><td><a href="../demo-blogs/gpu-embedding-demo.md">gpu-embedding-demo.md</a></td><td><a href="../.gitbook/assets/more-examples/gpu-embedding-demo.png">gpu-embedding-demo.png</a></td></tr>
<tr><td><strong>Tune XGBoost on 1,000 CPUs</strong></td><td>Train 36 XGBoost models across 1,000 CPUs and pick the best flight-delay model.</td><td><a href="../examples/parallel-hyperparameter-tuning.md">parallel-hyperparameter-tuning.md</a></td><td><a href="../.gitbook/assets/more-examples/pandas-apply-parallel.png">pandas-apply-parallel.png</a></td></tr>
<tr><td><strong>Run batch LLM inference</strong></td><td>Load a Hugging Face model once per worker and score Parquet batches without building an endpoint.</td><td><a href="../demo-blogs/ml-inference-batch.md">ml-inference-batch.md</a></td><td><a href="../.gitbook/assets/more-examples/ml-inference-batch.png">ml-inference-batch.png</a></td></tr>
<tr><td><strong>Embed 2.7M arXiv papers</strong></td><td>Cluster 2.7M abstracts and find isolated papers by running the embedding job at corpus scale.</td><td><a href="../demo-blogs/arxiv-fossils.md">arxiv-fossils.md</a></td><td><a href="../.gitbook/assets/more-examples/arxiv-fossils.png">arxiv-fossils.png</a></td></tr>
<tr><td><strong>Search 192K artworks with CLIP</strong></td><td>Fetch and embed Open Access museum images, then use FAISS to find visual matches without labels.</td><td><a href="../demo-blogs/met-weirdest-art.md">met-weirdest-art.md</a></td><td><a href="../.gitbook/assets/more-examples/met-weirdest-art.png">met-weirdest-art.png</a></td></tr>
<tr><td><strong>Rank 50 million Airbnbs</strong></td><td>Run listings, photos, CLIP, YOLOv8, reviews, and bootstrap confidence intervals across the public corpus.</td><td><a href="../demo-blogs/airbnb-burla.md">airbnb-burla.md</a></td><td><a href="../.gitbook/assets/more-examples/airbnb-burla.png">airbnb-burla.png</a></td></tr>
</tbody>
</table>

## See Other Examples

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Basic Examples</strong></td><td>Starter patterns for files, giant inputs, and database range jobs.</td><td><a href="basic-examples.md">basic-examples.md</a></td><td><a href="../.gitbook/assets/more-examples/one-parquet-file-per-worker.png">one-parquet-file-per-worker.png</a></td></tr>
<tr><td><strong>Data Processing Examples</strong></td><td>Large-scale file, corpus, table scan, and Python data workloads.</td><td><a href="data-processing-examples.md">data-processing-examples.md</a></td><td><a href="../.gitbook/assets/more-examples/amazon-review-distiller.png">amazon-review-distiller.png</a></td></tr>
<tr><td><strong>Production Job Examples</strong></td><td>ETL, image processing, API backfills, web scraping, and simulations.</td><td><a href="production-data-jobs.md">production-data-jobs.md</a></td><td><a href="../.gitbook/assets/more-examples/python-etl-no-airflow.png">python-etl-no-airflow.png</a></td></tr>
<tr><td><strong>Science & Geo Examples</strong></td><td>Bioinformatics, raster processing, and scientific data scans.</td><td><a href="scientific-and-geospatial-work.md">scientific-and-geospatial-work.md</a></td><td><a href="../.gitbook/assets/more-examples/bioinformatics-alignment.png">bioinformatics-alignment.png</a></td></tr>
</tbody>
</table>
