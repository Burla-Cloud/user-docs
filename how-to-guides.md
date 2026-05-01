# How-to Guides

Practical guides for common Burla patterns and job controls.

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
      <td><strong>Read/Write Files to Cloud Storage</strong></td>
      <td>Use <code>/workspace/shared</code> to read and write files through Google Cloud Storage.</td>
      <td><a href="https://docs.burla.dev/how-to-guides/read-and-write-gcs-files">read-and-write-gcs-files.md</a></td>
      <td><a href=".gitbook/assets/more-examples/parquet-parallel.png">parquet-parallel.png</a></td>
    </tr>
    <tr>
      <td><strong>Use custom Docker images & GPUs</strong></td>
      <td>Run workers with custom images, native tools, CUDA dependencies, and GPU resources.</td>
      <td><a href="https://docs.burla.dev/how-to-guides/use-custom-docker-images-and-gpus">use-custom-docker-images-and-gpus.md</a></td>
      <td><a href=".gitbook/assets/more-examples/gpu-embedding-demo.png">gpu-embedding-demo.png</a></td>
    </tr>
    <tr>
      <td><strong>Run jobs in the background</strong></td>
      <td>Detach long jobs so they keep running if your laptop sleeps or disconnects.</td>
      <td><a href="https://docs.burla.dev/how-to-guides/run-python-in-the-background">run-python-in-the-background.md</a></td>
      <td><a href=".gitbook/assets/more-examples/nyc-taxi-history.png">nyc-taxi-history.png</a></td>
    </tr>
    <tr>
      <td><strong>Limit parallelism for APIs or databases</strong></td>
      <td>Keep Burla jobs inside external service limits with chunking and <code>max_parallelism</code>.</td>
      <td><a href="https://docs.burla.dev/how-to-guides/limit-parallelism-for-apis-databases-and-websites">limit-parallelism-for-apis-databases-and-websites.md</a></td>
      <td><a href=".gitbook/assets/more-examples/rate-limited-api-requests.png">rate-limited-api-requests.png</a></td>
    </tr>
    <tr>
      <td><strong>Combine many results/files into one</strong></td>
      <td>Use a simple map-reduce pattern to merge parallel outputs into one final result.</td>
      <td><a href="https://docs.burla.dev/how-to-guides/combine-many-results-files-into-one-map-reduce">combine-many-results-files-into-one-map-reduce.md</a></td>
      <td><a href=".gitbook/assets/more-examples/billion-path-monte-carlo.png">billion-path-monte-carlo.png</a></td>
    </tr>
    <tr>
      <td><strong>Decide how to split your work</strong></td>
      <td>Choose the right input unit before scaling a job across many workers.</td>
      <td><a href="https://docs.burla.dev/how-to-guides/choose-how-to-split-your-work">choose-how-to-split-your-work.md</a></td>
      <td><a href=".gitbook/assets/more-examples/one-parquet-file-per-worker.png">one-parquet-file-per-worker.png</a></td>
    </tr>
  </tbody>
</table>
