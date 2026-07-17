# Module 8 – Computational Foundations of Genome Assembly

In the previous section, we discussed how sequencing technologies generate millions of DNA reads from fragmented genomic DNA. The next challenge is reconstructing the original genome from these reads.

Genome assembly is fundamentally a computational problem. The assembler must determine which reads originated from adjacent genomic regions and reconstruct the most likely genome sequence while accounting for sequencing errors, repetitive DNA, and uneven sequencing coverage.

Unlike a traditional jigsaw puzzle, genome assembly is considerably more difficult because:

- The final genome sequence is unknown.
- Many pieces (reads) are nearly identical.
- Some pieces contain sequencing errors.
- Large portions of the genome may consist of repetitive DNA.
- Certain regions may have little or no sequencing coverage.

Modern genome assemblers solve these problems using graph-based algorithms.

---

# Learning Objectives

By the end of this module, students should be able to:

- Explain how genome assembly algorithms reconstruct genomes.
- Understand the role of sequence overlap.
- Define k-mers and describe their importance.
- Explain Overlap-Layout-Consensus (OLC) assembly.
- Explain De Bruijn Graph assembly.
- Compare OLC and De Bruijn Graph approaches.
- Understand graph simplification.
- Select an appropriate assembler for different sequencing technologies.

---

# Module 9 – Sequence Overlap

The simplest idea behind genome assembly is that DNA fragments originating from neighboring genomic regions should share identical sequences at their ends.

Consider three sequencing reads:

```text
Read 1

ATCGATCG

Read 2

     GATCGAAA

Read 3

          CGAAATTT
```

The assembler identifies overlapping regions:

```text
ATCGATCG
     GATCGAAA
          CGAAATTT
```

These overlaps allow reconstruction of a longer sequence:

```text
ATCGATCGAAATTT
```

This principle forms the basis of nearly every genome assembly algorithm.

---

## Exact vs Approximate Overlaps

Perfect overlaps are uncommon because sequencing errors introduce mismatches.

Example:

```text
Read 1

ATCGATCG

Read 2

GATCAAAA
```

The assembler must determine whether the mismatch is:

- a sequencing error
- a true biological mutation
- an overlap between unrelated regions

Assembly algorithms therefore use alignment scores rather than requiring perfect matches.

---

# Module 10 – K-mers

A **k-mer** is a DNA sequence of length **k**.

If

```text
Read

ATCGAT
```

and

k = 3

the read is decomposed into

```text
ATC

TCG

CGA

GAT
```

Each k-mer overlaps the next by **k−1** bases.

---

## Why are k-mers Important?

Comparing every sequencing read against every other read is computationally expensive.

For a dataset containing one billion reads,

every read would need to be compared with every other read.

This would require approximately

```
10^18

comparisons
```

which is computationally impractical.

Instead,

modern assemblers compare k-mers.

Because k-mers are shorter and repeated many times, efficient indexing methods (such as hash tables and Bloom filters) allow rapid lookup.

---

## Choosing the Value of k

Choosing an appropriate k-mer size is one of the most important decisions in short-read assembly.

### Small k

Advantages

- Connects reads easily
- Better for low coverage
- Handles sequencing errors

Disadvantages

- Merges repetitive regions
- Produces tangled graphs

---

### Large k

Advantages

- Resolves repeats
- More specific overlaps

Disadvantages

- Requires higher coverage
- Sensitive to sequencing errors

---

Example

Suppose the genome contains

```text
AAAAAAAAAACCCCC
```

If

k = 3

many identical

```
AAA

AAA

AAA
```

k-mers occur,

making repeat resolution difficult.

Increasing k reduces ambiguity.

---

## Multi-k Assembly

Modern assemblers such as SPAdes do not rely on a single k-mer size.

Instead,

multiple k values are used,

for example

```
21

33

55

77

99

127
```

Small k values recover low-coverage regions,

while larger k values improve repeat resolution.

Combining multiple graphs often produces superior assemblies.

---

# Module 11 – Assembly Algorithms

Genome assembly algorithms fall into two major categories.

1. Overlap-Layout-Consensus (OLC)

2. De Bruijn Graph (DBG)

Both reconstruct genomes,

but they differ in how sequence relationships are represented.

---

# Overlap-Layout-Consensus (OLC)

OLC was developed before high-throughput sequencing and remains the preferred strategy for long-read sequencing.

The workflow consists of three stages.

---

## Step 1 — Overlap

Every read is compared with every other read.

Example

```text
Read A

ATCGATCG

Read B

GATCGAAA

Read C

CGAAATTT
```

Overlap graph

```text
Read A

↓

Read B

↓

Read C
```

Each edge represents an overlap.

---

## Step 2 — Layout

The assembler determines the most likely order of reads.

```text
Read A

↓

Read B

↓

Read C
```

---

## Step 3 — Consensus

Because multiple reads cover each genomic position,

the assembler computes the consensus nucleotide.

Example

```text
Read 1

ATCGATCG

Read 2

ATCGATCA

Read 3

ATCGATCG
```

Consensus

```text
ATCGATCG
```

Errors are corrected because most reads support the correct nucleotide.

---

## Advantages of OLC

- Excellent for long reads
- Handles variable read lengths
- Robust for complex genomes

---

## Limitations

- Computationally expensive
- Pairwise comparison of every read
- Poor scalability for billions of Illumina reads

---

