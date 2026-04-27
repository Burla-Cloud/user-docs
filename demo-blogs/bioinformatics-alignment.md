# Align every FASTQ sample without building a scheduler first

In this example we:

* Read a paired-end FASTQ manifest.
* Run BWA-MEM and samtools in a custom worker image.
* Produce one BAM per sample, with one sample per Burla worker.

One aligned sample proves the command works. It does not prove the cohort ran.

### Step 1: Use an image with the native tools

Bioinformatics tools need native binaries, so the worker image matters.

```python
IMAGE = "us-docker.pkg.dev/test-burla/burla-demos/burla-bio-worker:latest"
S3_OUT = "s3://my-bam-bucket"

with open("manifest.tsv") as f:
    samples = [line.strip().split("\t") for line in f if line.strip()]

sample_jobs = [{"sample_id": s[0], "fq1": s[1], "fq2": s[2]} for s in samples]
```

### Step 2: Align one sample per worker

The worker downloads the FASTQs, runs the command-line tools, indexes the BAM, and writes the output to S3.

```python
def align_sample(job: dict) -> dict:
    import os, subprocess, time

    sid, fq1, fq2 = job["sample_id"], job["fq1"], job["fq2"]
    work = f"/tmp/{sid}"
    os.makedirs(work, exist_ok=True)

    def run(cmd: str):
        subprocess.run(cmd, shell=True, check=True, executable="/bin/bash")

    t0 = time.time()
    run(f"aws s3 cp {fq1} {work}/R1.fastq.gz")
    run(f"aws s3 cp {fq2} {work}/R2.fastq.gz")
    run(f"bwa mem -t 4 {work}/ref.fa {work}/R1.fastq.gz {work}/R2.fastq.gz | samtools sort -@ 4 -o {work}/{sid}.bam -")
    run(f"samtools index {work}/{sid}.bam")
    run(f"aws s3 cp {work}/{sid}.bam {S3_OUT}/bams/{sid}.bam")
    return {"sample_id": sid, "elapsed_s": round(time.time() - t0, 1)}
```

### Step 3: Run the cohort

Each sample gets 4 CPUs and 16GB of RAM.

```python
from burla import remote_parallel_map

reports = remote_parallel_map(
    align_sample,
    sample_jobs,
    func_cpu=4,
    func_ram=16,
    image=IMAGE,
    grow=True,
)
```

### What's the point?

The command is known. The pain is getting the same command, reference, binaries, and output path onto enough machines at once.

This is why I like one-sample-per-worker. The report gives sample-specific runtime and failures, and the output is already in S3. Once the smoke test works, run the cohort. That is where bad pairs, corrupt FASTQs, and mapping-rate outliers show up.
