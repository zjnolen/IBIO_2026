# Tutorial 5: Maximum-likelihood tree inference 🪾

## IQ-TREE3
Now that we learned how to use IQ-TREE for model and partition search, we will move on to doing phylogenetic inference with it.

**Command line instructions and tree inference** 🪾

Tree inference is one of the most frequently used features of IQ-TREE and allows users to carry out phylogenetic analysis on a multiple sequence alignment (MSA). In the most basic case, no more than an MSA file is required to submit the job. Without further input, IQ-TREE will run with the default parameters and auto-detect the sequence type as well as the best-fitting substitution model. Additionally, Ultrafast Bootstrap ([Hoang et al., 2018](https://academic.oup.com/mbe/article/35/2/518/4565479)) and the SH-aLRT branch test ([Guindon et al., 2010](https://academic.oup.com/sysbio/article/59/3/307/1702850)) will be performed.

Go into the same folder in your terminal as you did for the [previous tutorial](../ModelSelection/Readme.md#Running-ModelFinder-with-IQ-TREE).


**Command reference**

`iqtree`: will call the executable in our activated Conda environment

`-p <alignment-folder>`: this option points your program to the alignment folder

`-m MFP+MERGE`: like the option `MF+MERGE` that we used last time, but actually does phylogenetic inference with the found best models for the best partitioning scheme

`-B`: will perfom Ultrafast Bootstrapping

`-alrt`: will perform the SH-aLRT test

`-bnni`: reduces the risk of overestimating branch supports with UFBoot due to severe model violation


1. Now you can try to run a simple analysis by entering

```
iqtree -p final_nuc_alignments
```

In this case, IQ-TREE3 will first concatenate the sequences found in the folder *final_nuc_alignments* and then infer a tree from this concatenated dataset with the best-fit model automatically selected.


2. However, to find the best partition scheme by possibly merging partitions, followed by tree inference and branch support, run an analysis by entering:

```
iqtree -p final_nuc_alignments -m MFP+MERGE -B 1000 -alrt 1000 --prefix tree_inference_nuc -bnni
```

**Output files to check**

   Suffix	     
   
- `.iqtree`          Full result of the run – this is the main report file

- `.log`	            Run log

- `.treefile`	      Maximum likelihood tree in NEWICK format, can be visualized with treeviewer programs

- `.ufboot`          Bootstrap trees

- `.best_scheme.nex` Best partitioning scheme


3. Now, infer a tree also from the mitochondrial genes.

**Congratulations! Now that you have run the analyses, you can compare the results of the trees.**


# Simple tree visualization 🪾

To have a quick look at your inferred trees, we suggest you open your IQ-TREE `.treefile` in FigTree. You can do this from your terminal by typing `figtree <treefile>`.

IQ-TREE estimates an unrooted tree unless a root was specified in the
analysis. The way an unrooted tree is drawn does not identify a biologically
meaningful root, and the tree is therefore in FigTree rooted by default on the first taxon in your dataset or on the longest branch in the dataset. As we have included outgroups to the butterflies in our tree inference, we can now reroot the tree using these. Accurate rooting is important for the polarity of the tree, and we should therefore do some research before rooting our tree. Have a look at this [article](https://www.pnas.org/doi/10.1073/pnas.1907847116), which includes a large phylogenetic tree of Lepidoptera. (Epermenioidea is missing in this tree, but (*hint!*) the tree should not be rooted on this group.) *Considering this tree and our outgroup taxa, which clade should we root the tree on?*

Find this clade and select the branch leading to the clade in FigTree, then click 'Reroot' in the top panel. Note that these examples use a different set of taxa than you used, but the interface remains the same.

<p align="center"><img src="./IQTREEraw.png" alt="Unedited IQ-TREE tree in FigTree" width="600"></p>

<p align="center"><img src="./IQTREErooted.png" alt="Example tree rerooted on an outgroup" width="600"></p>

Finally, to interpret how robust our tree is we need to know the support for our clades – in this case the bootstrap support and the support of the SH-aLRT (Shimodaira–Hasegawa approximate likelihood-ratio) test we performed. To make the branch support visible, click on the `Node Labels` tab on the left, and then *Display:* `label`. The first value you see is for the SH test, the second for the ultrafast bootstrapping. Often, a branch is considered well supported if it receives both SH ≥80 and UFB ≥95. Finally, now inspect the tip names and branch lengths, and check the internal labels for the support values reported by your analysis.

<p align="center"><img src="./IQTREEc.png" alt="IQTREEc" width="600"></p>

**Questions**

1. *Are the UFBoot values higher/lower compared to those recovered from the SH-like test in IQTREE?*

2. *How are the nodes/relationships supported?*

3. *Take a look at the best partitioning scheme. What are the merged partitions? Can you see any similarity pattern beween the merged partitions?*

4. *How do the mitochondrial and nuclear trees differ?*

5. *How robust are your trees? Are any branches poorly supported?*




