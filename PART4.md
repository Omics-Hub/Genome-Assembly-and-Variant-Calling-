# Part 4 — Hands-on Practical: De Novo Genome Assembly Using Illumina Short Reads

---

# Practical Overview

This practical provides a complete end-to-end genome assembly workflow using Illumina paired-end sequencing data. Participants will perform quality assessment, trim sequencing reads, assemble a bacterial genome using SPAdes, evaluate the resulting assembly, assess completeness with BUSCO, validate the assembly by mapping reads back to the assembled contigs, and interpret the results.

The practical emphasizes understanding why each step is performed rather than simply executing commands. By the end of the exercise, participants should be able to perform a complete de novo genome assembly independently.

---

# Practical Learning Objectives

Upon completion of this practical, participants should be able to:

- Organize sequencing data for assembly
- Perform quality assessment of raw reads
- Interpret FastQC reports
- Trim adapters and low-quality bases
- Assemble a genome using SPAdes
- Interpret assembly statistics
- Evaluate completeness using BUSCO
- Map sequencing reads back to the assembly
- Troubleshoot common assembly problems
- Produce a reproducible assembly report

---

# Dataset

This practical assumes paired-end Illumina sequencing data.

Example directory structure:

```text
GenomeAssemblyWorkshop/
│
├── raw_reads/
│   ├── sample_R1.fastq.gz
│   └── sample_R2.fastq.gz
│
├── qc/
├── trimmed/
├── assembly/
├── quast/
├── busco/
├── mapping/
└── reports/
```

---

# Required Software

Install the following software before beginning.

| Software | Purpose |
|-----------|----------|
| FastQC | Read quality assessment |
| MultiQC | Aggregate FastQC reports |
| fastp | Read trimming |
| SPAdes | Genome assembly |
| QUAST | Assembly evaluation |
| BUSCO | Genome completeness |
| BWA | Read alignment |
| SAMtools | BAM processing |
| Bandage | Assembly graph visualization (optional) |

---

# Conda Installation

Create a Conda environment:

```bash
conda create -n genome_assembly \
python=3.11 \
fastqc \
multiqc \
fastp \
spades \
quast \
busco \
bwa \
samtools \
bandage \
-y
```

Activate:

```bash
conda activate genome_assembly
```

Verify installation:

```bash
fastqc --version

fastp --version

spades.py --version

quast.py --version

busco --version
```

---

# Step 1 — Inspect the Sequencing Data

Before beginning any analysis, inspect the FASTQ files.

```bash
ls raw_reads/
```

Example:

```text
sample_R1.fastq.gz

sample_R2.fastq.gz
```

Determine file sizes:

```bash
du -sh raw_reads/*
```

Count reads:

```bash
zcat raw_reads/sample_R1.fastq.gz | echo $((`wc -l`/4))
```

Remember:

Each sequencing read occupies four lines in a FASTQ file.

---

# Step 2 — Quality Assessment

Run FastQC.

```bash
mkdir qc

fastqc \
raw_reads/*.fastq.gz \
-o qc
```

Expected output:

```text
sample_R1_fastqc.html

sample_R1_fastqc.zip

sample_R2_fastqc.html

sample_R2_fastqc.zip
```

---

# Step 3 — Summarize Results

Generate a MultiQC report.

```bash
multiqc qc \
-o qc
```

Open

```text
multiqc_report.html
```

Review:

- Per base quality
- Adapter content
- GC distribution
- Sequence duplication
- Overrepresented sequences

---

# Discussion

Before proceeding, answer:

1. Is the overall read quality acceptable?

2. Are sequencing adapters present?

3. Does GC content appear normal?

4. Are low-quality bases concentrated near the 3′ end?

5. Would trimming improve assembly quality?

---

# Step 4 — Read Trimming

Trim adapters and low-quality bases.

```bash
mkdir trimmed

fastp \
-i raw_reads/sample_R1.fastq.gz \
-I raw_reads/sample_R2.fastq.gz \
-o trimmed/sample_R1.trimmed.fastq.gz \
-O trimmed/sample_R2.trimmed.fastq.gz \
-h trimmed/fastp.html \
-j trimmed/fastp.json \
--detect_adapter_for_pe
```

Outputs:

```text
sample_R1.trimmed.fastq.gz

sample_R2.trimmed.fastq.gz

fastp.html

fastp.json
```

---

# Why Trim?

Trimming removes:

