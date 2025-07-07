# MobiDeep


![image](https://github.com/user-attachments/assets/0c1280ba-074a-4b9e-925c-85444c6b6241)

An AI-based Metascore for Robust and Scalable Prioritization of Non-Coding Variants in Whole-Genome Sequencing Data

Command-line tool that annotates non coding variants from a VCF file and predicts their pathogenicity using MobiDeep a pre-trained Multi-Layer Perceptron (MLP) model.

The entire workflow is packaged within an Apptainer (formerly Singularity) container, ensuring complete reproducibility, ease of use, and eliminating complex setup procedures. You can run this tool on any Linux system with Apptainer installed, including High-Performance Computing (HPC) clusters.

## Features

Standard VCF Input: Directly processes standard VCF files for analysis, in hg38/GRCH38

Comprehensive Annotation: Enriches variants with scores from top performing VEPs:

- CADD (v1.7)
- GPN-MSA
- ReMM (v.0.4)
- phyloP (cactus241way)
- phyloP (phyloP Primates)
- MobiDeep: our model pathogenicity score (MobiDeep_Score).

Customizable Threshold: Classifies variants as "Pathogenic" or "Neutral" based on a user-defined threshold (you can use 0.6 by default or region specific threshold).
Portable & Reproducible: Distributed as a single Apptainer (.sif) file. No need to install Python, scikit-learn, or any other dependencies.


## Requirements

Apptainer (or Singularity version 3.5+) installed. See the Apptainer Installation Guide.

A Linux/macOS environment.

The required annotation data files (see Data Setup section below).

## Setup
1. Obtain the Container

Download the ready-to-use container file, mobideep.sif, from this repository.

(Alternatively, for developers, you can build it from source by cloning the repository and running apptainer build --fakeroot mobideep.sif mobideep.def)

2. Download and Organize Annotation Data

MobiDeep requires several large annotation data files. These can be downloaded either from their official sources, or from Mobidetails download page https://mobidetails.chu-montpellier.fr/about and organized in a single directory.



If annotation files are downloaded from their official sources, please ensure you download the versions specified below and place them in a single directory. The filenames must match exactly.

Database	Expected Filename	Version
CADD (SNVs)	whole_genome_SNVs.tsv.gz	v1.7
CADD (Indels)	gnomad.genomes.r4.0.indel.tsv.gz	v1.7
GPN-MSA	scores.tsv.bgz	-
ReMM	ReMM_v0.4.hg38.tsv.gz	v0.4
PhyloP (241-way)	cactus241way.phyloP.bw	-
PhyloP (Primates)	phyloPPrimates.bigWig	-

Your data directory must be structured as follows:

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

Important: Ensure that all gzipped files (.gz, .bgz) are indexed with Tabix. The index files (.tbi) should be in the same directory.

## Usage

The tool is executed via the apptainer run command. You must provide the paths to your VCF file, the annotation data directory, and an output file.

### Command-Line Arguments
```simple text in table
Argument	Description	Required
--vcf	Path to the input VCF file to be scored.	Yes
--data_dir	Path to the directory containing all annotation data files.	Yes
--output_file	Path where the output TSV file will be saved.	Yes
--threshold	The probability score threshold to classify a variant as "Pathogenic".	No (Default: 0.6)
```

### Example Workflow

Let's assume:

The mobideep.sif container is in your current directory.

Your annotation files are located at /data/annotations/.

Your VCF file is at /data/variants/input.vcf.

### bCommand:

```bash
# Define paths for clarity
ANNOTATION_DIR="/data/annotations"
VARIANT_DIR="/data/variants"

# Run the container
apptainer run \
    --bind ${ANNOTATION_DIR}:/annotations \
    --bind ${VARIANT_DIR}:/variants \
    # alternavely, you can use :  --bind ${HOST_PATH}:{HOST_PATH} if you prefer, after assign HOST_PATH=$(pwd)
    mobideep.sif \
    --vcf /variants/input.vcf \
    --data_dir /annotations \
    --output_file /variants/results.tsv
```


### To use a different threshold (e.g., 0.85 for non coding exons):

```bash
apptainer run \
    --bind ${ANNOTATION_DIR}:/annotations \
    --bind ${VARIANT_DIR}:/variants \
    # alternavely, you can use :  --bind ${HOST_PATH}:{HOST_PATH} if you prefer, after assign HOST_PATH=$(pwd)
    mobideep.sif \
    --vcf /variants/input.vcf \
    --data_dir /annotations \
    --output_file /variants/results_t0.85.tsv \
    --threshold 0.85
```



## Explanation of the --bind Flag

The --bind flag makes directories from your computer (the "host") visible inside the container.

--bind ${ANNOTATION_DIR}:/annotations: Mounts your annotation data to the /annotations path inside the container.

--bind ${VARIANT_DIR}:/variants: Mounts your VCF directory to /variants. This is also where the output will be written.

## Output Format

The output is a tab-separated file (.tsv) containing the original variant information along with the collected annotation scores and the final MobiDeep predictions.

#CHROM	POS	ID	REF	ALT	CADD_PHRED	...	MobiDeep_Score	MobiDeep_Class
1	55040253	rs12345	C	T	14.8900	...	0.9543	Pathogenic
10	114221763	.	A	G	5.4321	...	0.0210	Neutral

- MobiDeep_Score: The raw probability score from the MLP model (0 to 1).
- MobiDeep_Class: "Pathogenic" or "Neutral", based on whether the MobiDeep_Score is above or below the specified --threshold.

# Citation

If you use MobiDeep in your research, please cite the MobiDeep project.
(BOUAZZAOUI ET AL. BLABLA)

# License

This project is licensed under the GPL v3 License. See the LICENSE file for details.
