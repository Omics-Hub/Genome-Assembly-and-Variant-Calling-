# Genome Annotation: From Assembled Genomes to Biological Interpretation

---

# Overview

Genome annotation is the process of identifying and assigning biological information to genomic sequences after genome assembly. While genome assembly reconstructs the nucleotide sequence of an organism, annotation determines **what those sequences represent biologically**.

Genome annotation consists of two major components:

1. **Structural Annotation** – Identifying genomic features such as genes, exons, introns, transfer RNAs (tRNAs), ribosomal RNAs (rRNAs), non-coding RNAs (ncRNAs), regulatory elements, and repetitive sequences.

2. **Functional Annotation** – Assigning biological functions to predicted genes using sequence similarity, protein domains, gene ontology, metabolic pathways, and orthology.

Genome annotation transforms an anonymous DNA sequence into a biologically interpretable genome that can be used for downstream analyses such as comparative genomics, transcriptomics, evolutionary biology, and functional studies.

---

# Learning Objectives

By the end of this section, participants should be able to:

- Explain the purpose of genome annotation.
- Differentiate structural and functional annotation.
- Describe bacterial and eukaryotic annotation workflows.
- Identify commonly used annotation software.
- Perform annotation of bacterial genomes.
- Perform annotation of eukaryotic genomes.
- Evaluate annotation quality.
- Interpret annotation outputs.

---

# Genome Annotation Workflow

```text
Genome Assembly
        │
        ▼
Repeat Identification (Eukaryotes)
        │
        ▼
Gene Prediction
        │
        ▼
Protein Prediction
        │
        ▼
Functional Annotation
        │
        ▼
Gene Ontology
        │
        ▼
Pathway Annotation
        │
        ▼
Annotated Genome
```

---

# Structural Annotation

Structural annotation identifies the physical locations of genomic features.

Typical features include:

- Protein-coding genes
- Exons
- Introns
- Start codons
- Stop codons
- UTRs
- tRNAs
- rRNAs
- snoRNAs
- miRNAs
- Regulatory regions
- Repetitive DNA

---

# Functional Annotation

Functional annotation attempts to determine the biological role of predicted genes.

Common approaches include:

- Sequence similarity searches
- Protein domain analysis
- Orthology inference
- Gene Ontology assignment
- KEGG pathway mapping
- Enzyme classification
- Protein family identification

---

# Bacterial Genome Annotation

## Characteristics of Bacterial Genomes

Compared to eukaryotic genomes, bacterial genomes are relatively simple.

Characteristics include:

- Circular chromosomes
- High gene density
- Few repetitive regions
- No introns (in most bacteria)
- Operons
- Polycistronic transcripts

Because bacterial genes generally lack introns, gene prediction is considerably easier than in eukaryotes.

---

# Bacterial Annotation Workflow

```text
Genome Assembly
      │
      ▼
Quality Assessment
      │
      ▼
Gene Prediction
      │
      ▼
Protein Translation
      │
      ▼
Functional Annotation
      │
      ▼
Annotated Genome
```

---

# Common Bacterial Annotation Software

| Software | Purpose |
|-----------|----------|
| Prokka | Complete bacterial annotation |
| Bakta | Modern bacterial annotation |
| DFAST | Rapid annotation |
| PGAP | NCBI annotation pipeline |
| RASTtk | Web-based annotation |

---

# Practical: Annotation with Prokka

## Install

```bash
conda install -c bioconda prokka
```

---

## Input

```
assembly.fasta
```

---

## Run Annotation

```bash
prokka \
assembly.fasta \
--outdir prokka_output \
--prefix ecoli
```

---

## Important Outputs

```text
ecoli.gff

ecoli.gbk

ecoli.faa

ecoli.ffn

ecoli.fna

ecoli.tbl

ecoli.tsv
```

---

## Output Description

| File | Description |
|------|-------------|
| GFF | Genome annotation |
| GBK | GenBank file |
| FAA | Protein sequences |
| FFN | Coding nucleotide sequences |
| FNA | Genome sequence |
| TSV | Annotation summary |

