# Model Selection


In the first tutorials you have learned how to both handle raw data and assemble genomes as well as how to download sequences from GenBank and align them. For the next tutorial, we are going to use *ModelFinder* to find the best partitioning scheme for our dataset and also the best models for each partition.

In the last tutuorial we aligned ten genes. If you open *Aliview* to view the them you should have something similar to this:

<p align="center"><img src="../../Tutorials/ModelSelection/Aliview7.png" alt="Aliview7" width="900"></p>

## Choosing the right substitution model

Now, we will use ModelFinder ([Kalyaanamoorthy et al. 2017](https://www.nature.com/articles/nmeth.4285)), as implemented in [IQ-TREE3](https://iqtree.github.io/) ([Nguyen et al. 2015](https://academic.oup.com/mbe/article/32/1/268/2925592)), to determine the best-fit model for these ten different genes. IQ-TREE3 for Windows, MacOSX and Linux can be downloaded [here](https://iqtree.github.io/#download), which you can also install on your personal computer.

ModelFinder chooses the model that minimizes the BIC score (you can also change the selection criteria to AIC or AICc by adding the option -AIC or -AICc, respectively). If you have a multi-gene dataset, it is common practise to concatenate the gene alignments for all taxa into a supermatrix, and also to partition the dataset in some way. A dataset may be partitioned in many different ways (e.g. by codon position or by locus), but here we will do it by gene. By partitioning the dataset by gene, we will determine the best fit models for every gene before inferring our tree.

Luckily for us, IQ-TREE has a built-in function which lets us concatenate our aligned genes, if they are found in the same folder. before running ModelFinder and inferring our tree, and it will also create the partition files for us, which look like this:

Nexus-style partition file may look like:

```
#nexus
begin sets;
    charset part1 = 1-69; 
    charset part2 = 70-666;
end;
```

### Running ModelFinder with IQ-TREE
Running IQ-TREE is very easy. We will start by typing the following command and running it by hitting enter.

```
iqtree -p gene_dir --prefix first -m TESTMERGEONLY

```

Here, we are:
- selecting the best-fit model for the concatenated alignment `-s ATP6_COI_CytB_ND5.phy`,
- by possibly merging partitions `-m TESTMERGEONLY` to reduce over-parameterization and increase model fit. 
- saving all generated files from this run to files that all have names starting with `--prefix first`.

Above settings for model selection will only consider the invariable site and Gamma rate heterogeneity. If you want to consider the FreeRate heterogeneity model too, then run:

```
iqtree -p gene_dir --prefix second -m MF+MERGE

```

As a third option, if you want to restrict your model search to only those models supported by a specific program (Mr Bayes in our case), then run:
```
iqtree -s ATP6_COI_CytB_ND5.phy -p Gene_partitions.txt -m TESTMERGEONLY -mset mrbayes --prefix third

```

If you are in this section, well done! now you have several files with the selected best-fit model for our dataset.

Questions:

1. Compare the three files that you should see now that all end with `.best_scheme.nex`. These will include the estimated best partitioning schemes and the best fit models for them. Are there any differences?
