# Module 14 – The Genome Assembly Workflow

## Introduction

Genome assembly is not a single computational step but rather a multi-stage bioinformatics workflow. Each stage influences the quality of the final assembly. Poor-quality sequencing reads, inappropriate parameter selection, or inadequate quality control can result in fragmented or inaccurate assemblies regardless of the assembler used.

A typical genome assembly workflow consists of the following stages:

```text
DNA Extraction
      │
      ▼
Library Preparation
      │
      ▼
DNA Sequencing
      │
      ▼
Raw FASTQ Files
      │
      ▼
Quality Control
      │
      ▼
Read Trimming & Filtering
      │
      ▼
Error Correction (Optional)
      │
      ▼
Genome Assembly
      │
      ▼
Assembly Evaluation
      │
      ▼
Assembly Polishing
      │
      ▼
Genome Annotation
      │
      ▼
Downstream Analysis
```

Each stage is discussed below.

---

# Module 15 – Quality Assessment of Raw Reads

## Why Perform Quality Control?

Sequencing instruments are not perfect. Raw sequencing data often contain:

- Adapter contamination
- Low-quality bases
- PCR duplicates
- Overrepresented sequences
- Uneven GC content
- Sequence duplication
- Poor-quality sequencing cycles

These issues can negatively affect assembly quality by introducing incorrect overlaps, false graph branches, and fragmented contigs.

Quality assessment identifies these problems before assembly begins.

---

# FastQC

FastQC is the most widely used program for evaluating sequencing quality.

Input:

```
FASTQ files
```

Output:

```
HTML report
```

Each report contains several modules.

---

## Per Base Sequence Quality

This plot shows the quality score at every sequencing position.

Example:

```text
Quality

40 ─────────────────────────

35 ────────────────────────

30 ─────────────────────

25 ───────────────

20 ─────────

15 ─────

     Base Position →
```

Good sequencing runs generally maintain high quality across the read.

Illumina reads often show decreasing quality toward the 3′ end.

---

## Per Sequence Quality Scores

This module summarizes the average quality score of each read.

A high-quality dataset should have most reads clustered around Q30 or higher.

---

## Per Base GC Content

GC content is calculated at every base position.

Unexpected patterns may indicate:

- contamination
- amplification bias
- sequencing artifacts

---

## Per Sequence GC Content

The distribution of GC percentages should approximately follow a normal distribution for most genomes.

Multiple peaks may indicate contamination.

---

## Sequence Length Distribution

Ideally,

all Illumina reads should have nearly identical lengths before trimming.

Large variation may indicate aggressive trimming or mixed datasets.

---

## Adapter Content

Adapters are synthetic DNA sequences added during library preparation.

If adapter sequences remain,

the assembler may incorrectly interpret them as genomic DNA.

Adapters should therefore be removed before assembly.

---

## Overrepresented Sequences

FastQC identifies sequences occurring much more frequently than expected.

Possible causes:

- adapters
- contamination
- ribosomal RNA
- PCR artifacts

---

# MultiQC

Large sequencing projects often contain dozens or hundreds of samples.

Instead of opening multiple FastQC reports individually,

MultiQC combines all reports into a single summary.

Advantages:

- easier comparison
- batch quality assessment
- publication-quality figures

---

# Module 16 – Read Trimming

## Why Trim Reads?

Although assemblers can tolerate some sequencing errors,

removing poor-quality regions generally improves assembly quality.

Common trimming operations include:

- adapter removal
- quality trimming
- length filtering
- poly-G removal
- poly-A trimming

---

## Adapter Removal

Sequencing reads occasionally extend beyond the DNA insert into the sequencing adapters.

Example:

```text
Genome DNA

ATCGATCGATCGATCG

Adapter

AGATCGGAAGAGC
```

If adapters are not removed,

they create artificial overlaps during assembly.

---

## Quality Trimming

Low-quality bases are usually found near the 3′ end of Illumina reads.

Example

Before trimming

```text
ATCGATCGATCGATCGTTAACG
```

After trimming

```text
ATCGATCGATCGATCG
```

