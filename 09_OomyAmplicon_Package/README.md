# OomyAmplicon v0.1.0: ASV and taxonomy package

OomyAmplicon source code is licensed under the MIT License (see `LICENSE`). This license applies to the software code; reference databases are not bundled and retain their own license terms.

OomyAmplicon turns Illumina FASTQ files into a DADA2 ASV count table, representative sequences, and taxonomy annotations. It uses a user-supplied QIIME 2 classifier or trains a classifier from a user-supplied reference FASTA and taxonomy TSV. It follows the modular amplicon processing approach described for EasyAmplicon, while using QIIME 2/DADA2 for denoising and QIIME feature-classifier for taxonomy.

## Installation (Linux x86_64)

The commands below install Miniconda if needed, create the tested QIIME 2 Amplicon 2025.4 environment if needed, and install OomyAmplicon. QIIME 2's official 2025.4 environment file is used so the required DADA2 and feature-classifier plugins are included. Other QIIME 2 versions may also be used if they provide these plugins and a Python version supported by this package (Python 3.10 or 3.11). Install OomyAmplicon into the same environment from which QIIME 2 is run.

When changing QIIME 2 versions, use a classifier trained in that target environment to avoid QIIME artifact compatibility issues. OomyAmplicon can train it from the matching reference FASTA and taxonomy TSV using `--reference-fasta` and `--reference-taxonomy`; alternatively, provide a `.qza` classifier trained with that QIIME 2 version.

### 1. Install Conda if it is not already installed

These commands are for Linux x86_64. If Conda is already available, skip this step.

```bash
curl -L -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
~/miniconda3/bin/conda init bash
```

Follow the installer prompts, then close and reopen the terminal. Check the installation:

```bash
conda --version
```

### 2. Install QIIME 2 if it is not already installed

If an environment with QIIME 2 2025.4, `q2-dada2`, and `q2-feature-classifier` is already available, skip environment creation and activate that environment instead.

```bash
conda env create \
  --name qiime2-amplicon-2025.4 \
  --file https://raw.githubusercontent.com/qiime2/distributions/refs/heads/dev/2025.4/amplicon/released/qiime2-amplicon-ubuntu-latest-conda.yml
conda activate qiime2-amplicon-2025.4
qiime info
```

