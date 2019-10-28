---
title: Bioinformatics Tools for Microbiome Analysis
last_modified_at: 2019-10-02
primary_reviewers: sminot
---

Data generated from microbiome experiments tends to require a distinct analytical
approach, one that takes into account the many different organisms which may be
present within a single biological sample. The considerations of sample preparation
also tend to be fairly specific to the microbiome, e.g. 16S rRNA gene amplicon
sequencing tends to be used only for the purpose of performing taxonomic identification
for a mixture of bacteria.

The Microbiome Research Initiative was started at Fred Hutch to support and provide
a community for scientists researching the microbiome, and part of that effort
includes some work to develop a relatively standardized toolkit of bioinformatic
analysis tools which can be used by multiple investigators across the center. This
page is provided to host a description of analysis tools that are available, and
will be updated as more are made available. 

The tools below happen to use Nextflow as a system for running reproducible and
portable analytical workflows. 
See [this documentation](https://sciwiki.fredhutch.org/compdemos/nextflow/) 
for more details on running Nextflow at Fred Hutch, as well as 
[the docs](https://www.nextflow.io/) for more details on Nextflow itself.

For any questions on the tools presented here, either in trying to get them running
on your data, or if you would like access to additional functionality (or would like
to offer your own utilities), please contact Sam Minot (sminot@fredhutch.org).

## Ribosomal 16S Amplicon Analysis

One of the primary tools used by microbiome researchers to detect organisms present
in a microbiome sample is 16S amplicon sequencing. This technique takes advantage of
a highly conserved gene present in bacterial genomes which can be targeted by PCR
with well-designed primers, and then processed with high-thoughput sequencing. The
analysis of 16S datasets is a highly-developed analytical process with a long series
of steps, and therefore is highly amenable to automation with a formalized workflow.

Dr. Jonathan Golob is a physician-scientist at the University of Michigan, and he
has developed a highly accurate and effective workflow for analyzing 16S datasets.
This workflow uses `dada2` to identify exact sequence varients and ultimately 
performs taxonomic identification using a phylogenetic approach which is very much
on the cutting edge of the 16S field (using `pplacer` for phylogenetic assignment).

GitHub repository: [jgolob/maliampi/](https://github.com/jgolob/maliampi/)

### Example Run:

All input files are specified in a single file (`manifest.csv`) with columns used to
identify the `specimen`, `read__1`, and `read__2`. You may also include a `batch` 
column to indicate which samples were processed for sequencing together. 

```
set -e

ml nextflow

REF_FOLDER="s3://fh-ctr-public-reference-data/tool_specific_data/maliampi/ya16sdb_20190821/dedup/1200bp/named/filtered"

nextflow \
    run \
    jgolob/maliampi \
    --manifest manifest.csv \
    --repo_fasta $REF_FOLDER/seqs.fasta \
    --repo_si $REF_FOLDER/seq_info.csv \
    --email <EMAIL> \
    --output <OUTPUT_FOLDER> \
    -w <WORK_DIR> \
    -resume
```

See the MaLiAmPi [documentation](https://github.com/jgolob/maliampi/) for more
details on running the tool and interpreting the output. 

## Microbial Genome Assembly

One common task in microbiology is sequencing the genome of microbial isolates.
With the advent of single-molecule long-read PacBio sequencing, it is now possible
to routinely generate fully closed genome assemblies. To automate this process, we
have implemented the UniCycler assembler in an easy-to-use workflow. This assembler
provides the advantage of accommodating both short- and long-reads, performing
hybrid assembly when both are provided. 

GitHub Repository: [FredHutch/unicycler-nf/](https://github.com/FredHutch/unicycler-nf/)

```
Usage:

nextflow run fredhutch/unicycler-nf <ARGUMENTS>

Arguments:
  --sample_sheet       CSV file listing samples to analyze
  --output_folder      Folder to place outputs

Options:
  --short_reads        Sample sheet contains short read data (`short_R1` and `short_R2`)
  --long_reads         Sample sheet contains long read data (`long_reads`)
  --min_fasta_length   Minimum contig length (default: 100)
  --help               Display this message

Sample Sheet:
  The sample_sheet is a CSV with a header indicating which samples correspond to which files.
  The file must contain the column `name`, and `long_reads`, `short_R1`, `short_R2` as appropriate.
```

## Microbial Pan-Genome Analysis

Microbial researchers often need to compare multiple genomes in order to identify
similarities and differences. The best tool available in the community for this analysis
is the [anvi'o software suite](http://merenlab.org/software/anvio/) developed by the 
Meren Lab at the University of Chicago. The anvi'o software does _many things_, but
we wanted to provide an easy point of entry with a workflow that imports a set of 
bacterial genomes into the anvi'o database format, and then launches a graphical viewer
which allows the user to explore their pan-genome collection. 

The guidance and instructions for running this tool can be found on the
[FredHutch/nf-anvio-pangenome GitHub repository](https://github.com/FredHutch/nf-anvio-pangenome)


## Microbial RNAseq

One application of whole-genome shotgun sequencing (WGS) for microbiome research
is the analysis of microbial mixtures on the basis of what microbes are present
(DNA) or are transcriptionally active (RNA). To address this analytical need, we
developed an analysis tool which takes a set of WGS input data and aligns it against
a set of whole microbial genomes. With the orientation towards RNAseq, the tool
takes a parameter `--min_cov_pct` which limits the analysis to those organisms
which have greater than the specified level of coverage across the rRNA genes found
in their genomes. Using just those organisms, the pipeline will then measure the
depth of sequencing across all genes for all organisms across all samples, and provide
those results to the user in the form of a set of CSV files.

GitHub repository: [FredHutch/microbial-rnaseq](https://github.com/FredHutch/microbial-rnaseq)

### Example Run:

All input files are specified in a single file `batchfile.csv` which notes each
sample with the columns `name` and `fastq` (`fastq1` and `fastq2` for paired-end
data).

```
nextflow \
    run \
    fredhutch/microbial-rnaseq \
    --batchfile batchfile.csv \
    --host_genome "s3://fh-ctr-public-reference-data/tool_specific_data/microbial-rnaseq/2019-06-10/Homo_sapiens_assembly38.fasta.tar" \
    --database_folder "s3://fh-ctr-public-reference-data/tool_specific_data/microbial-rnaseq/2019-07-03/" \
    --database_prefix 2019-07-03-rnaseq-database \
    --min_cov_pct 90 \
    --output_folder results/ \
    --output_prefix 2019-05-08-test \
    -work-dir work/ \
    -resume
```

## Viral Metagenomics

When studying viruses in the human microbiome, one fruitful approach can simply be to
perform whole-genome shotgun sequencing (WGS) and then align all reads against a set of
viruses from some reference database. This is an extremely parallelizable process, and 
therefore benefits heavily from execution systems like Nextflow which can distribute
tasks to cloud computing services (like AWS). 

This utility takes a set of input files (FASTQ format) and aligns them all against a set
of viral genomes (specified by NCBI `accession` in a provided CSV file).

GitHub repository: [FredHutch/nf-viral-metagenomics](https://github.com/FredHutch/nf-viral-metagenomics)

### Example Run:

`<INPUT_DIRECTRY>` contains the set of FASTQ files to analyze
`<OUTPUT_DIRECTRY>` is the location where all outputs will be placed
`<VIRAL_GENOME_CSV>` is a CSV with a column `accession` containing the set of viral genomes to align against (as NCBI Nucleotide accessions)
`<NAME_OF_OUTPUT_CSV>` is the name of the output file to be placed in the `<OUTPUT_DIRECTORY>`

```
nextflow \
    run \
        FredHutch/nf-viral-metagenomics \
        --input_directory <INPUT_DIRECTORY> \
        --output_directory <OUTPUT_DIRECTORY> \
        --viral_genome_csv <VIRAL_GENOME_CSV> \
        --output_csv <NAME_OF_OUTPUT_CSV>
```
