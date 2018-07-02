# DE-kupl Annotation

DE-kupl annotation is part of the DE-kupl package, and performs annotations of DE contigs identified by DE-kupl.

## Usage

### Creating the index

To use DEkupl Annotation, you first need to build an index directory, that will prepare all
the file for annotation.

Required file are a genome as a multi-FASTA and annotations in a GFF3 format. Both
of this file can be downloaded from [Ensembl](https://www.ensembl.org/info/data/ftp/index.html).

Input files can be gzipped.

The following command with create the index under the `index_dir` directory.

```
dkpl index -g genome.fasta -a annotations.gff3 -i index_dir
```

### Annotating contigs

Once an index has been constructed you can annotate contigs from a DEkupl-run. 
The minimal input to run DEkupl-annotation is the contigs file 
(merged-diff-counts.tsv.gz) produced by DEkupl-run.

```
dkpl annot -i index_dir merged-diff-counts.tsv.gz
```

Output files will be placed under the `DEkupl_annotation` directory unless you specify
another output directory with `-o` option.

Extra file can be supplied to complete the annotation process (see ontolgy table) :

* differentially expressed genes (`--deg`) for DEG analyzer
* nomarlized gene counts (`--norm-gene-counts`) and sample conditions (`--sample-conditions`) for Switches analyzer.

### Tutorial & toys

Toy files are available with this repository to test dkpl-annot.

```
dkpl index -g toy/references/GRCh38-chr22.fa.gz -a toy/references/GRCh38-chr22.gff.gz -i test_index
dkpl annot -i test_index --deg toy/dkpl-run/DEGs.tsv.gz --norm-gene-counts toy/dkpl-run/normalized_counts.tsv --sample-conditions toy/dkpl-run/sample_conditions_full.tsv toy/dkpl-run/merged-diff-counts.tsv.gz
```

### Installation

### Required dependencies

* bash (version >= 4.3.46)
* R (version >= version 3.2.3) with libraries DESeq2
* GSNAP (version >= 2016-11-07)
* samtools (version >= 1.3)

### Install from the sources

#### Install Dependancies

Dependencies are cpan-minus (aka cpanm) and Dist::Zilla :

```
apt-get install cpanminus libdist-zilla-perl
```

#### Global install

The following command, will clone the repository and install dkpl-annot globaly with dzil and cpanm.

```
git clone https://github.com/Transipedia/dekupl-annotation.git && cd dekupl-annotation
dzil install --install-command 'cpanm .'
```

#### Local install

For local install you need to use the `-l LOCAL_DIR` parameter of cpanm.
Then you need to make sure that the Perl library that have been installed locally
are available to the path using the `PERL5LIB` environnement variable. 

For example :

```
dzil install --install-command 'cpanm -l $HOME/.local .'
export PERL5LIB=$HOME/.local/lib:$PERL5LIB
```

## Output files

- Table `DiffContigsInfos.tsv`, summarizing for each contig, its location on the genome (if it's aligned), the neighborhood, the sequence alignment informations, and the differential expression informations.

- BED file `diff_contigs.bed` for the visualization ; it contains useful informations from the summarization table. BED12 file of aligned contigs with GSNAP/Blast, with strand-specific color (red : strand + ; blue : strand -).

- Table `ContigsPerLoci.tsv`, contigs grouped by loci (genic, antisense, intergenic, unmapped). The locus ID for a genic/antisense locus is the Ensembl ID followed by the strand (separated by "&"). For an intergenic locus, we have the concatenation of : chromosome, strand, 5'-gene, 3'-gene (separated by "&").

## Ontology

| Term                    | Type  | Analyzer    | File Source                    | Description                                                                                        |
| ----------------------- | ----- | ----------- | ------------------------------ | -------------------------------------------------------------------------------------------------- |
| tag                     | Str   | Contigs     | contigs                        | kmer of reference                                                                                  |
| nb_merged_kmers         | Int   | Contigs     | contigs                        | Number of k-mer merged to produce the contig                                                       |
| contig                  | Str   | Contigs     | contigs                        | Sequence of the contig                                                                             |
| contig_size             | Int   | Contigs     | contigs                        | Size (in nucleotides) of the contig                                                                |
| pvalue                  | Float | Contigs     | contigs                        | Pvalue statistics from dekupl-run                                                                  |
| meanA                   | Float | Contigs     | contigs                        | Average counts for condition A                                                                     |
| meanB                   | Float | Contigs     | contigs                        | Average counts for condition B                                                                     |
| log2FC                  | Float | Contigs     | contigs                        | Log2 Fold-Change between samples in condition A vs B                                               |
| is_mapped               | Bool  | Bam         | bam                            | Is the contig mapped to at least one location                                                      |
| line_in_sam             | Int   | Bam         | bam                            | Line number in the SAM/BAM                                                                         |
| chromosome              | Str   | Bam         | bam                            | Chromosome                                                                                         |
| start                   | Int   | Bam         | bam                            | Begining of the alignment on the reference                                                         |
| end                     | Int   | Bam         | bam                            | End of the alignment on the reference                                                              |
| strand                  | Char  | Bam         | bam                            | Strand of the alignment (+/-). set to NA is unstranded data.                                       |
| cigar                   | Str   | Bam         | bam                            | CIGAR string from the SAM alignment.                                                               |
| nb_insertion            | Int   | Bam         | bam                            | Number of insertions in the alignment (from cigar)                                                 |
| nb_deletion             | Int   | Bam         | bam                            | Number of deletions in the alignment (from cigar)                                                  |
| nb_splice               | Int   | Bam         | bam                            | Number of splices in the alignment (from cigar)                                                    |
| clipped_3p              | Bool  | Bam         | bam                            | Number of clipped bases (soft/hard) from 3prim contig                                              |
| clipped_5p              | Bool  | Bam         | bam                            | Number of clipped bases (soft/hard) from 5prim contig                                              |
| is_clipped_3p           | Bool  | Bam         | bam                            | True if contig's 3prim is soft/hard clipped (from cigar)                                           |
| is_clipped_5p           | Bool  | Bam         | bam                            | True if contig's 5prim is soft/hard clipped (from cigar)                                           |
| query_cover             | Float | Bam         | bam                            | Fraction of the query that have been aligned to the reference                                      |
| alignment_identity      | Float | Bam         | bam                            | Fraction of exact match over the query alignment length (splices do not count)                     |
| nb_hit                  | Int   | Bam         | bam                            | Number of alignment given for the contig (NH field)                                                |
| nb_mismatches           | Int   | Bam         | bam                            | Number of mismatches in the alignment (NM field)                                                   |
| gene_id                 | Str   | Annotations | gff                            | Overlapping gene ID (from GFF ID field). On the same strand if --stranded option.                  |
| gene_symbol             | Str   | Annotations | gff                            | Overlapping gene symbol (from GFF Name field). On the same strand if --stranded option.            |
| gene_strand             | Char  | Annotations | gff                            | Overlapping gene strand (+/-). On the same strand if --stranded option.                            |
| gene_biotype            | Str   | Annotations | gff                            | Overlapping gene biotype (from GFF biotype field). On the same strand if --stranded option.        |
| as_gene_id              | Str   | Annotations | gff                            | Overlapping antisense gene ID (from GFF ID field). Only defined if --stranded option.              |
| as_gene_symbol          | Str   | Annotations | gff                            | Overlapping antisense gene symbol (from GFF Name field). Only defined if --stranded option.        |
| as_gene_strand          | Char  | Annotations | gff                            | Overlapping antisense gene strand (+/-). Only defined if --stranded option.                        |
| as_gene_biotype         | Str   | Annotations | gff                            | Overlapping antisense gene biotype (from GFF biotype field). Only defined if --stranded option.    |
| upstream_gene_id        | Str   | Annotations | gff                            | Nearest downstream gene ID. Same strand if --stranded option, downstream otherwise.                |
| upstream_gene_symbol    | Str   | Annotations | gff                            | Nearest downstream gene symbol. Same strand if --stranded option, downstream otherwise.            |
| upstream_gene_strand    | Str   | Annotations | gff                            | Nearest downstream gene strand (+/-).                                                              |
| upstream_gene_biotype   | Str   | Annotations | gff                            | Nearest downstream gene biotype.                                                                   |
| downstream_gene_id      | Str   | Annotations | gff                            | Nearest upstream gene ID. Same strand if --stranded option, upstream otherwise.                    |
| downstream_gene_symbol  | Str   | Annotations | gff                            | Nearest upstream gene symbol. Same strand if --stranded option, upstream otherwise.                |
| downstream_gene_strand  | Str   | Annotations | gff                            | Nearest upstream gene strand (+/-).                                                                |
| downstream_gene_biotype | Str   | Annotations | gff                            | Nearest upstream gene biotype.                                                                     |
| exonic                  | Bool  | Annotations | gff                            | Overlap between an exon and the contig. Same strand if --stranded option, both strand otherwise.   |
| intronic                | Bool  | Annotations | gff                            | Overlap between an intron and the contig. Same strand if --stranded option, both strand otherwise. |
| gene_is_diff            | Bool  | DEG         | DEGs                           | The main annotated gene (gene_id ontology) is differantially expressed                             |
| du_pvalue               | Float | Switches    | DEGs                           | P-value for contig differential usage                                                              |
| du_stats                | Float | Switches    | gene_counts, sample_conditions | Differential usage statistic                                                                       |

## Dev environnement

For developpement, `git clone` this repository and enter the project folder.

Then, add the local dir to the PERL5LIB env var to use the modules locally.

```
export PERL5LIB=$PWD/lib:$PERL5LIB
```

You are ready to code!