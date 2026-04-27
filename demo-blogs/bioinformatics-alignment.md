# Parallel genome alignment without a scheduler

Bioinformatics pipelines often start as shell commands and end up wrapped in a scheduler because each sample is independent but expensive. This demo keeps the shell commands visible. Each Burla worker copies FASTQ pairs and reference files from S3, runs `bwa mem` and `samtools sort`, indexes the BAM, uploads BAM plus BAI, and returns a small report row.

The worker image is explicit: `us-docker.pkg.dev/test-burla/burla-demos/burla-bio-worker:latest`. That image carries the bioinformatics binaries. Burla supplies the fan-out and resource shape.

## What The Job Does

The compromised version aligns one sample locally or submits a few samples to a test cluster. That verifies the reference index and read group string. It does not show what happens when hundreds or thousands of samples need the same reference assets and isolated scratch space.

The real version reads `manifest.tsv`, where each line contains a sample ID and two FASTQ S3 paths. It builds one job dict per sample, sends those dicts to Burla, and gives each worker 4 CPUs and 16 GB RAM. The worker creates `/tmp/{sample_id}`, downloads the GRCh38 FASTA and BWA index files, copies both reads, runs alignment and sorting with four threads, writes the output BAM and index to `s3://my-bam-bucket/bams/`, and returns elapsed seconds and BAM size.

This is a direct map job. There is no reduce phase because the meaningful output is one BAM per sample.

## The Code Shape

The worker wraps shell commands, but the orchestration is Python.

```python
IMAGE = "us-docker.pkg.dev/test-burla/burla-demos/burla-bio-worker:latest"

def align_sample(job: dict) -> dict:
    import os
    import subprocess
    import time

    sid, fq1, fq2 = job["sample_id"], job["fq1"], job["fq2"]
    work = f"/tmp/{sid}"
    os.makedirs(work, exist_ok=True)

    def run(cmd: str):
        subprocess.run(cmd, shell=True, check=True, executable="/bin/bash")

    run(f"aws s3 cp s3://my-refs/GRCh38.fa {work}/ref.fa")
    run(f"aws s3 cp {fq1} {work}/R1.fastq.gz")
    run(f"aws s3 cp {fq2} {work}/R2.fastq.gz")
    run(
        f"bwa mem -t 4 -R '@RG\\tID:{sid}\\tSM:{sid}\\tLB:{sid}\\tPL:ILLUMINA' "
        f"{work}/ref.fa {work}/R1.fastq.gz {work}/R2.fastq.gz "
        f"| samtools sort -@ 4 -o {work}/{sid}.bam -"
    )
    run(f"samtools index {work}/{sid}.bam")
    run(f"aws s3 cp {work}/{sid}.bam {S3_OUT}/bams/{sid}.bam")
    return {"sample_id": sid, "bam_bytes": os.path.getsize(f"{work}/{sid}.bam")}
```

The map call selects the container image:

```python
reports = remote_parallel_map(
    align_sample,
    sample_jobs,
    func_cpu=4,
    func_ram=16,
    image=IMAGE,
    grow=True,
)
```

## Why It Matters

This is the kind of workload where "distributed compute" should not force a rewrite. The domain tool is still `bwa`. The storage is still S3. The sample manifest is still a TSV.

Burla matters because it lets the user treat each sample as a Python input and attach a Docker image with the required binaries. The result is a normal `alignment_report.csv` plus BAM artifacts in S3, not a new pipeline language.
