# Tutorial: Genome assembly

## Background

Genome assembly reconstructs longer DNA sequences (contigs) from many short
sequencing reads. In this exercise, you will download paired-end reads from the
NCBI Sequence Read Archive (SRA), check and clean them, assemble them with
SPAdes, and assess the assembly.

We will perform this exercise on a sample of *Schizosaccharomyces pombe*,
'fission yeast', which has a relatively small genome (~12Mb). We will download
short-reads uploaded for this species that we can assemble the genome from, you
can find info about the sequencing data at
[NCBI's SRA record](https://www.ncbi.nlm.nih.gov/sra/?term=ERR17038703) or 
[ENA's run record](https://www.ebi.ac.uk/ena/browser/view/ERR17038703) for the
sample.

The main steps we will perform are:

1. Download the reads
2. Clean up the reads, remove PCR duplicates, and review the quality control stats
3. Assemble the cleaned up reads
4. Assess the assembly quality.

While assembling the whole genome this way should also assemble the nuclear and
mitochondrial genome at the same time, we often can get a much better assembly
of that using a dedicated organelle assembler, so we will follow up by repeating
3 & 4 with a mitochondrial genome assembler.

## Downloading sequencing reads with SRA toolkit

> Remember to change to the working directory (`cd ~/phylogenomics_course`) and run
> `conda activate phylo` to activate your conda environment before running commands!

Set up a folder to download the raw reads to:

```bash
mkdir -p data/raw_reads
```

Set the accession and download its SRA archive using the `fasterq-dump` command from
SRA toolkit:

```bash
ACCESSION=ERR17038703

fasterq-dump $ACCESSION \
    --split-files \
    --threads 4 \
    --outdir data/raw_reads
```

The files are not compressed, so they take up quite a bit of space. If you're downloading
many large sets of sequencing reads, consider compressing them:

```bash
gzip data/raw_reads/${ACCESSION}_1.fastq data/raw_reads/${ACCESSION}_2.fastq
```

You should now have two files, you can verify by listing them:

```text
ls -lh data/raw_reads/ERR17038703_1.fastq.gz data/raw_reads/ERR17038703_2.fastq.gz
```

The `_1` and `_2` files contain the two reads from each pair.

## Trimming and deduplicate removal with fastp

fastp detects adapters, trims low-quality read ends, filters reads, and removes
duplicate read pairs. It also produces an HTML report with statistics before and
after trimming, so is a good way to process the reads and do QC all at once:

```bash
mkdir -p outputs/trimmed_reads
mkdir -p outputs/qc/trimming

fastp \
    -i data/raw_reads/${ACCESSION}_1.fastq.gz \
    -I data/raw_reads/${ACCESSION}_2.fastq.gz \
    -o outputs/trimmed_reads/${ACCESSION}.R1.trimmed.fastq.gz \
    -O outputs/trimmed_reads/${ACCESSION}.R2.trimmed.fastq.gz \
    --dedup \
    --html outputs/qc/trimming/${ACCESSION}_fastp_report.html \
    --json outputs/qc/trimming/${ACCESSION}_fastp_report.json
```

Here, we are using the default trimming settings for fastp, which generally do
a good job of removing adapters from Illumina data and trimming out the low
quality bases that often occur at the ends of reads. We also enable read
deduplication. When DNA is prepared for sequencing, fragments are amplified
by PCR, which means duplicate reads of the same fragment can get generated.
These are pseudoreplicates, as they come from the same original piece of DNA,
so we let fastp find them and only keep one copy.

Once fastp finishes, open its HTML report on your desktop. You can do this by
navigating to it in your file browser and opening it in your browser, or
using the command line:

```bash
firefox outputs/qc/trimming/${ACCESSION}_fastp_report.html
```
Compare the **before filtering** and **after filtering** summaries, including:

- read and base counts retained or removed;
- per-base quality, especially toward read ends;
- adapter content; and
- the estimated duplication rate.

Check how many lines each of the cleaned outputs has:

```bash
zcat outputs/trimmed_reads/${ACCESSION}.R1.trimmed.fastq.gz | wc -l
zcat outputs/trimmed_reads/${ACCESSION}.R2.trimmed.fastq.gz | wc -l
```

The two cleaned files should have matching line counts

## Assemble the genome with Spades

Run SPAdes on the cleaned paired reads:

```bash
mkdir -p outputs/assemblies/$ACCESSION

spades.py \
    -1 outputs/trimmed_reads/${ACCESSION}.R1.trimmed.fastq.gz \
    -2 outputs/trimmed_reads/${ACCESSION}.R2.trimmed.fastq.gz \
    -o outputs/assemblies/$ACCESSION \
    -t 6 \
    -m 12
```

`-t 6` requests six CPU threads and `-m 12` sets an 12 GB memory limit. These
work for the desktops in these course, but you will want to adjust them when
running spades elsewhere. This should take a couple minutes to run for a
genome of this size.

Spades will write its output to the `outputs/assemblies/$ACCESSION` folder. There
you'll find the log (`spades.log`), as well as the scaffolds (`scaffolds.fasta`)
and contigs (`contigs.fasta`) assembled by spades.

Take a look at the fasta headers for the scaffolds file:

```bash
grep ">" outputs/assemblies/$ACCESSION/scaffolds.fasta | less
```

This will show the names of each scaffold assembled by spades, from largest to smallest.
The header includes the average coverage of the reads used to assemble the scaffold
as well as the length of the scaffolds.

Take a look at the [reference genome for this species on NCBI](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000002945.2/).
Based on the number of chromosomes and their length there, how did your assembly fare?
Did you have any scaffolds that got close to assembling a whole chromosome?

## Assess the whole-genome assembly with BUSCO

You'll also see assembly statistics on the reference genome's web page. We will
assess the completeness of our assembly with BUSCO and get some similar stats,
as well as estimate completeness. BUSCO uses databases of single copy orthologs,
genes present in one copy in most of the genomes of a taxonomic group, to assess
completeness. If an assembly has most of the expected ones, and they aren't
extremely duplicated, the assembly is relatively complete and the haplotypes have
collapsed mostly (i.e. both copies of the genome are being represented in one sequence).

We will run BUSCO using the database of single copy orthologs for fungi here:

```bash
mkdir -p outputs/busco

busco \
    -i outputs/assemblies/$ACCESSION/scaffolds.fasta \
    -o outputs/busco/$ACCESSION \
    -l fungi_odb12 \
    -c 6 \
    -m genome \
    --metaeuk
```

Here, `-i` and `-o` define input and output, respectively, `-l` defines the
database of BUSCOs we want to search for, `-c` sets the number of CPUs,
`-m` sets the input mode as genome, and `--metaeuk` enables the metaeuk gene
predictor. This method uses [MetaEuk](https://github.com/soedinglab/metaeuk)
a tool for gene discovery that identifies genes in the genome, then BUSCO
matches them to genes in the BUSCO database set with `-l`. This may take a few
minutes.

Take a look at the summary file for the BUSCO run:

```bash
less outputs/busco/${ACCESSION}/short_summary.specific.fungi_odb12.${ACCESSION}.txt
```

What you will likely see is that you have all right completeness, something
like 80-90% of the expected genes, as well as a total assembly length around the
12 Mb expected for this species. However, you likely will have a much lower N50 than
the reference genome and many more contigs. This suggests we have a fragmented
assembly, but that we recover most of the genome, especially genic regions. Long-read data
likely would build us a more contiguous assembly, but for pulling out genes to
build phylogenetic trees, we actually can get quite far with short-reads.

Speaking of pulling out genes, BUSCO has kindle pulled them out and organized them
for us. Check out the contents of the
`outputs/busco/ERR17038703/run_fungi_odb12/busco_sequences/single_copy_busco_sequences/`
folder. There will be three files for each complete, single-copy BUSCO in the genome:
a coding nucleotide sequence (`.fna`), the translated protein sequence (`.faa`), and an
annotation file with its location in the assembly (`.gff`).

For phylogenomics, we can use BUSCO to get single copy orthologs from a number of assemblies,
then align each gene, giving us hundreds or thousands of genes to infer phylogenetic trees
from. We will cover this in a later tutorial, for now, let's take a look at assembling
the mitochondrial genome.

## Assemble the mitochondrial genome with GetOrganelle

Take a look again at your assembly from before, examining the fasta headers:

```bash
grep ">" outputs/assemblies/$ACCESSION/scaffolds.fasta | less
```

Scroll through and look at the coverages, most should be similar, but at least
one or two should stand out, maybe hundreds or thousands times coverage. These
scaffolds are also probably less than 15 kb. These are mitochondrial DNA most
likely, there are many mitochondria in each cell, so its DNA is abundant in
whole-genome sequencing data. This also makes it easy to assemble, but you'll
likely see multiple scaffolds like this as Spades hasn't done the best job.

Let's try assembling it with a dedicated mitochondrial genome assembler.
[GetOrganelle](https://github.com/Kinggerm/GetOrganelle) recruits reads likely
to belong to the organelle, assembles them, and attempts to resolve them into
an assembly graph. Since it is for organelles, you can also use it for assembling
plastomes in plants. We will use its fungal mitochondrial template, `fungus_mt`,
on the same cleaned paired reads. We'll need to download the template first:

```bash
get_organelle_config.py --add fungus_mt
```

Then we can run the mitochondrial assembly. It's a similar command to Spades:

```bash
get_organelle_from_reads.py \
    -1 outputs/trimmed_reads/${ACCESSION}.R1.trimmed.fastq.gz \
    -2 outputs/trimmed_reads/${ACCESSION}.R2.trimmed.fastq.gz \
    -o outputs/assemblies/${ACCESSION}-mito \
    -F fungus_mt \
    -R 10 -k 21,45,65,85,105 \
    -t 6
```

`-t 6` sets the number of threads, `-R 10` allows up to ten read-recruitment
rounds, and `-k` supplies the k-mer sizes for assembly. Here we are using the
suggested values for a fungus mitochondria in the
[GetOrganelle documentation](https://github.com/Kinggerm/GetOrganelle#readme).

### Check the mitochondrial assembly

The mitochondrial genome is circular, and a highly successful assembly will be
able to reconstruct the full circular assembly, let's check for that in the log:

```bash
grep "Result status" outputs/assemblies/${ACCESSION}-mito/get_org.log.txt
```

We should be able to get a circular genome out of these reads, so it should say
so in the output there. If not, GetOrganelle has some suggested [parameter
adjustments to try to get there](https://github.com/Kinggerm/GetOrganelle/wiki/FAQ#what-should-i-do-with-incomplete-resultbroken-assembly-graph).

### Examine the assembly graph with Bandage

GetOrganelle saves an organelle-specific graph as a `*.selected_graph.gfa`
file. In the graph, nodes represent assembled sequence segments and edges
represent connections between them. To see the files produced by your run:

```bash
ls -lh outputs/assemblies/${ACCESSION}-mito
```

Open [Bandage](https://github.com/rrwick/Bandage) on your Linux desktop:

```bash
Bandage
```

In Bandage, use **File > Load graph** to select the
`*.selected_graph.gfa` file in `outputs/assemblies/ERR17038703-mito/`, then
click **Draw graph**. If you have a complete circular genome, this might be
a rather boring graph. Try exploring the nuclear graph afterwards, it is a
`.gfa` file in the output folder from Spades.