- adapter contamination
- low-quality bases
- sequencing artifacts

Benefits include:

- fewer sequencing errors
- fewer false k-mers
- improved graph construction
- higher-quality assemblies

---

# Step 5 — Assess Trimmed Reads

Run FastQC again.

```bash
mkdir qc_trimmed

fastqc \
trimmed/*.fastq.gz \
-o qc_trimmed
```

Generate a new MultiQC report.

```bash
multiqc qc_trimmed \
-o qc_trimmed
```

Compare:

Raw reads

vs

Trimmed reads

---

# Discussion

Questions:

- Did adapter contamination disappear?

- Did average quality improve?

- Were many reads removed?

- Was read length substantially reduced?

---

# Step 6 — Genome Assembly

Create an output directory.

```bash
mkdir assembly
```

Run SPAdes.

```bash
spades.py \
-1 trimmed/sample_R1.trimmed.fastq.gz \
-2 trimmed/sample_R2.trimmed.fastq.gz \
-o assembly \
-t 8 \
-m 32
```

Parameters:

| Parameter | Description |
|------------|-------------|
| -1 | Forward reads |
| -2 | Reverse reads |
| -o | Output directory |
| -t | CPU threads |
| -m | RAM (GB) |

---

# SPAdes Workflow

Internally, SPAdes performs:

1. Error correction
2. Construction of multiple De Bruijn graphs
3. Graph simplification
4. Repeat resolution
5. Contig generation
6. Scaffold generation

---

# Assembly Output

Important files:

```text
assembly/

contigs.fasta

scaffolds.fasta

assembly_graph.fastg

params.txt

spades.log
```

---

# Discussion

Open

```text
spades.log
```

Locate:

- k-mer values
- graph construction
- assembly statistics
- warnings

---

# Step 7 — Assembly Statistics

Run QUAST.

```bash
quast.py \
assembly/contigs.fasta \
-o quast
```

Outputs include:

```text
report.html

report.tsv

transposed_report.tsv
```

---

# Metrics to Examine

- Number of contigs
- Largest contig
- Total length
- GC%
- N50
- L50

---

# Interpretation Exercise

Consider the following assembly:

| Metric | Value |
|---------|-------|
| Contigs | 45 |
| Largest contig | 725 kb |
| Total Length | 5.1 Mb |
| GC | 50.3% |
| N50 | 320 kb |

Questions:

- Is the assembly fragmented?

- Does total length appear reasonable?

- Is GC consistent with expectations?

- What additional information is needed?

---

# Step 8 — Evaluate Genome Completeness

Run BUSCO.

```bash
busco \
-i assembly/contigs.fasta \
-m genome \
-l bacteria_odb10 \
-o busco
```

Important output:

```text
short_summary.txt
```

---

# BUSCO Interpretation

Example:

```text
Complete

97%

Duplicated

1%

Fragmented

1%

Missing

1%
```

Interpretation:

The genome assembly is highly complete with very few missing conserved genes.

---

# Discussion

Would you trust an assembly with:

```
N50

2 Mb
```

but

```
BUSCO

55%
```

Explain your reasoning.

---

# Step 9 — Map Reads Back to the Assembly

Index the assembly.

```bash
bwa index assembly/contigs.fasta
```

Map reads.

```bash
bwa mem \
assembly/contigs.fasta \
trimmed/sample_R1.trimmed.fastq.gz \
trimmed/sample_R2.trimmed.fastq.gz \
> mapping/sample.sam
```

Convert SAM to BAM.

```bash
samtools view \
-bS mapping/sample.sam \
> mapping/sample.bam
```

Sort BAM.

```bash
samtools sort \
mapping/sample.bam \
-o mapping/sample.sorted.bam
```

Index BAM.

```bash
samtools index \
mapping/sample.sorted.bam
```

---

# Alignment Statistics

```bash
samtools flagstat \
mapping/sample.sorted.bam
```

Interpret:

- Mapping rate
- Properly paired reads
- Secondary alignments

---

# Discussion

A mapping rate of

```
99%
```

suggests:

- nearly all reads are represented in the assembly.

A mapping rate of

```
65%
```

may indicate:

- contamination
- incomplete assembly
- incorrect assembly
- sequencing problems

---

# Step 10 — Visualize the Assembly Graph (Optional)

Bandage visualizes the assembly graph.

Open:

```text
assembly_graph.fastg
```

Observe:

