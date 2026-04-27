---
description: Split a Burla workflow into stages with different resources.
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: false
  tags:
    visible: true
---

# Run pipeline stages on different hardware

Use this when different parts of a job need different CPUs, RAM, GPUs, or Docker images.
Do not use this for a single homogeneous map job.
The unit of work can change by stage: files, samples, image batches, document shards, or candidate lists.
Each worker writes a durable artifact that the next stage can read.
The output is a chain of artifacts, plus a final report, index, table, or manifest.

Real pipelines are not always one map call. Some stages inspect, some transform, some score, and some reduce.

## The pattern

Split the workflow by artifact boundaries:

1. cheap inspection
2. broad CPU work
3. custom-image work for native tools
4. GPU work for model stages
5. high-memory reduce

Each stage should read stable inputs and write stable outputs. Pass paths between stages instead of large Python objects.

## Plan the artifacts first

Start with the files each stage owns.

```python
SHARED_ROOT = "/workspace/shared/doc-pipeline"

raw_files = [
    f"{SHARED_ROOT}/raw/customer-a.pdf",
    f"{SHARED_ROOT}/raw/customer-b.pdf",
    f"{SHARED_ROOT}/raw/customer-c.pdf",
]
```

A useful plan names the contract for every stage:

1. raw PDF in, profile JSON out
2. normal PDF path in, text JSONL out
3. scanned PDF path in, OCR text JSONL out
4. text shard path in, embedding shard out
5. embedding paths in, index path out

The artifact names are not ceremony. They are what make reruns cheap.

## Stage 1: inspect on cheap CPU workers

The first stage should find the shape of the data without spending expensive hardware.

```python
def inspect_pdf(path):
    from pathlib import Path
    import json

    profile = {
        "path": path,
        "bytes": Path(path).stat().st_size,
        "needs_ocr": path.endswith("-scan.pdf"),
    }
    output_path = Path("/workspace/shared/doc-pipeline/profiles") / f"{Path(path).stem}.json"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(json.dumps(profile))
    return str(output_path)
```

```python
from burla import remote_parallel_map

profile_paths = remote_parallel_map(
    inspect_pdf,
    raw_files,
    func_cpu=1,
    func_ram=2,
    grow=True,
)
```

This stage returns paths, not parsed documents.

## Stage 2: route work from the profiles

Use the profile artifacts to build the next input lists.

```python
import json
from pathlib import Path

profiles = [json.loads(Path(path).read_text()) for path in profile_paths]

normal_pdfs = [profile["path"] for profile in profiles if not profile["needs_ocr"]]
scanned_pdfs = [profile["path"] for profile in profiles if profile["needs_ocr"]]
```

The branching logic stays in Python. You do not need to turn it into a scheduler.

## Stage 3: parse normal PDFs on CPU

```python
def parse_pdf(path):
    from pathlib import Path
    import json
    from pypdf import PdfReader

    reader = PdfReader(path)
    text = "\n".join(page.extract_text() or "" for page in reader.pages)
    output_path = Path("/workspace/shared/doc-pipeline/text") / f"{Path(path).stem}.jsonl"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(json.dumps({"source": path, "text": text[:4000]}) + "\n")
    return str(output_path)
```

```python
text_paths = remote_parallel_map(
    parse_pdf,
    normal_pdfs,
    image="python:3.12",
    func_cpu=4,
    func_ram=8,
    grow=True,
)
```

## Stage 4: OCR scans in a custom image

The OCR stage needs different dependencies. Use a different image only for this call.

```python
def ocr_pdf(path):
    from pathlib import Path
    import json
    import subprocess

    sidecar_path = "/tmp/ocr.txt"
    subprocess.run(["ocrmypdf", "--sidecar", sidecar_path, path, "/tmp/out.pdf"], check=True)
    output_text = Path(sidecar_path).read_text()
    output_path = Path("/workspace/shared/doc-pipeline/ocr-text") / f"{Path(path).stem}.jsonl"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(json.dumps({"source": path, "text": output_text}) + "\n")
    return str(output_path)
```

