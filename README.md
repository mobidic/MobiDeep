# MobiDeep


![image](https://github.com/user-attachments/assets/0c1280ba-074a-4b9e-925c-85444c6b6241)


An AI-based Metascore for Robust and Scalable Prioritization of Non-Coding Variants in Whole-Genome Sequencing Data

MobiDeep is a metascore for non-coding variants (SNVs only) based on a multilayer perceptron using 5 features: ReMM v.0.4, CADD 1.7, GPN-MSA, and two conservation scores (Cactus 241-way vertebrates and PhyloP primates) to capture different evolutionary depths.

**🌐 Try MobiDeep Online**: MobiDeep is now integrated into the [MobiDetails web application](https://mobidetails.chu-montpellier.fr) for easy variant analysis without any setup required.

## Scoring System

- **Raw score**: 0 = benign, 1 = maximum pathogenicity
- **Log score**: logarithmic transformation using the formula: `log_score = -log10(1 - raw_score)`

### MobiDeep Thresholds

| Classification | Raw score | Log score |
|----------------|-----------|-----------|
| Neutral | < 0.6 | < 0.3979 |
| Likely deleterious | > 0.6 | > 0.3979 |
| High confidence deleteriousness | > 0.9684 | > 1.5 |

**Usage recommendation**: Use the orange threshold (0.6) for general pathogenicity prediction and the red threshold (0.9684) for high confidence deleteriousness assessment. Region-specific thresholds are available in the MobiDetails web interface.

## Available Resources

### 1. Web Application (Recommended for Single Variants)
Visit [MobiDetails](https://mobidetails.chu-montpellier.fr) to:
- Score individual variants instantly
- Access region-specific thresholds with radar view visualization
- View comprehensive variant annotations including MobiDeep scores

### 2. The MobiDeep model (joblib format)

Download the model file, `mobideep_20250520.joblib`, from the [MobiDetails website](https://mobidetails.chu-montpellier.fr/static/resources/mobideep/mobideep.sif).

### 3. Command-Line Tool (For Batch Processing)
Process VCF files locally using our Apptainer container for:
- Batch analysis of multiple variants
- Integration into bioinformatics pipelines
- Offline processing

### 4. Precalculated Genome-Wide Dataset
Download pre-computed dataset of MobiDeep scores for 8,.773 billion single nucleotide variants covering 94.7% of all genomic positions across the GRCh38p14 reference genome. through our download portal: https://mobidetails.chu-montpellier.fr/about

## Command-Line Tool Setup

### Requirements
- Apptainer (or Singularity version 3.5+) installed. See the [Apptainer Installation Guide](https://apptainer.org/docs/user/main/quick_start.html#quick-installation-steps).
- A Linux/macOS environment
- Annotation data files (see Data Setup section below)

### 1. Obtain the Container
Download the ready-to-use container file, `mobideep.sif`, from the [MobiDetails website](https://mobidetails.chu-montpellier.fr/static/resources/mobideep/mobideep.sif).

Otherwise you can rebuild it using the mobideep.def apptainer definitio file.

```bash
apptainer build mobideep.sif mobideep.def
```


### 2. Download Annotation Data
MobiDeep requires several large annotation data files. You can download them from the appropriate websites.

#### Required Files and Organization

| Database | Expected Filename | Version |
|----------|-------------------|---------|
| [CADD (SNVs)](https://cadd.gs.washington.edu/download) | whole_genome_SNVs.tsv.gz | v1.7 |
| [CADD (Indels)](https://cadd.gs.washington.edu/download) | gnomad.genomes.r4.0.indel.tsv.gz | v1.7 |
| [GPN-MSA](https://huggingface.co/datasets/songlab/gpn-msa-hg38-scores) | scores.tsv.bgz | - |
| [ReMM](https://remm.bihealth.org/score) | ReMM_v0.4.hg38.tsv.gz | v0.4 |
| [PhyloP](https://hgdownload.soe.ucsc.edu/goldenPath/hg38/cactus241way/) (241-way) | cactus241way.phyloP.bw | - |
| PhyloP (Primates) | phyloPPrimates.bigWig | - |

**Directory Structure:**
```bash
/path/to/your/annotation_data/
├── whole_genome_SNVs.tsv.gz
├── whole_genome_SNVs.tsv.gz.tbi
├── gnomad.genomes.r4.0.indel.tsv.gz
├── gnomad.genomes.r4.0.indel.tsv.gz.tbi
├── scores.tsv.bgz
├── scores.tsv.bgz.tbi
├── ReMM_v0.4.hg38.noheader.tsv.gz
├── ReMM_v0.4.hg38.noheader.tsv.gz.tbi
├── cactus241way.phyloP.bw
└── phyloPPrimates.bigWig
```

**Important**: Ensure that all gzipped files (.gz, .bgz) are indexed with Tabix. The index files (.tbi) should be in the same directory.

## Usage

### Command-Line Arguments

| Argument | Description | Required |
|----------|-------------|----------|
| `--vcf` | Path to the input VCF file to be scored (hg38/GRCh38) | Yes |
| `--data_dir` | Path to the directory containing all annotation data files | Yes |
| `--output_file` | Path where the output TSV file will be saved | Yes |
| `--threshold` | The probability score threshold to classify a variant as "Pathogenic" | No (Default: 0.6) |

### Example Commands

**Test the container**
```bash
# Define paths for clarity
ANNOTATION_DIR="/data/annotations" # replace with your own path
apptainer run \
    --bind ${ANNOTATION_DIR}:/annotations \
    mobideep.sif \
    --vcf test_variants.vcf \
    --data_dir /annotations \
    --output_file /variants/results.tsv
```

**Basic usage with default threshold (0.6):**
```bash
# Define paths for clarity
ANNOTATION_DIR="/data/annotations" # replace with your own path
VARIANT_DIR="/data/variants" # replace with your own path

# Run the container
apptainer run \
    --bind ${ANNOTATION_DIR}:/annotations \
    --bind ${VARIANT_DIR}:/variants \
    mobideep.sif \
    --vcf /variants/input.vcf \
    --data_dir /annotations \
    --output_file /variants/results.tsv
```

**High confidence analysis (threshold 0.9684):**
```bash
apptainer run \
    --bind ${ANNOTATION_DIR}:/annotations \
    --bind ${VARIANT_DIR}:/variants \
    mobideep.sif \
    --vcf /variants/input.vcf \
    --data_dir /annotations \
    --output_file /variants/results_high_confidence.tsv \
    --threshold 0.9684
```

### Explanation of the --bind Flag
The `--bind` flag makes directories from your computer (the "host") visible inside the container:
- `--bind ${ANNOTATION_DIR}:/annotations`: Mounts your annotation data to the `/annotations` path inside the container
- `--bind ${VARIANT_DIR}:/variants`: Mounts your VCF directory to `/variants` (also where output will be written)

## Output Format

The output is a tab-separated file (.tsv) containing the original variant information along with annotation scores and MobiDeep predictions:

```
#CHROM	POS	ID	REF	ALT	CADD_PHRED	...	MobiDeep_Score	MobiDeep_Class
1	55040253	rs12345	C	T	14.8900	...	0.9543	Pathogenic
10	114221763	.	A	G	5.4321	...	0.0210	Neutral
```

- **MobiDeep_Score**: The raw probability score from the MLP model (0 to 1)
- **MobiDeep_Class**: "Pathogenic" or "Neutral", based on whether the MobiDeep_Score is above or below the specified `--threshold`

## Features

- **Standard VCF Input**: Directly processes standard VCF files (hg38/GRCh38)
- **Comprehensive Annotation**: Enriches variants with scores from top-performing predictors:
  - CADD (v1.7)
  - GPN-MSA
  - ReMM (v0.4)
  - phyloP (cactus241way)
  - phyloP (phyloP Primates)
  - MobiDeep pathogenicity score
- **Flexible Thresholds**: Multiple classification thresholds for different confidence levels
- **Portable & Reproducible**: Distributed as a single Apptainer (.sif) file
- **Web Integration**: Also available through the MobiDetails web interface

## Citation

If you use MobiDeep in your research, please cite:
(BOUAZZAOUI ET AL. - Citation details to be updated)

## License

This project is licensed under the GPL v3 License. See the LICENSE file for details.

## Support

For questions or issues:
- Web application support: Visit [MobiDetails](https://mobidetails.chu-montpellier.fr)
- Command-line tool issues: Open an issue in this repository