- graph branches
- unresolved repeats
- circular contigs
- dead ends

Graph visualization helps explain why some contigs remain fragmented.

---

# Step 11 — Compare Assemblies

Repeat the assembly using different k-mer values.

Example:

```bash
spades.py \
-k 21,33,55
```

Compare:

- N50
- Number of contigs
- Largest contig
- BUSCO completeness

Discussion:

How does k-mer size influence assembly quality?

---

# Practical Exercises

## Exercise 1

Calculate sequencing coverage.

Given:

Genome size

```
4.8 Mb
```

Reads

```
24 million
```

Read length

```
150 bp
```

Questions:

- What is the sequencing coverage?
- Is it sufficient for assembly?

---

## Exercise 2

Interpret FastQC.

Given a report showing:

- High adapter content
- Low-quality 3′ ends
- High duplication

Recommend an appropriate preprocessing strategy.

---

## Exercise 3

Interpret QUAST.

Compare two assemblies.

| Metric | Assembly A | Assembly B |
|---------|------------|------------|
| Contigs | 30 | 150 |
| N50 | 450 kb | 120 kb |
| BUSCO | 91% | 98% |

Which assembly is preferable?

Justify your answer.

---

## Exercise 4

Troubleshooting

An assembly produced:

- 5,400 contigs
- N50 = 8 kb
- BUSCO = 52%

Suggest possible causes.

---

## Exercise 5

Research Exercise

Investigate one of the following assemblers:

- Flye
- Canu
- Hifiasm
- ABySS
- Velvet

Prepare a brief presentation discussing:

- sequencing technology
- assembly algorithm
- strengths
- weaknesses
- applications

---

# Mini Project

Each student will receive an unknown Illumina paired-end dataset.

Tasks:

1. Perform quality control.
2. Trim sequencing reads.
3. Assemble the genome.
4. Evaluate the assembly.
5. Assess completeness.
6. Map reads back to the assembly.
7. Interpret the results.

---

# Final Report

Each student should prepare a report containing:

## Introduction

Describe:

- organism
- sequencing technology
- assembly objective

---

## Methods

Describe:

- software
- versions
- parameters

---

## Results

Include:

- FastQC summary
- QUAST metrics
- BUSCO results
- Mapping statistics

---

## Discussion

Interpret:

- assembly quality
- completeness
- limitations
- possible improvements

---

## Conclusion

Summarize:

- assembly success
- challenges encountered
- recommendations for future analyses

---

# Best Practices

- Always inspect raw reads before assembly.
- Do not skip quality control.
- Remove adapters before assembly.
- Keep detailed records of software versions and parameters.
- Evaluate assemblies using multiple complementary metrics.
- Avoid relying on N50 alone.
- Validate assemblies by mapping reads back to the assembly.
- Use BUSCO to assess completeness.
- Archive raw data and final assemblies.
- Ensure analyses are reproducible using workflow management systems such as Snakemake or Nextflow.

---

# Suggested Reading

1. Compeau, P. E. C., & Pevzner, P. A. *Bioinformatics Algorithms: An Active Learning Approach.*

2. Miller, J. R., Koren, S., & Sutton, G. (2010). Assembly algorithms for next-generation sequencing data. *Genomics.*

3. Nagarajan, N., & Pop, M. (2013). Sequence assembly demystified. *Nature Reviews Genetics.*

4. Pevzner, P. A. *Computational Molecular Biology.*

5. Schatz, M. C., Delcher, A. L., & Salzberg, S. L. Assembly of large genomes using second-generation sequencing. *Genome Research.*

---

# Course Summary

Throughout this course, you have learned how genomes are reconstructed from sequencing reads using computational algorithms. Beginning with DNA sequencing technologies and progressing through graph-based assembly methods, quality assessment, assembly generation, evaluation, polishing, and validation, you have explored the complete workflow used in modern genome assembly projects.

The practical component demonstrated how these concepts are implemented using widely adopted bioinformatics tools. While this course focused on Illumina short-read assembly, the same principles extend to long-read and hybrid assembly strategies, with appropriate modifications to the algorithms and software.

Genome assembly continues to evolve as sequencing technologies improve, producing increasingly contiguous and accurate assemblies. A strong understanding of the biological principles, computational algorithms, and quality assessment methods presented in this course provides the foundation for advanced studies in comparative genomics, functional genomics, evolutionary biology, metagenomics, and genome annotation.
