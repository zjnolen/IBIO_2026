
# **MULTIPLE SEQUENCE ALIGNMENT**
You have now learned how to assemble genomes, pull sequences from those genomes, download data from the internet, and more. However, our sequences probably have different lengths, and nucleotide positions are not automatically *homologous*! To do phylogenetics our sequences need to be aligned.

Here we are going to align our datasets using one of the most used aligners: MAFFT, an acronym for *multiple alignment using fast Fourier transform*

MAFFT is not the only aligner out there, but it is very widely used. It is made for unix-like operating systems and it offers loads of options and methods. We will use MAFFT because it is so well known and common, but we will not get into anything too complicated here.

Our alignment workflow will follow two easy steps:
- Alignment of the amino acid files using mafft
- Trimming and back translation into nucleotides using trimAl

Trimming is the process of removing weird gaps and uninformative sites from our alignments and should make our alignment better. Much like alignment it can be done manually or, as we will do here, automatically. 

By aligning the amino acids and then backtranslating them to nucleotides we are producing a codon aware alignment.  This should also make for a better alignment.

We will need the following tools: mafft, trimAl

## Part 1. Setup

You have already gotten your mitochondrial genes, but we have also prepared a set of BUSCO genes for you. Let's start by fetching those. 

The BUSCOS ar sitting here: *Data/papilionoidea_10buscos.tar.gz*

Extract them and put them in a suitable folder. Something like this:

```bash
mkdir Data/BUSCO/genes
cd Data/BUSCO/genes
tar -xf ../../Data/papilionoidea_10buscos.tar.gz
ls -h
cd -
```

Check it out! Look's like we now have some genes. You'll notice we have files ending in .faa and .fna for each gene. Those are amino acids and nucleotides, respectively. We shall use both!

Here we will show you how to align and trim these genes, and then you will have to do the same thing again but for the mitochondrial genes.

## Part 2. Alignment

Now that we have our genes, let's run mafft!

Let's first make a directory for our alignments.

```bash
cd Data/BUSCO
mkdir -p aligned 
```
Now we can run mafft. We can check out the options like so:

```bash
mafft -h
```

There are different settings we can explore. For our purposes, the simplest way of just using the *--auto* option should work just fine. It will also be able to detect that we are using amino acids.

We have to run it once for each gene. You can do that manually or as a loop like this one:

```bash
#Loop through each .faa file and align the genes within
for i in genes/*.faa; 
do 
        gene=${i%.multifasta.faa} 
        mafft --thread 1 --auto "$i" > "aligned/${gene}.aligned.faa"
done
```

You should now have a set of aligned amino acid sequences. You can check them out using a text editor or **Aliview**, just to see what they look like.

## Part 3. Trimming and back translation

Now we will trim our alignments. We will use **trimAL**, which is a common trimming tool and which has an automatic trimming feature. It can also backtranslate amino acids to nuclotides, so we can do two things in the same command.

```bash
trimAl -h #check out the options
```

The *-automated1* trimming method should be fine. Feel free to explore others; this dataset is small enough that you may actually be able to visually perceive the differences.

We also see that to properly backtranslate **trimAl** needs both the aligned amino acids and the original unaligned nucleotides.

This little loop does what we need; it takes each amino acid alignment and the corresponding nucleotide fasta and feeds these to **trimAl** to trim and backtranslate.

```bash
for i in aligned/*.aligned.faa; do
    gene=$(basename "$i" .aligned.faa)
    trimal -in "$i" \
        -automated1 \
        -backtrans "genes/${gene}.multifasta.fna" \
        -out "aligned/${gene}.aligned.fna" \
        -fasta
done
```

That should be all you need to do! But, we of course should also check our alignments. 

## Part 4. Visualising the alignments
It is always good to actually look at your alignments and not jsut trust that the automatic tools did everything perfectly. Open the **Aliview** for that.

*How does it look? Does it look "aligned"?* You should see something like this now (but with different taxa):

<p align="center"><img src="./Aliview7.png" alt="Aliview7" width="800"></p>

*Is there anything in the alignment which catches your attention?* Scrolling along the alignment, it might not look very good in some parts, whereas other parts look quite nicely aligned. *Why might that be?* Does this look like an alignment you would be happy to analyse using phylogenetic methods? Maybe not. Remember, alignments are statements of homology for the positions of nucleotides in your sequences, and in some positions we may be unsure of homology (lots of gaps).

Check each of your alignments in **Aliview**. *How do the alignments look? Would you be willing to analyse these data in a phylogenetic framework?* Visualizing an alignment like this is very complicated. Try to use the visualizing options you have in **Aliview**. For example if you click on the `Translate nucleotide sequence to Amino Acids`, you should see something like this:

<p align="center"><img src="./Aliview4.png" alt="Aliview4" width="800"></p>

The majority of the changes in the nucleotide level should not change the corresponding amino acid. This are **Synonymous Mutations** as you have probably guessed. Are there some places where this is not the case? Anything that looks weird? 

You could manually remove or add some sites to make the alignments look better (using commands in **Aliview**), if you feel it is needed. However, be aware that searching for the perfect alignment can be a nearly infinite task.

## Done!

If you are happy with your alignments, we are finished with this section. Now you have to figure out how to do the same thing with the mitochondrial genes!