Removing unreliable bases reduces false k-mers.

---

## Length Filtering

Very short reads contribute little useful information.

Many assemblers recommend removing reads shorter than 30–50 bp.

---

# Common Trimming Software

| Tool | Advantages |
|-------|------------|
| fastp | Fast, automatic adapter detection |
| Trimmomatic | Highly configurable |
| Cutadapt | Excellent adapter trimming |
| BBDuk | Comprehensive filtering |

Among these, **fastp** is commonly used because it integrates quality filtering, adapter removal, duplication estimation, and HTML report generation in a single program.

---

# Module 17 – Error Correction

Sequencing errors create false k-mers that complicate graph construction.

Error correction identifies rare k-mers that likely resulted from sequencing mistakes and replaces them with more probable alternatives.

Example

True read

```text
ATCGATCG
```

Sequencing error

```text
ATCAATCG
```

Because the erroneous k-mer appears only once while the correct k-mer appears many times, the assembler can often infer the correct sequence.

Some assemblers perform automatic error correction.

Examples include:

- SPAdes
- BFC
- Musket
- Lighter

---

# Module 18 – Genome Assembly

Once reads have been cleaned and corrected,

assembly can begin.

The assembler constructs a graph,

simplifies the graph,

and extracts the most likely genomic paths.

The output typically includes:

```text
contigs.fasta

scaffolds.fasta

assembly_graph.fastg

assembly.log
```

---

## Contigs

Contigs contain only confidently assembled sequence.

No unknown nucleotides occur within contigs.

---

## Scaffolds

Scaffolds connect multiple contigs using additional evidence.

Unknown regions are represented by

```
NNNNNN
```

---

## Assembly Graph

Many assemblers output the assembly graph itself.

The graph provides valuable information regarding:

- unresolved repeats
- branching
- structural ambiguity

Programs such as Bandage allow visualization of assembly graphs.

---

# Choosing Assembly Parameters

Assemblers contain numerous parameters.

Important examples include:

### k-mer size

Determines graph resolution.

### Coverage cutoff

Removes low-frequency k-mers likely caused by sequencing errors.

### Number of threads

Controls computational speed.

### Memory allocation

Large genomes require substantial RAM.

---

# Module 19 – Assembly Statistics

Assembly quality cannot be judged solely by whether an assembly completed successfully.

Several quantitative metrics are used.

---

## Total Assembly Length

The sum of all assembled contigs.

Ideally,

this should approximate the expected genome size.

Significant deviations may indicate:

- contamination
- collapsed repeats
- incomplete assembly

---

## Number of Contigs

Fewer contigs generally indicate a more contiguous assembly.

However,

a single incorrect contig is worse than several correct contigs.

Therefore,

contig count should never be interpreted alone.

---

## Largest Contig

Represents the length of the longest assembled sequence.

Longer contigs often indicate improved assembly continuity.

---

## GC Content

GC percentage should be consistent with the expected organism.

Unexpected GC content may indicate contamination.

---

# N50

N50 measures assembly contiguity.

Calculation:

1. Sort contigs by length.
2. Calculate cumulative length.
3. Identify the contig where cumulative length reaches 50% of the assembly.

Example

| Contig | Length |
|---------|--------|
| 1 | 900 kb |
| 2 | 700 kb |
| 3 | 300 kb |
| 4 | 100 kb |

Total assembly

```
2.0 Mb
```

Half

```
1.0 Mb
```

The cumulative length exceeds 1 Mb after the second contig.

Therefore

```
N50 = 700 kb
```

Higher N50 generally indicates better assembly continuity.

---

# L50

L50 is the number of contigs required to reach 50% of the assembly.

Using the previous example,

two contigs reach half of the assembly.

```
L50 = 2
```

Smaller L50 values indicate more contiguous assemblies.

---

# Module 20 – Assembly Evaluation

Producing an assembly is only the beginning.

Evaluation determines whether the assembly accurately represents the genome.

---

# QUAST

QUAST (Quality Assessment Tool for Genome Assemblies) is the standard program for evaluating genome assemblies.

QUAST reports:

