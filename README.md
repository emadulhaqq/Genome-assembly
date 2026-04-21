VGP-Style Genome Assembly Pipeline

Diploid genome assembly of Saccharomyces cerevisiae using PacBio HiFi, Hi-C, and Bionano optical mapping

Galaxy • HiFi • Hi-C • Bionano • VGP Workflow

Overview

This project presents a diploid genome assembly workflow inspired by the Vertebrate Genomes Project (VGP), implemented within the Galaxy platform. Starting from raw PacBio HiFi sequencing reads, the pipeline produces chromosome-level, haplotype-resolved assemblies.

The workflow integrates:

HiFi-based contig construction
Optical map scaffolding (Bionano)
Hi-C–based chromatin scaffolding

The model organism used is Saccharomyces cerevisiae strain S288C, with an expected haploid genome size of approximately 11.7 Mb.

Dataset Description
Data Type	Format	Source	Purpose
PacBio HiFi reads (3 files)	.fasta	Zenodo 6098306	Contig generation
Illumina Hi-C reads (paired)	.fastqsanger.gz	Zenodo 5550653	Phasing and scaffolding
Bionano optical maps	.cmap	Zenodo 5887339	Structural scaffolding

Recommended coverage:

HiFi: ≥30×
Hi-C: ~60× (for diploid assemblies)
Pipeline Workflow

The assembly process follows six major stages:

Raw HiFi Reads
      ↓
[1] Adapter Removal (Cutadapt)
      ↓
[2] Genome Profiling (Meryl → GenomeScope2)
      ↓
[3] Hi-C Assisted Assembly (hifiasm)
      ├── Haplotype 1 contigs
      └── Haplotype 2 contigs
            ↓
      [4] Quality Assessment
            ↓
      [5a] Bionano Scaffolding
            ↓
      [5b] Hi-C Scaffolding (YaHS)
            ↓
      Chromosome-scale Assembly
            ↓
      [6] Contact Map Validation
Tools and Outputs
Stage 1 — Adapter Filtering

Tool: Cutadapt v4.4

HiFi reads are screened for internal adapter sequences. Because long-read adapters can occur anywhere within reads, any read containing adapters is completely discarded rather than trimmed.

Key parameters:

Error tolerance: 0.1
Minimum overlap: 35 bp
Reverse complement detection: enabled
Action: discard read

Input: HiFi read collection
Output: Cleaned HiFi reads

Stage 2 — Genome Profiling
2a. K-mer Counting

Tool: Meryl v1.3

Reads are broken into 31-mers and counted. Separate databases are generated per input file and then merged into a unified dataset.

Steps:

Count k-mers (k=31)
Merge databases (union-sum)
Generate histogram

Outputs:

Combined k-mer database
Histogram for downstream analysis
2b. Genome Characterization

Tool: GenomeScope2

Uses the k-mer histogram to estimate genome features via statistical modeling.

Parameters:

Ploidy: 2
k-mer size: 31

Estimated properties:

Genome size: ~11.7 Mb
Heterozygosity: ~0.576%
Coverage peaks: ~25× (haploid), ~50× (diploid)
Model fit: >93%

These estimates guide later steps in the pipeline.

Stage 3 — Hi-C Guided Assembly

Tool: hifiasm v0.19.8

Generates contigs from HiFi reads and uses Hi-C data to separate them into two haplotypes.

Outputs:

Hap1 assembly graph (GFA)
Hap2 assembly graph (GFA)
GFA Conversion

Tool: gfastats v1.3.6

Transforms GFA assembly graphs into FASTA sequences for downstream processing.

Stage 4 — Assembly Evaluation
4a. Assembly Statistics

Tool: gfastats

Computes standard metrics such as:

Total length
Contig count
Largest contig
N50 / NG50
GC content

Approximate results:

Hap1: ~11.3 Mb (16 contigs)
Hap2: ~12.2 Mb (17 contigs)
4b. Gene Completeness

Tool: BUSCO v5.5.0

Assesses completeness using conserved orthologs from the Saccharomycetes lineage.

4c. K-mer Validation

Tool: Merqury

Evaluates:

Assembly completeness
Consensus accuracy (QV)
Phasing quality

Key observations:

Expected coverage peaks confirmed
Haplotypes properly separated
Shared k-mers represent homozygous regions
Stage 5a — Bionano Scaffolding

Tool: Bionano Hybrid Scaffold v3.7.0

Aligns optical maps to contigs to:

Order and orient sequences
Detect misassemblies
Estimate gap sizes

Outputs include scaffolded and unscaffolded contigs, which are merged into a final assembly.

Stage 5b — Hi-C Scaffolding
Read Alignment

Tool: BWA-MEM2

Hi-C reads are mapped independently (not paired) due to highly variable insert sizes.

Filtering

Tool: Arima filter

Combines and cleans alignments, handling chimeric Hi-C reads.

Initial Contact Map

Tools: PretextMap + Pretext Snapshot

Generates a pre-scaffolding interaction map for comparison.

Final Scaffolding

Tool: YaHS v1.2a.2

Uses Hi-C contact frequencies to build chromosome-scale scaffolds.

Parameter:

Restriction site: CTTAAG
Stage 6 — Final Validation

Hi-C reads are remapped to the final scaffolds and a new contact map is generated to verify correct chromosome structure.

Assembly Statistics
Contig-Level (Post-hifiasm)
Metric	Hap1	Hap2
Contigs	16	17
Total length	11,305 kb	12,161 kb
Largest contig	~1.53 Mb	~1.53 Mb
N50	~922 kb	~923 kb
GC content	~38%	~38%
Final Assembly vs Reference
Metric	Final Assembly	Reference
Length	12,161 kb	12,157 kb
Scaffolds	17	17
N50	922 kb	924 kb
GC%	~38.18	~38.00

The close agreement with the reference genome indicates a high-quality, near-complete assembly.

Glossary
Contig: Continuous DNA sequence without gaps
Scaffold: Linked contigs separated by gaps
Haplotype: One copy of a chromosome set
Phasing: Assigning sequences to specific haplotypes
N50: Length covering 50% of the genome
QV: Quality score indicating error rate
BUSCO: Metric for gene completeness
