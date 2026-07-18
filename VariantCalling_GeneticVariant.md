# Variant Calling: From Sequencing Reads to Genetic Variants

---

# Overview

Variant calling is the process of identifying differences between an individual's genome and a reference genome. These differences, known as **genetic variants**, include single nucleotide polymorphisms (SNPs), insertions and deletions (indels), multiple nucleotide polymorphisms (MNPs), and larger structural variants.

Variant calling is one of the most widely used analyses in genomics and underpins research in population genetics, evolutionary biology, agriculture, medical genetics, microbial genomics, and cancer genomics.

Unlike **de novo genome assembly**, which reconstructs an unknown genome from sequencing reads, variant calling aligns sequencing reads to an existing reference genome and identifies positions where the sample differs from that reference.

---

# Learning Objectives

By the end of this section, participants should be able to:

- Explain the principles of variant calling.
- Differentiate germline and somatic variants.
- Describe the complete variant calling workflow.
- Perform variant calling using widely adopted software.
- Understand variant filtering and quality control.
- Interpret Variant Call Format (VCF) files.
- Annotate variants using public databases.
- Evaluate variant calling quality.

---

# What is a Genetic Variant?

A genetic variant is a DNA sequence difference between two genomes.

Example:

Reference sequence

```text
ATCGATCGATCG
```

Sample sequence

```text
ATCGATTGATCG
```

At one position:

```text
Reference: C

Sample:    T
```

This represents a **Single Nucleotide Polymorphism (SNP).**

---

# Types of Genetic Variants

## Single Nucleotide Polymorphism (SNP)

One nucleotide changes.

Example

```text
Reference

A T C G A

Sample

A T T G A
```

---

## Insertions

Additional nucleotides are present.

Reference

```text
ATCGA
```

Sample

```text
ATCCGA
```

Insertion

```
C
```

---

## Deletions

One or more nucleotides are absent.

Reference

```text
ATCGA
```

Sample

```text
ATGA
```

Deleted

```
C
```

---

## Multiple Nucleotide Polymorphism (MNP)

Several adjacent nucleotides change simultaneously.

Reference

```text
ATCG
```

Sample

```text
GCTA
```

---

## Structural Variants (SVs)

Large genomic changes (>50 bp).

Examples

- Large insertions
- Large deletions
- Duplications
- Inversions
- Translocations
- Copy Number Variants (CNVs)

---

# Germline vs Somatic Variants

## Germline Variants

Present in every cell.

Inherited from parents.

Applications

- Human genetics
- Population genomics
- Evolutionary biology
- Plant breeding

---

## Somatic Variants

Occur after fertilization.

Present in only some cells.

Applications

- Cancer genomics
- Tumor evolution
- Precision medicine

---

# Variant Calling Workflow

```text
Raw FASTQ Files
        │
        ▼
Quality Control
        │
        ▼
Read Trimming
        │
        ▼
Read Alignment
        │
        ▼
Sorting BAM
        │
        ▼
Mark Duplicates
        │
        ▼
Base Quality Recalibration (optional)
        │
        ▼
Variant Calling
        │
        ▼
Variant Filtering
        │
        ▼
Variant Annotation
        │
        ▼
Biological Interpretation
```

---

# Input Files

Variant calling requires:

- Reference genome
- Sequencing reads
- Reference annotation (optional)

Example

```text
reference.fa

sample_R1.fastq.gz

sample_R2.fastq.gz
```

---

# Step 1 – Quality Assessment

Assess sequencing quality using FastQC.

```bash
fastqc sample_R1.fastq.gz sample_R2.fastq.gz
```

Summarize reports.

```bash
multiqc .
```

Evaluate:

- Base quality
- Adapter contamination
- GC distribution
- Sequence duplication

---

# Step 2 – Read Trimming

Trim adapters and low-quality bases.

```bash
fastp \
-i sample_R1.fastq.gz \
-I sample_R2.fastq.gz \
-o sample_R1.trimmed.fastq.gz \
-O sample_R2.trimmed.fastq.gz \
-h fastp.html
```

---

# Step 3 – Reference Genome Indexing

