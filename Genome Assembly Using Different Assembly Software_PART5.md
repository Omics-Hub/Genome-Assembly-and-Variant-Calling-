# Part 5 — Practical: Genome Assembly Using Different Assembly Software

---

# Overview

There is no single genome assembler that performs best for every sequencing technology or genome type. The choice of assembler depends on factors such as read length, sequencing accuracy, genome size, repeat content, ploidy, and available computational resources.

This section demonstrates how to assemble genomes using several widely used assemblers for:

- Illumina short reads
- PacBio HiFi reads
- Oxford Nanopore reads
- Hybrid (Illumina + Long Reads)

The objective is not only to learn how to run each assembler but also to understand **when** each assembler should be used.

---

# Learning Objectives

After completing this practical, students should be able to:

- Select an appropriate assembler
- Run multiple genome assemblers
- Compare assembly outputs
- Evaluate assembler performance
- Interpret differences among assemblies

---

# Directory Structure

```text
GenomeAssembly/

├── illumina/
│     sample_R1.fastq.gz
│     sample_R2.fastq.gz
│
├── nanopore/
│     nanopore.fastq.gz
│
├── pacbio/
│     hifi.fastq.gz
│
├── hybrid/
│     illumina_R1.fastq.gz
│     illumina_R2.fastq.gz
│     nanopore.fastq.gz
│
└── assemblies/
```

---

# Short Read Assemblers

---

# SPAdes

## Best For

- Illumina paired-end data
- Small bacterial genomes
- Small eukaryotic genomes

---

## Basic Assembly

```bash
spades.py \
-1 illumina/sample_R1.fastq.gz \
-2 illumina/sample_R2.fastq.gz \
-o assemblies/spades
```

---

## Specify Threads and Memory

```bash
spades.py \
-1 sample_R1.fastq.gz \
-2 sample_R2.fastq.gz \
-t 16 \
-m 64 \
-o assemblies/spades
```

---

## Assemble Using Specific k-mer Sizes

```bash
spades.py \
-k 21,33,55,77,99,127 \
-1 sample_R1.fastq.gz \
-2 sample_R2.fastq.gz \
-o assemblies/spades
```

---

## Output

```text
contigs.fasta

scaffolds.fasta

assembly_graph.fastg

spades.log
```

---

# ABySS

## Best For

- Large Illumina datasets
- Parallel computing
- Distributed memory clusters

---

## Single Machine Assembly

```bash
abyss-pe \
name=assembly \
k=96 \
in='sample_R1.fastq.gz sample_R2.fastq.gz'
```

---

## MPI Assembly

```bash
mpirun -np 16 \
abyss-pe \
name=assembly \
k=96 \
in='sample_R1.fastq.gz sample_R2.fastq.gz'
```

---

## Important Parameters

| Parameter | Meaning |
|------------|----------|
| k | k-mer size |
| np | CPU cores |
| name | Output prefix |

---

# Velvet

---

## Step 1

Create Hash Table

```bash
velveth velvet_output \
91 \
-shortPaired \
-fastq.gz \
-separate \
sample_R1.fastq.gz \
sample_R2.fastq.gz
```

---

## Step 2

Run Assembly

```bash
velvetg velvet_output
```

---

## Output

```text
contigs.fa

stats.txt

Graph2
```

---

# SOAPdenovo2

SOAPdenovo requires a configuration file.

Example

```text
max_rd_len=150

[LIB]
avg_ins=350
reverse_seq=0
asm_flags=3

q1=sample_R1.fastq.gz
q2=sample_R2.fastq.gz
```

---

Run

```bash
SOAPdenovo-127mer all \
-s config.txt \
-K 91 \
-o soap
```

---

# MEGAHIT

Although primarily developed for metagenomes,

MEGAHIT performs well on bacterial genomes.

```bash
megahit \
-1 sample_R1.fastq.gz \
-2 sample_R2.fastq.gz \
-o megahit
```

---

# SKESA

Designed for bacterial genome assembly.

```bash
skesa \
--fastq \
sample_R1.fastq.gz,sample_R2.fastq.gz \
--contigs_out \
assembly.fasta
```

---

# Long Read Assemblers

---

# Flye

## Best For

- Oxford Nanopore
- PacBio CLR

---

Nanopore

```bash
flye \
--nano-raw nanopore.fastq.gz \
--out-dir flye \
--threads 16
```

---

PacBio

```bash
flye \
--pacbio-raw pacbio.fastq.gz \
--out-dir flye
```

---

Genome Size

```bash
flye \
--nano-raw nanopore.fastq.gz \
--genome-size 5m \
--out-dir flye
```

---

Output

```text
assembly.fasta

assembly_graph.gfa

assembly_info.txt
```

---

# Canu

---

Nanopore

```bash
canu \
-p ecoli \
-d canu \
genomeSize=5m \
-nanopore nanopore.fastq.gz
```

---

PacBio

```bash
canu \
-p ecoli \
-d canu \
genomeSize=5m \
-pacbio pacbio.fastq.gz
```