```python
ocr_text_paths = remote_parallel_map(
    ocr_pdf,
    scanned_pdfs,
    image="us-docker.pkg.dev/my-project/burla/ocr-worker:latest",
    func_cpu=8,
    func_ram=16,
    grow=True,
)
```

Native dependencies belong in the image. The input contract stays a path.

## Stage 5: embed text on GPUs

Join the text artifacts, then pass those paths to the GPU stage.

```python
all_text_paths = text_paths + ocr_text_paths
```

```python
def embed_text_file(path):
    import json
    from pathlib import Path

    import numpy as np
    from sentence_transformers import SentenceTransformer

    if not hasattr(embed_text_file, "_model"):
        embed_text_file._model = SentenceTransformer("BAAI/bge-large-en-v1.5", device="cuda")

    rows = [json.loads(line) for line in Path(path).read_text().splitlines()]
    vectors = embed_text_file._model.encode([row["text"] for row in rows], normalize_embeddings=True).astype("float32")

    output_path = Path("/workspace/shared/doc-pipeline/embeddings") / f"{Path(path).stem}.npy"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    np.save(output_path, vectors)
    return str(output_path)
```

```python
embedding_paths = remote_parallel_map(
    embed_text_file,
    all_text_paths,
    image="us-docker.pkg.dev/my-project/burla/embedder:latest",
    func_gpu="A100",
    func_cpu=4,
    func_ram=32,
    max_parallelism=8,
    grow=True,
)
```

The GPU stage has its own quota, image, and batch behavior. It should not force the OCR or parse stages onto GPUs.

## Stage 6: reduce on a larger CPU worker

When a stage needs a global view, run a reducer with the resources that reducer needs.

```python
def build_manifest(paths):
    from pathlib import Path
    import json

    output_path = Path("/workspace/shared/doc-pipeline/final/embedding-manifest.json")
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(json.dumps({"embedding_paths": paths}, indent=2))
    return str(output_path)
```

```python
[manifest_path] = remote_parallel_map(
    build_manifest,
    [embedding_paths],
    func_cpu=8,
    func_ram=32,
    grow=True,
)
```

The reducer gets one input: the list of paths from the previous stage.

## Why stage boundaries matter

Stage boundaries make failures cheaper.

If OCR fails, you do not rerun inspection.

If the embedding image has the wrong CUDA wheel, you do not redownload or reparse documents.

If the final index code changes, you can reuse the embedding shards.

That is the main reason to write durable intermediate files.

## Choose resources per stage

Use the resource shape that matches one worker in that stage:

1. inspection: low CPU, low RAM
2. parsing: CPU and RAM for one file
3. OCR or native tools: custom image, more CPU
4. embeddings or vision models: CUDA image and `func_gpu`
5. reduce: higher RAM if it loads many artifacts
6. external writes: `max_parallelism` from the sink

The pipeline should spend expensive hardware only where the data needs it.

## When not to use this

Do not split a job into stages just to look organized.

If every input runs the same function with the same resources and returns a small result, use one `remote_parallel_map` call.

If the only final step is summing small values, use the basic map-reduce pattern.

## Examples that use this pattern

- [Genomic Pipeline on 1,000 CPUs](../examples/multi-stage-genomic-pipeline.md)
- [Test Airbnb hypotheses at public-data scale](../demo-blogs/airbnb-burla.md)
- [Put the embedding model on A100s, then ask the search question](../demo-blogs/gpu-embedding-demo.md)
- [Process every raster tile, not a pretty subset](../demo-blogs/gdal-raster-processing.md)
- [Align every FASTQ sample without building a scheduler first](../demo-blogs/bioinformatics-alignment.md)
- [Read/Write Files to Cloud Storage](../common-patterns/read-and-write-gcs-files.md)
