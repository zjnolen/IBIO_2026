# Model selection 💻

In the first tutorials you have learned how to both handle raw data and assemble genomes as well as how to download sequences from GenBank and how to align them. For the next tutorial, we are going to use *ModelFinder* to find the best partitioning scheme for our nuclear butterfly dataset 🦋.

<p align="center"><img src="Substitution model.png" alt="The foundations for a substitution model" width="600"></p>

*Here is a graphical representation of the six types of nucleotide substitutions that may occur in a phylogenetic dataset. As an example, JC69 is the simplest model where all substitutions (both transitions and transversions) are assumed to occur at an equal rate. Other commonly used models are HKY which assumes that transitions and transversions have different rates and GTR which assumes all six types of substitution rates differ.*

## Choosing the right substitution model 🧬

Now, we will use ModelFinder ([Kalyaanamoorthy et al. 2017](https://www.nature.com/articles/nmeth.4285)), as implemented in [IQ-TREE3](https://iqtree.github.io/) ([Nguyen et al. 2015](https://academic.oup.com/mbe/article/32/1/268/2925592)), to determine the best-fit model for these ten different genes. IQ-TREE3 for Windows, MacOSX and Linux can be downloaded [here](https://iqtree.github.io/#download), which you can also install on your personal computer.

ModelFinder chooses the model that minimizes the BIC score (you can also change the selection criteria to AIC or AICc by adding the option -AIC or -AICc, respectively). If you have a multi-gene dataset, it is common practise to concatenate the gene alignments for all taxa into a supermatrix (alignment containing all genes and taxa), and also to partition the dataset in some way. A dataset may be partitioned in many different ways (e.g. by codon position or by gene), and here we will do it by gene. By partitioning the dataset by gene, we will determine the best fit models for every gene before inferring our tree.

Luckily for us, IQ-TREE has a built-in function which lets us concatenate our aligned genes right before alignment, if they are found in the same folder. Therefore, first inspect your *aligned* folder with the final nuclear alignments, and copy these aligments to a separate folder, which you can name e.g. *final_nuc_alignments*.

### Running ModelFinder with IQ-TREE 🪾

Running IQ-TREE is relatively straightforward. It is a powerful program with many diffrent options, and there is good documentation that you may find [here](https://iqtree.github.io/doc/iqtree-doc.pdf). If you want an overview of the different options, run `iqtree -h`.

 Now, we will start our modelfinding by typing the following command and running it by hitting enter:

```
iqtree -p final_nuc_alignments --prefix first -m TESTMERGEONLY
```

Here, `-p final_nuc_alignments` designates *final_alignments* as the input folder containing the genes, `--prefix first` gives all the output files the prefix *first* and `-m TESTMERGEONLY` specifies that we want to only select the best-fit partitioning scheme of our genes by possibly merging partitions to reduce over-parameterization and increase model fit, without inferring a tree afterwards.

This command will produce a few files as output, including a file ending with `.best_scheme.nex`. This one will contain our inferred partitioning scheme in the so-called Nexus format, and shows us the gene partitions and the models that were chosen as the best fit. If you open the file it will look something like this:

```
#nexus

begin sets;
    charset gene1 = 1-987;
    charset gene2 = 988-1845;
    charset gene3 = 1846-2912;
end;

begin models;
    partition PARTITION =
        GTR+F+R3: gene1, gene3,
        TIM2+F+R2: gene2;
end;
```
The first part of the output shows as where the genes are located in the supermatrix and the second part which models were chosen for which gene partitions. Here *gene1* and *gene3* are merged into a single partition.

The above settings for model selection `-m TESTMERGEONLY` is the standard model selection which will only consider the invariable site and Gamma rate heterogeneity. If we want to consider an extended model selection that also includes the FreeRate heterogeneity model `-m MF` then we need to run:

```
iqtree -p final_nuc_alignments --prefix second -m MF+MERGE
```

Now, as a third option, if you want to restrict your model search to only those models supported by a specific program (Mr Bayes for example), then run:
```
iqtree -p final_nuc_alignments --prefix third -m TESTMERGEONLY -mset mrbayes
```

If you are in this section, well done! Now you have several files with the selected best-fit model for our dataset.

**Questions:**

1. Compare the three files that you should see now that all end with `.best_scheme.nex`. These will include the estimated best partitioning schemes and the best fit models for them. Are there any differences?
