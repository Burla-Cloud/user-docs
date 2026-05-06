# ML, Embeddings & Search

Examples for GPU embeddings, batch inference, vector search, and multimodal analysis.

<table data-view="cards">
<thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead>
<tbody>
<tr><td><strong>Embed 50K Wikipedia articles</strong></td><td>Embed 50,000 Wikipedia articles with a CUDA image, CPU download stage, GPU embedding stage, and shared vector artifacts.</td><td><a href="../demo-blogs/gpu-embedding-demo.md">gpu-embedding-demo.md</a></td><td><a href="../.gitbook/assets/more-examples/gpu-embedding-demo-card.png">gpu-embedding-demo-card.png</a></td></tr>
<tr><td><strong>Tune XGBoost on 1,000 CPUs</strong></td><td>Train 36 XGBoost models across 1,000 CPUs and pick the best flight-delay model.</td><td><a href="../examples/parallel-hyperparameter-tuning.md">parallel-hyperparameter-tuning.md</a></td><td><a href="../.gitbook/assets/more-examples/parallel-hyperparameter-tuning-card.png">parallel-hyperparameter-tuning-card.png</a></td></tr>
<tr><td><strong>Run batch LLM inference</strong></td><td>Load a Hugging Face model once per worker and score Parquet batches without building an endpoint.</td><td><a href="../demo-blogs/ml-inference-batch.md">ml-inference-batch.md</a></td><td><a href="../.gitbook/assets/more-examples/ml-inference-batch-card.png">ml-inference-batch-card.png</a></td></tr>
<tr><td><strong>Embed 2.7M arXiv papers</strong></td><td>Cluster 2.7M abstracts and find isolated papers by running the embedding job at corpus scale.</td><td><a href="../demo-blogs/arxiv-fossils.md">arxiv-fossils.md</a></td><td><a href="../.gitbook/assets/more-examples/arxiv-fossils-card.png">arxiv-fossils-card.png</a></td></tr>
<tr><td><strong>Search 192K artworks with CLIP</strong></td><td>Fetch and embed Open Access museum images, then use FAISS to find visual matches without labels.</td><td><a href="../demo-blogs/met-weirdest-art.md">met-weirdest-art.md</a></td><td><a href="../.gitbook/assets/more-examples/met-weirdest-art-card.png">met-weirdest-art-card.png</a></td></tr>
<tr><td><strong>Analyze every public Airbnb</strong></td><td>Run listings, photos, CLIP, Haiku Vision, reviews, and bootstrap confidence intervals across the public corpus.</td><td><a href="../demo-blogs/airbnb-burla.md">airbnb-burla.md</a></td><td><a href="../.gitbook/assets/more-examples/airbnb-burla-card.png">airbnb-burla-card.png</a></td></tr>
</tbody>
</table>
