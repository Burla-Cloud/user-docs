---
description: Using 1,300 CPUs and <100 lines of code.
---

# Multi-Stage Genomic Pipeline

In this example we:

* Download raw Illumina genomic-sequencing data from [this NCBI experiment](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE165845).
* Call and align each sample with a human reference genome.
* Combine samples into a single large .BED file, then convert to PGEN/PVAR/PSAM files.

This is a typical workflow to prepare Illumina sequencing data for downstream analysis.

### Step 1: Boot some VMs

In the "Settings" tab we select the hardware, container image, and quantity of VM's we want.\
Then hit **⏻ Start** on the homepage!

<figure><img src="../.gitbook/assets/boot.gif" alt=""><figcaption></figcaption></figure>

Here we boot 13, 80-CPU VM's, these VM's delete themself after 15min of inactivity.\
We also specify a custom docker image: `jakezuliani/idats_to_pgen:latest`\
This image has bcftools, PLINK, and PLINK2 installed, this is the image our code will run inside.

Once our machines are have booted, we can call `remote_parallel_map` !

### Step 2: download prerequisite data

This code downloads the reference genome, and BPM / EGT files then saves it all to `./shared`.\
This directory is network linked to a Google Cloud Storage bucket using GCSFuse.

<details>

<summary>Imports and URL definitions</summary>

```python
import io
import gzip
import zipfile
import requests
import subprocess
from shutil import move
from pathlib import Path
import pandas as pd
from burla import remote_parallel_map

def _run_cmd(cmd: str, print_output=False):
    kwargs = {} if print_output else dict(stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    process = subprocess.run(cmd, shell=True, text=True, **kwargs)
    if process.returncode != 0:
        print(process.stdout)
        print(process.stderr)
        raise subprocess.CalledProcessError(process.returncode, cmd)

# urls
SAMPLE_INFO_URL = "https://storage.googleapis.com/burla-demo-data/genomic_sample_info.csv"
REF_GZ_URL = "ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/human_g1k_v37.fasta.gz"
REF_FAI_URL = "ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/human_g1k_v37.fasta.fai"
illumina_base_url = "https://webdata.illumina.com/downloads/productfiles/global-screening-array/v1-0"
BPM_URL = f"{illumina_base_url}/infinium-global-screening-array-v1-0-c1-manifest-file-bpm-build37.zip"
CLUSTER_URL = f"{illumina_base_url}/infinium-global-screening-array-v1-0-c1-cluster-file.zip"
# filepaths
REF_PATH = Path("shared/idat_to_pgen_pipeline/human_g1k_v37.fasta")
REF_GZ_PATH = REF_PATH.with_suffix(".fasta.gz")
REF_FAI_PATH = REF_PATH.with_suffix(".fasta.fai")
BPM_PATH = Path("shared/idat_to_pgen_pipeline/GSA-24v1-0_C1.bpm")
EGT_PATH = Path("shared/idat_to_pgen_pipeline/GSA-24v1-0_C1_ClusterFile.egt")
```

</details>

```python
def download_prerequisite_data(_):
    Path("shared/idat_to_pgen_pipeline").mkdir(parents=True, exist_ok=True)
    
    # download bpm / cluster files:
    if not BPM_PATH.exists():
        bpm_zip = zipfile.ZipFile(io.BytesIO(requests.get(BPM_URL).content))
        cluster_zip = zipfile.ZipFile(io.BytesIO(requests.get(CLUSTER_URL).content))
        bpm_zip.extractall("shared/idat_to_pgen_pipeline")
        cluster_zip.extractall("shared/idat_to_pgen_pipeline")

    # download / unzip reference genome:
    if not REF_PATH.with_suffix(".fasta.gz").exists():
        _run_cmd(f"wget --passive-ftp -O {REF_GZ_PATH} {REF_GZ_URL}", print_output=True)
        # uses old RAZF format, this makes it unzippable:
        _run_cmd(f"truncate -s 891946027 {REF_GZ_PATH}", print_output=True)
        _run_cmd(f"gunzip -f {REF_GZ_PATH}", print_output=True)

    # index reference genome for bcftools
    if not REF_FAI_PATH.exists():
        _run_cmd(f"samtools faidx {REF_PATH}", print_output=True)

    # return list of (sample_id, cell_line, replicate) tuples:
    df = pd.read_csv(SAMPLE_INFO_URL)
    return list(zip(df.sample_id, df.cell_line, df.replicate))

sample_info = remote_parallel_map(download_prerequisite_data, [None])[0]
```

After downloading this data, it appears in the Filesystem tab in the dashboard: (GCS)

<figure><img src="../.gitbook/assets/CleanShot 2026-01-11 at 12.40.39.png" alt=""><figcaption></figcaption></figure>

### Step 3: Download IDAT files for all 360 samples in parallel

This code uses 720 parallel 1-CPU containers to download the red & green IDAT file for each sample.

<pre class="language-python"><code class="lang-python">def download_single_idat_file(_id, cell_line, replicate, color):
    idat_path = Path(f"shared/idat_to_pgen_pipeline/{_id}/{cell_line}_{replicate}_{color}.idat")
    idat_path.parent.mkdir(parents=True, exist_ok=True)