Before alignment, index the reference genome.

Using BWA:

```bash
bwa index reference.fa
```

---

# Step 4 – Read Alignment

Align reads to the reference genome.

```bash
bwa mem \
reference.fa \
sample_R1.trimmed.fastq.gz \
sample_R2.trimmed.fastq.gz \
> sample.sam
```

Output

```
SAM
```

---

# Step 5 – Convert SAM to BAM

SAM files are large text files.

Convert to BAM.

```bash
samtools view \
-bS sample.sam \
> sample.bam
```

---

# Step 6 – Sort BAM

Sort alignments.

```bash
samtools sort \
sample.bam \
-o sample.sorted.bam
```

---

# Step 7 – Index BAM

```bash
samtools index \
sample.sorted.bam
```

Produces

```text
sample.sorted.bam.bai
```

---

# Step 8 – Mark PCR Duplicates

PCR amplification may produce duplicate reads that can bias allele frequency estimates.

Using Picard:

```bash
picard MarkDuplicates \
I=sample.sorted.bam \
O=sample.dedup.bam \
M=metrics.txt
```

Index

```bash
samtools index sample.dedup.bam
```

---

# Why Mark Duplicates?

Duplicates are reads originating from the same DNA fragment rather than independent sequencing observations.

Failure to identify duplicates may:

- Inflate read depth
- Increase false-positive variants
- Bias allele frequency estimation

---

# Step 9 – Base Quality Score Recalibration (BQSR)

Sequencing machines occasionally assign inaccurate quality scores.

Base Quality Score Recalibration (BQSR) models systematic sequencing errors and recalibrates base quality values.

This step is commonly performed in the GATK Best Practices workflow for human germline variant calling.

For organisms without known variant databases, BQSR is often omitted.

---

# Variant Calling Algorithms

Variant callers use statistical models to distinguish true genetic variants from sequencing errors.

Common approaches include:

- Bayesian inference
- Likelihood-based models
- Haplotype reconstruction
- Local de novo assembly

---

# Popular Variant Callers

| Software | Best Application |
|-----------|-----------------|
| GATK HaplotypeCaller | Germline variants |
| FreeBayes | Population genomics |
| BCFtools | Small genomes |
| DeepVariant | Deep learning |
| Strelka2 | Germline and somatic |
| VarScan2 | Somatic mutations |
| LoFreq | Low-frequency variants |
| Clair3 | Long-read variant calling |

---

# Variant Calling with GATK HaplotypeCaller

Run HaplotypeCaller.

```bash
gatk HaplotypeCaller \
-R reference.fa \
-I sample.dedup.bam \
-O sample.g.vcf.gz \
-ERC GVCF
```

For a single sample, omit `-ERC GVCF` to produce a standard VCF directly:

```bash
gatk HaplotypeCaller \
-R reference.fa \
-I sample.dedup.bam \
-O sample.vcf.gz
```

---

# Variant Calling with FreeBayes

```bash
freebayes \
-f reference.fa \
sample.dedup.bam \
> sample.vcf
```

---

# Variant Calling with BCFtools

Generate genotype likelihoods.

```bash
bcftools mpileup \
-f reference.fa \
sample.dedup.bam \
-Ou
```

Call variants.

```bash
bcftools call \
-m \
-v \
-Oz \
-o variants.vcf.gz
```

Index VCF.

```bash
tabix -p vcf variants.vcf.gz
```

---

# Variant Calling with DeepVariant

DeepVariant uses deep neural networks to classify variants.

Typical workflow:

```text
Aligned BAM

↓

DeepVariant

↓

VCF
```

DeepVariant often achieves state-of-the-art accuracy for germline SNP and indel calling.

---

# Understanding the VCF File

VCF (Variant Call Format) is the standard format for storing genetic variants.

Example

```text
#CHROM POS ID REF ALT QUAL FILTER INFO

chr1 10583 . G A 99 PASS DP=42
```

Meaning:

| Field | Description |
|--------|-------------|
| CHROM | Chromosome |
| POS | Position |
| REF | Reference allele |
| ALT | Alternate allele |
| QUAL | Confidence score |
| FILTER | Pass/fail status |
| INFO | Additional annotations |

