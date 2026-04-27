# Align every FASTQ sample without building a scheduler first

The compromised genomics run aligns one public sample, writes down the command, and postpones the cohort. The real experiment starts every sample in the cohort and produces BAMs with the same reference, read group, and container. If the cohort never ran, you tested the command rather than the analysis.

This demo runs BWA-MEM and samtools on thousands of paired-end FASTQ samples, one sample per Burla worker.

## what we built

Bioinformatics tools need native binaries, so this demo uses a custom worker image.

```python
IMAGE = "us-docker.pkg.dev/test-burla/burla-demos/burla-bio-worker:latest"
S3_OUT = "s3://my-bam-bucket"

with open("manifest.tsv") as f:
    samples = [line.strip().split("\t") for line in f if line.strip()]

sample_jobs = [{"sample_id": s[0], "fq1": s[1], "fq2": s[2]} for s in samples]
```

The worker downloads the reference indexes and FASTQs, then shells out to the tools everyone already knows.

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

## how the pipeline works

Burla runs the container image and passes each sample dict to the worker.

```python
from burla import remote_parallel_map

reports = remote_parallel_map(
    align_sample, sample_jobs,
    func_cpu=4, func_ram=16,
    image=IMAGE,
    grow=True,
)
```

## why this demo is interesting

Alignment is a good demo because the hard part is not inventing a new algorithm. The command is known. The pain is getting the same command, the same reference, and the same binaries onto enough machines at once, then collecting BAMs without losing failures. That is infrastructure work pretending to be science.

The real experiment is cohort-level. Mapping rate distributions, missing pairs, corrupt FASTQs, and sample-specific runtime only appear when every sample runs. A compromised smoke test is still worth doing, but it should stay a smoke test. Once the command works, the useful next step is running the cohort and inspecting the report.

## how to build your version

Build the smallest image that has the binaries: BWA, samtools, minimap2, STAR, DeepVariant, or GATK. Put reference files in a bucket or bake them into the image if they are stable. Make one input equal one sample, and return BAM size, elapsed time, mapping rate, and output paths.

## why Burla fits

Burla removes Nextflow setup, Batch compute environments, job queues, and AMI scripts. The custom image gives workers the tools, while `remote_parallel_map` gives you the sample queue.

## what the single sample misses

One aligned sample proves the command. The real cohort run finds the sample with bad pairs, the reference mismatch, and the outlier mapping rate. That is the discovery the compromised run skips.