QIIME 2 does not support native Windows installation; Windows users should run these commands inside an Ubuntu environment under WSL2. For macOS or non-x86_64 systems, use the matching platform-specific environment file from the [QIIME 2 installation documentation](https://amplicon-docs.qiime2.org/en/2025.4/how-to-guides/install.html).

### 3. Install OomyAmplicon

Activate the QIIME 2 environment first. If you cloned the repository, install the release artifact directly:

```bash
conda activate qiime2-amplicon-2025.4
conda install ./packaging/releases/0.1.0/noarch/oomyamplicon-0.1.0-0.conda
```

If downloading from a GitHub Release, replace `OWNER/REPO` with the repository path and run these commands in the directory containing the downloaded files:

```bash
curl -L -O https://github.com/OWNER/REPO/releases/download/v0.1.0/oomyamplicon-0.1.0-0.conda
curl -L -O https://github.com/OWNER/REPO/releases/download/v0.1.0/oomyamplicon-0.1.0-0.conda.sha256
sha256sum -c oomyamplicon-0.1.0-0.conda.sha256
conda install ./oomyamplicon-0.1.0-0.conda
```

Verify the command and display its options:

```bash
oomyamplicon --version
oomyamplicon quality --help
oomyamplicon asv --help
```

The Conda package is a lightweight launcher and workflow; it does not bundle QIIME 2 or a reference database. Supply a pre-trained classifier or reference FASTA plus taxonomy TSV when running an analysis.

## Input and modes

Provide one FASTQ/FQ file per sample for single-end data, or matched R1/R2 files per sample for paired-end data. Gzip-compressed FASTQ and ZIP bundles are accepted. Paired files must include `_R1`/`_R2` or `_1`/`_2` in their filenames.

The default method is DADA2 ASV denoising. Taxonomy annotation requires a reference database supplied by the user. For raw reads, specify the amplification primer sequence(s); Cutadapt removes primers before denoising. If inline barcodes remain, reads must already be split into per-sample FASTQ files and a TSV barcode map is required. A pooled multiplexed FASTQ is not demultiplexed automatically in this version. Use `--clean-data` when both primers and inline barcode tags have already been removed.

## Two-stage run: quality preview, then ASV analysis

Run `quality` first with the input and trimming options planned for analysis. It imports FASTQ, removes inline barcodes if supplied, removes primers for raw reads, and writes `quality_summary.qzv`. Open the interactive plot, choose truncation positions that retain acceptable quality, and verify that paired reads will still overlap. This step does not generate ASVs.

```bash
oomyamplicon quality \
  --input-dir raw_fastq \
  --output-dir results/quality_check \
  --read-mode paired \
  --forward-primer CGGAAGGATCATTACCAC \
  --reverse-primer AGCCTAGACATCCACTGCTG \
  --threads 8

qiime tools view results/quality_check/quality_summary.qzv
```

After reviewing the plot, run `oomyamplicon asv` with the selected `--trunc-len-f` and `--trunc-len-r`. The analysis reruns import and trimming in its own work directory before DADA2 and taxonomy annotation.

### Raw paired-end FASTQ

```bash
oomyamplicon asv \
  --input-dir raw_fastq \
  --output-dir results/asv_run \
  --read-mode paired \
  --forward-primer CGGAAGGATCATTACCAC \
  --reverse-primer AGCCTAGACATCCACTGCTG \
  --classifier classifiers/custom_oomycete_classifier.qza \
  --reference-name UNITE_OomyHybrid_v1.0.0 \
  --threads 8
```

### Already-clean paired-end FASTQ

```bash
oomyamplicon asv \
  --input-dir clean_fastq \
  --output-dir results/asv_run \
  --read-mode paired \
  --clean-data \
  --classifier classifiers/custom_oomycete_classifier.qza \
  --reference-name UNITE_OomyHybrid_v1.0.0 \
  --trunc-len-f 240 \
  --trunc-len-r 200 \
  --threads 8
```

Single-end mode uses `--read-mode single` and only the forward primer/truncation options. To train a classifier at run time instead, provide `--reference-fasta reference.fasta --reference-taxonomy taxonomy.tsv`; the TSV must have `Feature ID` and `Taxon` columns matching the FASTA IDs. For repeated runs, train once and pass the resulting `.qza` with `--classifier`. `--confidence` sets the classify-sklearn confidence cutoff (default 0.7). Defaults retain all bases (`trunc-len=0`, `trim-left=0`); inspect read quality first and set truncation/trim parameters for the actual run. DADA2 removes low-quality reads and chimeras during denoising. The analysis log records every invoked command.

### Inline barcode map

For sample-split raw reads containing inline barcode tags, supply a tab-separated file with `sample-id`, `barcode-forward`, and (for paired-end) `barcode-reverse` columns:

```bash
oomyamplicon asv --input-dir raw_fastq --output-dir results/asv_run \
  --read-mode paired --barcode-map barcodes.tsv \
  --forward-primer CGGAAGGATCATTACCAC --reverse-primer AGCCTAGACATCCACTGCTG \
  --classifier classifiers/custom_oomycete_classifier.qza
```

## Output

- `asv_table.tsv`: ASV-by-sample read counts.
- `representative_sequences.fasta`: one representative sequence per ASV.
- `taxonomy_table.tsv`: assigned taxonomy and classifier confidence for each ASV.
- `denoising_stats.tsv`: input, filtered, denoised, merged, and non-chimeric read counts.
- `analysis_parameters.json`: read mode, primers, truncation settings, sample count, and runtime environment.
- `analysis.log`: commands and progress/errors.

Taxonomic resolution depends on the reference database. An oomycete-only reference can force non-oomycete sequences into oomycete lineages; use a broad background reference when screening mixed-community reads.

## Build from source

With `conda-build` installed in a Conda environment:

```bash
python packaging/build_conda_package.py --conda-exe /path/to/conda
```

The `.conda` package is written to `packaging/releases/0.1.0/noarch/`. Install the generated archive by path into the active QIIME 2 environment.