## Common OLC Assemblers

| Assembler | Sequencing Platform |
|------------|--------------------|
| Canu | PacBio, Nanopore |
| Flye | Nanopore |
| Miniasm | Long reads |
| Raven | Long reads |

---

# De Bruijn Graph Assembly

De Bruijn Graph assembly revolutionized short-read assembly by replacing read-to-read comparisons with k-mer relationships.

Instead of comparing millions of reads,

each read is decomposed into k-mers.

Example

Read

```text
ATCGAT
```

k = 3

```text
ATC

TCG

CGA

GAT
```

Graph

```text
ATC

↓

TCG

↓

CGA

↓

GAT
```

The genome corresponds to a path through the graph.

---

## Why De Bruijn Graphs Are Efficient

Suppose we sequence

100 million reads.

OLC compares

```
100 million

×

100 million
```

possible read pairs.

De Bruijn Graphs instead compare shared k-mers,

dramatically reducing computational complexity.

---

## Graph Construction

Every read contributes overlapping k-mers.

Example

```text
Read

ATCGATCG
```

Produces

```text
ATC

↓

TCG

↓

CGA

↓

GAT

↓

ATC

↓

TCG
```

Multiple identical k-mers merge into a single graph.

---

## Traversing the Graph

Once constructed,

the assembler searches for a valid path through the graph.

The reconstructed path represents the assembled genome.

---

# Graph Simplification

Raw assembly graphs are extremely complex.

Assemblers simplify graphs before constructing contigs.

---

## Tips

Tips are short branches usually caused by sequencing errors.

```text
───────┐

       │

───────┴────────
```

Tips are removed because they are poorly supported.

---

## Bubbles

Bubbles occur when two alternative paths connect the same nodes.

```text
───────┐

       │

───────┘
```

Causes include

- sequencing errors

- heterozygosity

- true polymorphisms

Assemblers evaluate coverage and sequence similarity to determine which path to retain.

---

## Repeats

Repeats produce highly connected graph structures.

Example

```text
Genome

AAAAAA

CCCCCC

AAAAAA
```

The repeated

```
AAAAAA
```

creates ambiguous graph paths.

Repeats are one of the largest obstacles in genome assembly.

---

## Dead Ends

Dead ends occur when insufficient sequencing coverage prevents continuation of the graph.

Possible causes

- low sequencing depth

- poor-quality reads

- contamination

---

# String Graph Assembly

String graphs are an optimized version of OLC.

Instead of storing every overlap,

redundant overlaps are removed.

Advantages

- Smaller graph
- Reduced memory
- Better scalability

Assemblers

- Fermi
- SGA

---

# Comparing Assembly Algorithms

| Feature | OLC | De Bruijn Graph |
|----------|-----|----------------|
| Read Length | Long | Short |
| Memory Usage | High | Lower |
| Pairwise Comparisons | Yes | No |
| Scalability | Moderate | Excellent |
| Best For | PacBio, ONT | Illumina |

---

# Module 12 – Choosing an Assembler

Different sequencing technologies require different assembly strategies.

## Illumina

Recommended

- SPAdes
- Velvet
- ABySS
- SOAPdenovo2
- MEGAHIT

---

## PacBio HiFi

Recommended

- Hifiasm
- HiCanu

---

## Oxford Nanopore

Recommended

- Flye
- Raven
- Shasta

---

## Hybrid Sequencing

Recommended

- Unicycler
- MaSuRCA
- hybridSPAdes

Hybrid assembly combines accurate Illumina reads with long PacBio or Nanopore reads to produce assemblies that are both contiguous and accurate.

---

# Module 13 – Factors Affecting Assembly Quality

Genome assembly quality is influenced by several biological and technical factors.

## Repetitive DNA

Repeats longer than the sequencing read cannot be uniquely resolved using short reads.

Examples include:

- Transposable elements
- Tandem repeats
- Satellite DNA
- Segmental duplications

Long-read sequencing helps bridge these regions.

---

## Sequencing Errors

Errors introduce false k-mers and incorrect graph branches, increasing fragmentation.

Error correction and read trimming reduce their impact.

---

## GC Bias

Regions with extremely high or low GC content may be underrepresented due to PCR amplification and sequencing biases.

This can result in gaps or uneven coverage.

---

## Heterozygosity

Diploid organisms contain two homologous copies of each chromosome.

Highly heterozygous genomes may produce alternative paths in the assembly graph, increasing duplication and fragmentation.

---

## Polyploidy

Polyploid organisms possess more than two copies of each chromosome.

Closely related chromosome copies complicate assembly because they are difficult to distinguish.

---

## Contamination

Contaminating DNA from bacteria, fungi, viruses, or laboratory sources can introduce foreign sequences into the assembly.

Routine contamination screening is therefore an important quality-control step.

---

# Summary

In this module, we explored the computational principles behind genome assembly. We introduced sequence overlaps, k-mers, graph-based assembly methods, and the major algorithms used to reconstruct genomes from sequencing reads. We also examined graph simplification strategies and discussed how biological features such as repetitive DNA, heterozygosity, and contamination influence assembly quality.

In **Part 3**, we will build on these concepts by covering the complete genome assembly workflow, including quality control, read trimming, de novo assembly with SPAdes, assembly evaluation using QUAST and BUSCO, assembly polishing, troubleshooting, and best practices for producing high-quality genome assemblies.