---

PacBio HiFi

```bash
canu \
-p ecoli \
-d canu \
genomeSize=5m \
-pacbio-hifi hifi.fastq.gz
```

---

# Hifiasm

Designed specifically for PacBio HiFi sequencing.

```bash
hifiasm \
-o assembly \
-t16 \
hifi.fastq.gz
```

Convert

```bash
awk '/^S/{print ">"$2"\n"$3}' \
assembly.bp.p_ctg.gfa \
> assembly.fasta
```

---

# Raven

```bash
raven \
nanopore.fastq.gz \
> raven.fasta
```

---

# Shasta

```bash
shasta \
--input nanopore.fastq.gz
```

---

# Miniasm

Requires Minimap2.

---

Generate overlaps

```bash
minimap2 \
-x ava-ont \
nanopore.fastq.gz \
nanopore.fastq.gz \
> overlaps.paf
```

---

Run

```bash
miniasm \
-f nanopore.fastq.gz \
overlaps.paf \
> assembly.gfa
```

Convert

```bash
awk '/^S/{print ">"$2"\n"$3}' \
assembly.gfa \
> assembly.fasta
```

---

# Hybrid Assemblers

Hybrid assembly combines

- accurate Illumina reads

with

- long Nanopore or PacBio reads.

---

# Unicycler

Excellent bacterial genome assembler.

```bash
unicycler \
-1 illumina_R1.fastq.gz \
-2 illumina_R2.fastq.gz \
-l nanopore.fastq.gz \
-o unicycler
```

---

Output

```text
assembly.fasta

assembly.gfa

unicycler.log
```

---

# MaSuRCA

Create configuration file

```bash
masurca config.txt
```

Run

```bash
./assemble.sh
```

---

# hybridSPAdes

```bash
spades.py \
--careful \
-1 illumina_R1.fastq.gz \
-2 illumina_R2.fastq.gz \
--nanopore nanopore.fastq.gz \
-o hybrid
```

---

PacBio

```bash
spades.py \
--careful \
-1 illumina_R1.fastq.gz \
-2 illumina_R2.fastq.gz \
--pacbio pacbio.fastq.gz \
-o hybrid
```

---

# Assembly Evaluation

Evaluate every assembly.

```bash
quast.py \
assembly.fasta \
-o quast_results
```

---

BUSCO

```bash
busco \
-i assembly.fasta \
-l bacteria_odb10 \
-m genome \
-o busco_results
```

---

Read Mapping

```bash
bwa index assembly.fasta

bwa mem \
assembly.fasta \
sample_R1.fastq.gz \
sample_R2.fastq.gz \
> mapped.sam
```

---

Convert

```bash
samtools view \
-bS mapped.sam \
> mapped.bam

samtools sort \
mapped.bam \
-o mapped.sorted.bam

samtools index \
mapped.sorted.bam
```

---

Statistics

```bash
samtools flagstat \
mapped.sorted.bam
```

---

# Comparative Exercise

Assemble the same bacterial genome using

- SPAdes
- Velvet
- ABySS
- SOAPdenovo2
- MEGAHIT
- SKESA

Compare

- Number of contigs
- Largest contig
- N50
- L50
- GC%
- BUSCO completeness
- Mapping rate
- Runtime
- Peak memory usage

---

# Long Read Comparison

Assemble the same Nanopore dataset using

- Flye
- Canu
- Raven
- Shasta

Compare

- Contiguity
- Number of contigs
- Runtime
- Memory
- Consensus accuracy

---

# Hybrid Comparison

Assemble the same dataset using

- Unicycler
- hybridSPAdes
- MaSuRCA

Evaluate

- Genome completeness
- Circular chromosome recovery
- Number of gaps
- Accuracy
- Runtime

---

# Recommended Assembler by Sequencing Technology

| Sequencing Technology | Recommended Assembler |
|------------------------|----------------------|
| Illumina | SPAdes |
| Illumina (Large Genome) | ABySS |
| Illumina (Metagenome) | MEGAHIT |
| Illumina (Bacteria) | SKESA |
| PacBio HiFi | Hifiasm |
| PacBio CLR | Flye, Canu |
| Oxford Nanopore | Flye |
| Hybrid (Bacteria) | Unicycler |
| Hybrid (General) | MaSuRCA |
| Hybrid (Short + Long) | hybridSPAdes |

---

# Practical Challenge

You are provided with three datasets:

1. Illumina paired-end reads
2. Oxford Nanopore long reads
3. PacBio HiFi reads

For each dataset:

1. Choose an appropriate assembler.
2. Justify your choice.
3. Assemble the genome.
4. Evaluate the assembly using QUAST and BUSCO.
5. Compare assembly quality across sequencing technologies.
6. Present your findings in a short report discussing the trade-offs between contiguity, accuracy, computational requirements, and suitability for downstream analyses.

This exercise reinforces that assembler selection should be guided by sequencing technology, genome characteristics, and the scientific objectives of the project rather than by a one-size-fits-all approach.