---

# Annotation with Bakta

Install

```bash
conda install -c bioconda bakta
```

Run

```bash
bakta \
--db bakta_db \
--output bakta_output \
assembly.fasta
```

Advantages

- Updated databases
- Improved protein annotation
- Better pseudogene detection

---

# NCBI PGAP

PGAP is the annotation pipeline used by NCBI for RefSeq genomes.

Advantages

- High-quality standardized annotation
- Required for RefSeq submissions

Workflow

```text
Assembly

↓

PGAP

↓

Annotated Genome

↓

NCBI Submission
```

---

# Functional Annotation of Bacterial Proteins

Protein sequences predicted by Prokka can be annotated further.

---

## BLAST

Compare proteins against known databases.

```bash
blastp \
-query proteins.faa \
-db nr \
-outfmt 6 \
-out blast_results.tsv
```

---

## InterProScan

Identify conserved protein domains.

```bash
interproscan.sh \
-i proteins.faa \
-appl Pfam \
-f TSV,GFF3
```

Outputs

- Pfam
- SMART
- TIGRFAM
- SUPERFAMILY
- Gene Ontology

---

## eggNOG-mapper

Assign

- Orthologs
- GO terms
- KEGG pathways
- COG categories

```bash
emapper.py \
-i proteins.faa \
-o eggnog
```

---

# Eukaryotic Genome Annotation

## Why is Eukaryotic Annotation More Difficult?

Eukaryotic genomes contain:

- Introns
- Exons
- Alternative splicing
- Large intergenic regions
- Repetitive DNA
- Pseudogenes

Gene prediction therefore requires additional evidence beyond the genome sequence alone.

---

# Typical Eukaryotic Annotation Workflow

```text
Genome Assembly
        │
        ▼
Repeat Masking
        │
        ▼
RNA-seq Alignment
        │
        ▼
Protein Alignment
        │
        ▼
Ab Initio Gene Prediction
        │
        ▼
Evidence Integration
        │
        ▼
Functional Annotation
```

---

# Repeat Masking

Most eukaryotic genomes contain large amounts of repetitive DNA.

Repeats can interfere with gene prediction by producing false genes.

Therefore,

repeat masking is usually the first annotation step.

---

# RepeatModeler

Build a repeat library.

```bash
BuildDatabase \
-name genome_db \
genome.fasta

RepeatModeler \
-database genome_db
```

---

# RepeatMasker

Mask repetitive regions.

```bash
RepeatMasker \
-pa 8 \
-lib consensi.fa.classified \
genome.fasta
```

Output

```
genome.fasta.masked
```

---

# RNA-seq Alignment

RNA-seq provides experimental evidence of expressed genes.

Common aligners:

- HISAT2
- STAR
- minimap2

Example

```bash
hisat2 \
-x genome \
-1 reads_R1.fastq.gz \
-2 reads_R2.fastq.gz \
-S rnaseq.sam
```

---

# Protein Evidence

Proteins from closely related species improve annotation accuracy.

Alignment tools include

- BLAST
- Exonerate
- GenomeThreader
- miniprot

---

# Ab Initio Gene Prediction

Ab initio predictors identify genes using statistical models.

Examples

- AUGUSTUS
- GeneMark
- SNAP
- GlimmerHMM

---

# AUGUSTUS

```bash
augustus \
--species=arabidopsis \
genome.masked.fasta
```

Output

```
GFF

Protein sequences

Coding sequences
```

---

# BRAKER3

BRAKER integrates

- RNA-seq
- Protein evidence
- GeneMark
- AUGUSTUS

Fully automated annotation.

```bash
braker.pl \
--genome genome.masked.fasta \
--bam rnaseq.bam \
--prot_seq proteins.faa
```

BRAKER is currently one of the most widely used annotation pipelines for newly assembled eukaryotic genomes.

---

# MAKER

MAKER combines

- Protein evidence
- RNA evidence
- Ab initio prediction

Workflow

