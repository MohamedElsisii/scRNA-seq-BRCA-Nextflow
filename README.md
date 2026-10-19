# Sc-BRCA-Nextflow: A Breast Cancer Single-Cell RNA-seq Case Study

<img width="2172" height="724" alt="Untitled-1 copy" src="https://github.com/user-attachments/assets/3c934be1-3c60-4371-886a-567a7f49b4a4" />

A reproducible Nextflow workflow for processing 10x Genomics single-cell
RNA-sequencing data from human breast cancer and mammary-gland samples,
combining read-level quality control, Cell Ranger processing, and
exploratory analysis with Seurat and Scanpy.

## Case study at a glance

| Item | Summary |
|---|---|
| Biological samples processed | 6 |
| Paired-end FASTQ files | 48 |
| Data type | 10x Genomics single-cell RNA-seq |
| Read QC | FastQC, MultiQC |
| Quantification and matrix generation | Cell Ranger |
| Downstream exploratory analysis | Seurat, Scanpy |
| Workflow engine | Nextflow |
| Compute environment | Linux HPC / SLURM |
| Study accession | [GEO: GSE281490](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE281490) |

**Case-study outcome:** The workflow was run on six biological samples
and generated raw-read QC reports, Cell Ranger metrics and filtered
feature-barcode matrices, per-sample Seurat and Scanpy objects and
figures, and a final MultiQC report.

> **Results scope:** The current Seurat and Scanpy scripts perform
> exploratory analysis separately for each sample. They do not yet
> perform joint multi-sample integration, definitive cell-type
> annotation, or differential-expression analysis. The results
> highlighted here therefore focus on workflow execution, QC, and
> generated analysis outputs rather than unsupported biological
> discovery claims.

## Highlights

-   **End-to-end processing:** Organizes FASTQ validation, read QC, Cell
    Ranger processing, downstream analysis, and report generation into a
    Nextflow workflow.
-   **Multi-lane handling:** Groups lane-specific paired FASTQs by
    biological sample and passes them to Cell Ranger without manually
    concatenating lanes.
-   **Two analysis ecosystems:** Generates exploratory per-sample
    outputs with both Seurat (R) and Scanpy (Python).
-   **Reproducible environments:** Defines separate Conda environments
    for MultiQC, Seurat, and Scanpy.
-   **HPC execution:** Uses a SLURM profile with process-specific
    resource settings.

## Results showcase

### 1. Read-quality assessment

The raw-read QC review reported the expected 10x read layout:

-   **R1:** 28-bp barcode/UMI read
-   **R2:** approximately 91--92-bp transcript read

Per-base quality, per-sequence quality, N-content, sequence-length, and
adapter-content checks were broadly acceptable in the reviewed report.
High duplication in R2 was interpreted in the context of amplified
single-cell libraries, rather than treated as automatic evidence that
trimming or sample rejection was necessary.

<img width="1600" height="800" alt="fastqc_sequence_counts_plot" src="https://github.com/user-attachments/assets/83344fe8-4b8b-4488-8c9d-1f3399ad0ccc" />

### 2. Cell Ranger processing

Cell Ranger was run for each of the six samples to produce per-sample
metrics, web summaries, and filtered feature-barcode matrices.

<img width="929" height="369" alt="Screenshot 2026-10-19 145332" src="https://github.com/user-attachments/assets/9d878e54-0108-49c6-a1c2-6bff4377ec27" />

### 3. Exploratory Seurat and Scanpy Analysis

Seurat and Scanpy were used to perform exploratory single-cell RNA-seq
analysis independently for each biological sample. To keep this README
concise, BBM5 is presented as a representative example.

#### Seurat — BBM5

![Seurat exploratory analysis of BBM5](docs/figures/BBM5_seurat_summary.png)

*Figure 3. Exploratory analysis of BBM5 using Seurat. The composite
figure summarizes selected QC, dimensionality-reduction, clustering,
and visualization results.*

#### Scanpy — BBM5

![Scanpy exploratory analysis of BBM5](docs/figures/BBM5_scanpy_summary.png)

*Figure 4. Exploratory analysis of BBM5 using Scanpy, showing selected
results from the Python-based workflow.*

**Interpretation note:** BBM5 is shown as a representative sample;
the figures do not represent a joint analysis of all six samples.
Clusters with the same numeric label across separate analyses should
not automatically be interpreted as the same biological cell type.
PCA and UMAP are exploratory visualizations and do not independently
establish biological differences or causality.

### 4. Workflow outputs

The completed workflow generated:

-   FastQC output and a raw-read MultiQC report
-   Cell Ranger `metrics_summary.csv` and `web_summary.html` files for
    each sample
-   Filtered feature-barcode matrices for downstream analysis
-   Per-sample Seurat `.rds` objects and PDF figures
-   Per-sample Scanpy `.h5ad` objects and PDF figures
-   A final MultiQC report

Where possible, link to or display a small, reproducible summary rather
than uploading entire output directories to GitHub.

## Dataset

The case study uses data associated with:

-   **GEO:**
    [GSE281490](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE281490)
-   **BioProject:**
    [PRJNA1184144](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA1184144)