---

# Genotype Representation

Examples

```text
0/0

Homozygous reference
```

```text
0/1

Heterozygous
```

```text
1/1

Homozygous alternate
```

---

# Variant Filtering

Raw variant calls contain false positives.

Filtering removes unreliable variants.

Common filtering criteria:

- Quality score (QUAL)
- Read depth (DP)
- Mapping quality (MQ)
- Strand bias (FS)
- Allele balance
- Base quality

Example with BCFtools:

```bash
bcftools filter \
-i 'QUAL>30 && DP>10' \
variants.vcf.gz \
-Oz \
-o filtered.vcf.gz
```

---

# Variant Annotation

Variant annotation predicts the biological impact of variants.

Common annotation includes:

- Gene name
- Coding consequence
- Amino acid change
- Population frequency
- Clinical significance
- Functional prediction

---

# SnpEff

Build database (if needed) and annotate:

```bash
snpEff \
reference_database \
filtered.vcf.gz \
> annotated.vcf
```

Outputs include annotations such as:

- synonymous_variant
- missense_variant
- nonsense_variant
- frameshift_variant
- splice_region_variant

---

# Ensembl Variant Effect Predictor (VEP)

```bash
vep \
-i filtered.vcf.gz \
-o annotated.vep.vcf \
--cache
```

VEP predicts:

- Gene affected
- Transcript affected
- Protein consequence
- HGVS nomenclature
- SIFT and PolyPhen predictions (where available)

---

# SnpSift

SnpSift is commonly used to filter and query annotated VCF files.

Example:

```bash
SnpSift filter \
"ANN[*].IMPACT has 'HIGH'" \
annotated.vcf \
> high_impact.vcf
```

---

# Evaluating Variant Calling Quality

Quality assessment should include:

- Mapping rate
- Mean coverage
- Duplicate rate
- Transition/Transversion (Ti/Tv) ratio
- Number of SNPs
- Number of indels
- Concordance with known variants (if available)

---

# Common Problems

## Low Coverage

May result in:

- False negatives
- Low-confidence genotypes

---

## PCR Duplicates

Can inflate allele counts and increase false positives.

---

## Mapping Errors

Incorrect alignment in repetitive regions can generate false variant calls.

---

## Reference Bias

Reads containing non-reference alleles may align less efficiently, reducing sensitivity.

---

## Contamination

DNA from another individual or species can produce spurious variants.

---

# Practical Exercises

## Exercise 1

Perform quality control and trimming of paired-end Illumina reads.

---

## Exercise 2

Align reads to the reference genome using BWA-MEM and generate a sorted, indexed BAM file.

---

## Exercise 3

Call variants using:

- GATK HaplotypeCaller
- FreeBayes
- BCFtools

Compare:

- Number of SNPs
- Number of indels
- Runtime

---

## Exercise 4

Filter variants using quality and depth thresholds.

Compare the number of variants before and after filtering.

---

## Exercise 5

Annotate filtered variants using:

- SnpEff
- VEP

Determine:

- Number of synonymous variants
- Number of missense variants
- Number of high-impact variants

---

# Best Practices

- Use a high-quality reference genome.
- Perform rigorous quality control before alignment.
- Mark duplicates when appropriate.
- Use adequate sequencing depth (≥30× for human germline studies is a common recommendation).
- Apply filtering appropriate to the organism and study design.
- Annotate variants using up-to-date databases.
- Validate important variants experimentally when required.
- Document software versions, parameters, and reference databases for reproducibility.

---

# Summary

Variant calling identifies genetic differences between sequencing reads and a reference genome. A typical workflow includes quality control, read trimming, alignment, duplicate marking, variant calling, filtering, annotation, and biological interpretation. Different variant callers use different statistical approaches, and no single tool is optimal for every application.

Accurate variant analysis depends on high-quality sequencing data, careful preprocessing, appropriate filtering, and thorough annotation. The resulting variant datasets provide the foundation for studies in disease genetics, microbial evolution, population genomics, cancer biology, breeding programs, and precision medicine.