- Total assembly length
- Number of contigs
- Largest contig
- GC%
- N50
- L50
- Misassemblies
- Genome fraction
- Duplication ratio

If a reference genome is available,

QUAST can compare the assembly directly against it.

---

## Interpreting QUAST

Good assemblies generally have:

- high N50
- low L50
- few contigs
- expected genome size
- few misassemblies

However,

multiple metrics should always be interpreted together.

---

# BUSCO

A genome may have excellent contiguity while still missing genes.

BUSCO addresses this problem.

BUSCO

**Benchmarking Universal Single-Copy Orthologs**

evaluates assembly completeness using highly conserved genes expected to exist in nearly all members of a taxonomic lineage.

Example:

```
bacteria_odb10

fungi_odb10

vertebrata_odb10
```

---

BUSCO classifies genes into:

### Complete

Expected genes found completely.

### Fragmented

Only part of the gene was recovered.

### Missing

Gene absent from the assembly.

### Duplicated

Multiple copies detected.

May indicate:

- polyploidy
- assembly duplication
- unresolved haplotypes

---

## Example BUSCO Output

| Category | Percentage |
|-----------|-----------|
| Complete | 96% |
| Duplicated | 2% |
| Fragmented | 1% |
| Missing | 1% |

This assembly would generally be considered highly complete.

---

# Merqury

Merqury evaluates assemblies using k-mers rather than reference genomes.

Advantages:

- reference-free
- consensus accuracy estimation
- completeness estimation

Useful when no reference genome exists.

---

# Read Mapping Back to the Assembly

One important validation step is aligning the original sequencing reads back to the assembled genome.

Common aligners:

- BWA
- Bowtie2
- minimap2

High mapping rates suggest that the assembly explains most sequencing reads.

Low mapping rates may indicate:

- contamination
- incomplete assembly
- incorrect assembly

---

# Module 21 – Assembly Polishing

Even high-quality assemblies contain sequencing errors.

Polishing improves consensus accuracy by correcting mismatches, insertions, and deletions.

---

## Short-read Polishing

Short reads have very high accuracy.

Programs:

- Pilon
- NextPolish

These tools align Illumina reads back to the assembly and correct errors.

---

## Long-read Polishing

Long-read assemblies often contain indel errors.

Programs include:

- Racon
- Medaka
- Arrow

Multiple polishing rounds are frequently recommended.

---

# Module 22 – Common Assembly Problems

## Fragmented Assembly

Possible causes:

- low coverage
- poor DNA quality
- excessive trimming
- sequencing errors

---

## Misassemblies

Incorrect joining of genomic regions.

May result from:

- repetitive DNA
- structural variation
- chimeric reads

---

## Collapsed Repeats

Nearly identical repeats may be assembled into a single copy,

making the assembly artificially shorter.

---

## Chimeric Contigs

Sequences originating from different genomic regions are incorrectly joined.

---

## Contamination

Foreign DNA may produce additional contigs.

Typical contaminants:

- bacteria
- fungi
- mitochondria
- chloroplasts
- laboratory contaminants

---

# Module 23 – Best Practices

High-quality genome assemblies generally follow these recommendations:

- Obtain high molecular weight DNA.
- Sequence at adequate depth.
- Perform quality control before assembly.
- Remove adapters and low-quality reads.
- Use an assembler appropriate for the sequencing platform.
- Evaluate assemblies using multiple metrics.
- Polish the assembly.
- Document software versions and parameters.
- Maintain reproducible workflows using Conda, Docker, or Snakemake.
- Archive raw data and final assemblies.

---

# Summary

This module described the complete computational workflow from raw sequencing reads to a polished genome assembly. We explored quality assessment, read trimming, error correction, assembly generation, assembly evaluation, polishing, and common troubleshooting strategies.

In **Part 4**, these concepts will be applied in a complete hands-on practical exercise. Participants will assemble a bacterial genome from Illumina paired-end sequencing data using **SPAdes**, evaluate the assembly with **QUAST** and **BUSCO**, map reads back to the assembly, interpret the results, and complete a guided analysis suitable for publication-quality reporting.