<strong>    if not idat_path.exists():
</strong>        url = f"https://www.ncbi.nlm.nih.gov/geo/download/?acc={_id}"
        url += f"&#x26;format=file&#x26;file={_id}%5F{cell_line}%5F{replicate}%5F{color}%2Eidat%2Egz"
        with gzip.open(io.BytesIO(requests.get(url).content)) as f_in, idat_path.open("wb") as f_out:
            f_out.write(f_in.read())

sample_info_colored = [(*a, color) for a in sample_info for color in ("Grn", "Red")]
remote_parallel_map(download_single_idat_file, sample_info_colored)
</code></pre>

Folders with Red/Green IDAT's for each sample are now visible in the Filesystem tab:

<figure><img src="../.gitbook/assets/result2 (1).gif" alt=""><figcaption></figcaption></figure>

### Step 4: Call and align all samples in parallel

This code uses 360 parallel containers each with 8 CPUs and 32G of RAM.\
For each pair of IDAT files this code:

* Performs base calling and genotype clustering with `bcftools +idat2gtc`&#x20;
* Aligns to a reference genome with `bcftools +gtc2vcf`&#x20;
* Filters to retain only biallelic variants with `bcftools view -m2 -M2`&#x20;
* Converts the VCF into PLINK BED, BIM, and FAM files using `plink`

```python
def convert_idats_to_plink_bed_format(_id, cell_line, replicate):
    gtc_path = Path(f"shared/idat_to_pgen_pipeline/{_id}/{cell_line}_{replicate}.gtc")
    vcf_path = gtc_path.with_suffix(".vcf")

    # Convert green & red IDAT to single GTC
    _run_cmd(f"bcftools +idat2gtc --bpm {BPM_PATH} --egt {EGT_PATH} --idats shared/idat_to_pgen_pipeline/{_id}", print_output=True)
    move(f"{cell_line}_{replicate}.gtc", gtc_path)

    # Convert GTC to VCF
    _run_cmd(f"bcftools +gtc2vcf {gtc_path} -b {BPM_PATH} -e {EGT_PATH} -f {REF_PATH} -o {vcf_path} --do-not-check-bpm", print_output=True)

    # Filter VCF, only keep biallelic sites
    filtered_vcf_path = vcf_path.with_suffix(".temp_filtered.vcf")
    _run_cmd(f"bcftools view -m2 -M2 -o {filtered_vcf_path} {vcf_path}", print_output=True)
    filtered_vcf_path.rename(vcf_path)

    # Convert VCF to BED
    _run_cmd(f"plink --vcf {vcf_path} --out {vcf_path.with_suffix('')} --const-fid 0, --make-bed --allow-extra-chr --keep-allele-order", print_output=True)

remote_parallel_map(convert_idats_to_plink_bed_format, sample_info, func_cpu=8)
```

Each sample's folder now contains output from the above commands:

<figure><img src="../.gitbook/assets/CleanShot 2026-01-11 at 13.00.49.png" alt=""><figcaption></figcaption></figure>

### Step 5: Merge samples into a single PGEN/PVAR/PSAM file.

This code uses a single container with 80 CPUs and 320G of RAM.

```python
MERGED_PATH = "shared/idat_to_pgen_pipeline/merged"

def combine_bed_files_into_pgen_file(sample_info):
    # Merge bed files into one big bed file
    mergelist = [f"shared/idat_to_pgen_pipeline/{_id}/{cl}_{r}" for _id, cl, r in sample_info]
    mergelist_path = Path("merge_list.txt")
    mergelist_path.write_text("\n".join(mergelist))
    cmd = f"plink --bfile {mergelist[0]} --merge-list {mergelist_path} --out {MERGED_PATH} --allow-extra-chr --biallelic-only strict"
    _run_cmd(cmd, print_output=True)
    # Convert merged BED to PGEN using plink2
    _run_cmd(f"plink2 --bfile {MERGED_PATH} --out {MERGED_PATH} --make-pgen --allow-extra-chr", print_output=True)

_ = remote_parallel_map(combine_bed_files_into_pgen_file, [sample_info], func_cpu=80, func_ram=320)
```

After running the PGEN/PVAR/PSAM files are available for download in the Filesystem tab! (GCS)

<figure><img src="../.gitbook/assets/CleanShot 2026-01-11 at 13.09.29.png" alt=""><figcaption></figcaption></figure>

### Want to modify or run this code?

This demo is available as a Google Colab notebook here:\
[https://colab.research.google.com/drive/1lEbeGOoowZ9FKA9yctziWyhH6TvLuxTi?usp=sharing](https://colab.research.google.com/drive/1lEbeGOoowZ9FKA9yctziWyhH6TvLuxTi?usp=sharing)

The notebook contains instructions to get Burla up and running as well as run the demo.\
Don't hesitate to email me (jake@burla.dev) if you get stuck! Thank you for trying Burla!

&#x20;

&#x20;

&#x20;