```text
Genome

↓

RepeatMasker

↓

RNA Alignment

↓

Protein Alignment

↓

Gene Prediction

↓

Final Annotation
```

Run

```bash
maker
```

---

# EvidenceModeler (EVM)

EVM combines predictions from

- AUGUSTUS
- GeneMark
- SNAP
- RNA evidence
- Protein evidence

to generate consensus gene models.

---

# Functional Annotation of Eukaryotic Genes

After structural annotation,

proteins are functionally annotated.

---

## InterProScan

```bash
interproscan.sh \
-i proteins.faa \
-f tsv,gff3
```

---

## eggNOG-mapper

```bash
emapper.py \
-i proteins.faa
```

---

## BLAST

```bash
blastp \
-query proteins.faa \
-db swissprot \
-outfmt 6
```

---

## KEGG Annotation

```bash
kofamscan \
-f mapper \
proteins.faa
```

Assigns

- KEGG Orthologs
- Metabolic pathways

---

# Annotation Quality Assessment

Annotation should always be evaluated.

Metrics include:

- Number of genes
- Average gene length
- Exons per gene
- BUSCO completeness
- Functional annotation rate

---

# BUSCO on Gene Predictions

```bash
busco \
-i proteins.faa \
-m proteins \
-l eukaryota_odb10
```

High BUSCO completeness indicates a high-quality annotation.

---

# Common Annotation Outputs

| File | Description |
|------|-------------|
| GFF3 | Gene coordinates |
| GTF | Gene annotation |
| FAA | Protein sequences |
| CDS | Coding sequences |
| mRNA | Transcript sequences |
| GBK | GenBank file |

---

# Comparing Annotation Pipelines

## Bacteria

| Pipeline | Recommended |
|-----------|------------|
| Prokka | ✓ |
| Bakta | ✓ |
| PGAP | ✓ |
| DFAST | ✓ |

---

## Eukaryotes

| Pipeline | Recommended |
|-----------|------------|
| BRAKER3 | ✓✓✓ |
| MAKER | ✓✓ |
| AUGUSTUS | ✓ |
| GeneMark | ✓ |
| EvidenceModeler | ✓✓ |

---

# Practical Exercises

## Exercise 1

Annotate a bacterial genome using:

- Prokka
- Bakta

Compare:

- Number of genes
- tRNAs
- rRNAs
- CDS

---

## Exercise 2

Annotate a eukaryotic genome using:

- RepeatMasker
- BRAKER3

Evaluate

- Number of predicted genes
- BUSCO completeness

---

## Exercise 3

Run InterProScan on predicted proteins.

Determine:

- Number of proteins with Pfam domains
- GO terms assigned
- KEGG pathways identified

---

## Exercise 4

Compare annotations produced by:

- AUGUSTUS
- BRAKER3

Discuss differences.

---

# Best Practices

- Use a high-quality genome assembly before annotation.
- Mask repetitive elements in eukaryotic genomes.
- Incorporate RNA-seq and protein evidence whenever possible.
- Use lineage-specific BUSCO datasets to assess annotation completeness.
- Perform functional annotation using multiple complementary databases (e.g., InterPro, eggNOG, KEGG, Swiss-Prot).
- Manually inspect important genes or gene families of interest.
- Document software versions, databases, and parameters to ensure reproducibility.

---

# Summary

Genome annotation converts an assembled genome into a biologically meaningful resource by identifying genes and assigning putative functions. **Bacterial genome annotation** is relatively straightforward because of compact genomes with few introns, and tools such as **Prokka**, **Bakta**, and **PGAP** can produce high-quality annotations rapidly. **Eukaryotic genome annotation** is substantially more complex due to introns, alternative splicing, and repetitive DNA, requiring repeat masking, transcript and protein evidence, and sophisticated gene prediction pipelines such as **BRAKER3** and **MAKER**.

The highest-quality annotations combine **ab initio predictions**, **RNA-seq evidence**, **homology-based evidence**, and **functional annotation** from multiple databases. Careful evaluation using metrics such as BUSCO completeness and functional annotation coverage is essential before using annotated genomes for downstream analyses.