-   **SRA study:**
    [SRP544221](https://www.ncbi.nlm.nih.gov/sra/?term=SRP544221)

The six biological samples processed were `BBM0`, `BBM5`, `BSM1`,
`hMG1`, `hMG2`, and `hMG3`.

| Sample | SRA runs | Lane range |
|---|---|---|
| hMG3 | SRR31298425 | L001 |
| hMG2 | SRR31298426 | L001 |
| hMG1 | SRR31298427 | L001 |
| BBM5 | SRR31298428–SRR31298432 | L001–L005 |
| BSM1 | SRR31298433–SRR31298440 | L001–L008 |
| BBM0 | SRR31298441–SRR31298448 | L001–L008 |

The study was reported as private at the time of the original analysis.
Confirm current data availability and applicable data-use conditions
before redistributing data or metadata. Raw FASTQ files are not stored
in this repository.

## Workflow overview

``` text
Paired-end FASTQ files
          |
          v
Filename validation and lane pairing
          |
          v
FastQC per sample
          |
          v
Raw-read MultiQC report
          |
          v
Cell Ranger count per sample
          |
          +-------------------+
          |                   |
          v                   v
    Seurat analysis      Scanpy analysis
          |                   |
          +---------+---------+
                    |
                    v
             Final MultiQC report
```

## Repository structure

``` text
Sc-BRCA-Nextflow/
├── main.nf
├── nextflow.config
├── scripts/
│   └── submit_pipeline.sh
├── bin/
│   ├── seurat_script.R
│   └── scanpy_script.py
├── envs/
│   ├── multiqc_env.yml
│   ├── seurat_env.yml
│   └── scanpy_env.yml
├── docs/
│   └── figures/              # Selected, shareable README figures
├── references/               # Reference data; usually not tracked in Git
└── results/                  # Generated outputs; usually not tracked in Git
```

The reference directory, raw inputs, and generated results must be
supplied or created separately; they are not guaranteed to be included
in a GitHub clone.

## Software

| Tool | Role | Image |
|---|---|---|
| Nextflow | Workflow orchestration | <img width="613" height="123" alt="Nextflow_logo" src="https://github.com/user-attachments/assets/a0a10264-55ef-4fe3-b54e-9fef53bb431d" /> | 
| FastQC 0.11.5 | Read-level QC | <img width="1254" height="1254" alt="FastQC Interlocking Logo" src="https://github.com/user-attachments/assets/607d6c0c-ff4b-4801-afdc-d04b32833211" /> |
| MultiQC 1.35 | QC report aggregation | <img width="1318" height="250" alt="68747470733a2f2f6769746875622e636f6d2f4d756c746951432f4d756c746951432f7261772f6d61696e2f646f63732f696d616765732f4d756c746951435f6c6f676f2e706e67" src="https://github.com/user-attachments/assets/80d4e222-614f-41f0-ad32-fc2599d2b634" /> | 
| Cell Ranger 10.0.0 | 10x Genomics processing | <img width="402" height="402" alt="9890104" src="https://github.com/user-attachments/assets/9ff6f395-07ca-4ca5-8a5b-86c8cc46f590" /> |
| Seurat 5.3.0 | R-based exploratory analysis | <img width="4167" height="4164" alt="SeuratV5" src="https://github.com/user-attachments/assets/9efa68df-5718-4418-82cf-2d17ed14be54" /> | 
| Scanpy 1.11 | Python-based exploratory analysis | <img width="1755" height="774" alt="Scanpy_Logo_RGB" src="https://github.com/user-attachments/assets/a8aaa6d0-1c8b-4bc9-a03f-eafef2dc34b7" /> |


The repository contains Conda environment manifests in `envs/`. FastQC
and Cell Ranger are configured as HPC modules; module names and resource
settings may need adaptation for another cluster. Cell Ranger uses the
`refdata-gex-GRCh38-2024-A` reference, which must be installed
separately and accessible to the compute nodes.

## Running the workflow

The workflow expects pre-downloaded paired-end FASTQ files named using
this pattern:

``` text
SAMPLE_S1_L###_R1_001.fastq.gz
SAMPLE_S1_L###_R2_001.fastq.gz
```

Place them in `results/fastqs/`. The complete case-study dataset
contains 48 FASTQ files.

Check the input count:

``` bash
find results/fastqs -maxdepth 1 -type f -name '*.fastq.gz' | wc -l
```

Expected count for the complete dataset:

``` text
48
```

### Configure the environment

Configure Conda initialization, the Cell Ranger reference, and the
required HPC modules for your system. For example:

``` bash
export CONDA_SH=/path/to/conda.sh
export TRANSCRIPTOME=/path/to/refdata-gex-GRCh38-2024-A
```

Replace example paths with real locations accessible from the compute
nodes. Confirm that the Nextflow version, Java, FastQC, Cell Ranger, and
SLURM configuration are compatible with the target HPC.

### Run raw-read QC

From the repository root:

``` bash
sbatch scripts/submit_pipeline.sh
```

The submission script defaults to QC mode. Review:

``` text
results/multiqc/raw/multiqc_raw.html
results/fastqc/
```

### Run the complete workflow

After reviewing raw-read QC:

``` bash
PIPELINE_MODE=all sbatch scripts/submit_pipeline.sh
```

The workflow uses Nextflow resume behavior to reuse eligible completed
tasks. Actual HPC setup and resource requirements depend on the target
cluster.

## Output files

| Output | Location |
|---|---|
| FastQC reports | `results/fastqc/` |
| Raw-read MultiQC | `results/multiqc/raw/multiqc_raw.html` |
| Cell Ranger metrics | `results/cellranger/<sample>/metrics_summary.csv` |
| Cell Ranger web summary | `results/cellranger/<sample>/web_summary.html` |
| Filtered feature-barcode matrix | `results/cellranger/<sample>/filtered_feature_bc_matrix/` |
| Seurat PDF and object | `results/seurat/` |
| Scanpy PDF and object | `results/scanpy/` |
| Final MultiQC report | `results/multiqc/final/multiqc_final.html` |


This project demonstrates a modular Nextflow workflow for single-cell
RNA-seq processing, HPC execution, quality control, and exploratory
analysis using complementary R- and Python-based tools.
