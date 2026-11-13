# Tree visualization and manipulation

Once trees are generated, they can be used for a variety of purposes, from
further analyses on the evolution of characters and other features, to making
nice figures for publications. In this tutorial we will learn how to make a
tree look nice for a publication or presentation, and think about the format
of a tree file for other programs to read in for further analyses. We will
first use **FigTree** to explore and make simple changes to the tree. We will
then make a reproducible plot in R with ggtree.

## Explore the tree in FigTree

Open your IQ-TREE `.treefile` in FigTree. You can do this from your terminal by
typing `figtree <treefile>`. Inspect the tip names and branch lengths, and check
the internal labels for the support values reported by your analysis.

IQ-TREE estimates an unrooted tree unless a root was specified in the
analysis. The way an unrooted tree is drawn does not identify a biologically
meaningful root. This is why we have included outgroups to the butterflies in
our tree inference, we can now reroot the tree using these. Find the clade
containing the outgroup taxa and select the branch leading to the clade in
FigTree, then click 'Reroot' in the top panel. Note that these examples use
a different set of taxa than you used, but the interface remains the same.

<p align="center"><img src="./IQTREEraw.png" alt="Unedited IQ-TREE tree in FigTree" width="600"></p>

<p align="center"><img src="./IQTREErooted.png" alt="Example tree rerooted on an outgroup" width="600"></p>

Now the tree is technically correct, but perhaps aesthetically not so nice,
the branches appear rather jumbled. One way to make the tree appear neater
is to *ladderize* it. Click on the `Trees` tab on the left and choose *Order nodes*.
Ordering can be `increasing` or `decreasing`, choose the one which looks nicest
to you! Here it is with `decreasing` chosen:

<p align="center"><img src="./IQTREEladder.png" alt="IQTREEladder" width="600"></p>

Perhaps we want to highlight some clades for our publication. We can do this by
colouring the branches. Choose ***Clade*** in the ***Selection Mode*** button on
the top, and then click on the root of the clade of interest. When the clade is
highlighted, click on the ***Colour*** button on the top, and choose a colour
for the clade. Do the same for as many clades as you want. Here is an example
where two main clades are coloured:

<p align="center"><img src="./IQTREEcolour.png" alt="IQTREEcolour" width="600"></p>

Most likely we would like to report bootstrap and other support values on our tree
for a figure in a publication. To make them visible, click on the `Node Labels`
tab on the left, and then *Display:* `label`:

<p align="center"><img src="./IQTREEsupport.png" alt="IQTREEsupport" width="600"></p>

There are various other things you can do in **FigTree**, e.g. make the branches
thicker, change the tree style, change the place of the tip labels, expand the tree,
increase/decrease font sizes, etc. Play around a bit with the options. To further
spice up your figure, e.g. to add pictures of the taxa in the tree, you can export
the tree as a pdf file and import it into a graphics program like Adobe Illustrator,
InkScape or even PowerPoint.

## Make a reproducible plot with ggtree

FigTree is convenient for browsing and quick edits. An R script makes the
plotting steps explicit and repeatable, and makes it easier to use species
taxa information from a table.

### Install and load the packages

We installed R and the packages we need in our conda environment at the start
of the course. You can run R in the command line with the `R` command.

Load the packages:

```r
library(ape)
library(ggplot2)
library(ggtree)
```

`ape` is a package for working with phylogenetic trees in R and is required by
almost all phylogenetics R packages. `ggtree` is used for plotting trees using
`ggplot2` syntax. We will make plots both with plain `ape` and `ggtree`.

### Read and check the tree

Set `tree_file` to the `.treefile` made by your IQ-TREE analysis. The path
below is an example; use the actual path and filename from your analysis.

```r
tree_file <- "outputs/iqtree/papilionoidea.treefile"
tree <- read.tree(tree_file)

length(tree$tip.label)
head(tree$tip.label)
```

The tip labels in the tree must match the labels used for annotation. Here,
each label is the species name followed by its assembly accession, separated
by a period (for example, `Hesperia_comma.GCA_905404135.1`). The species table
from Tutorial 2 is reproduced below as metadata for this plot; the `label`
column must match the tree tips exactly.

```r
metadata <- read.csv(text = "
label,family,superfamily
Hesperia_comma.GCA_905404135.1,Hesperiidae,Papilionoidea
Ochlodes_sylvanus.GCA_905404295.2,Hesperiidae,Papilionoidea
Thymelicus_lineola.GCA_963932265.1,Hesperiidae,Papilionoidea
Aricia_agestis.GCA_944452695.1,Lycaenidae,Papilionoidea
Polyommatus_icarus.GCA_937595015.1,Lycaenidae,Papilionoidea
Scolitantides_orion.GCA_964345685.1,Lycaenidae,Papilionoidea
Euphydryas_aurinia.GCA_975146965.1,Nymphalidae,Papilionoidea
Aglais_io.GCA_905147045.1,Nymphalidae,Papilionoidea
Vanessa_atalanta.GCA_905147765.2,Nymphalidae,Papilionoidea
Graphium_eurypylus.GCA_040363725.1,Papilionidae,Papilionoidea
Iphiclides_podalirius.GCA_933534255.1,Papilionidae,Papilionoidea
Papilio_elwesi.GCA_029849275.1,Papilionidae,Papilionoidea
Anthocharis_cardamines.GCA_905404175.1,Pieridae,Papilionoidea
Colias_croceus.GCA_905220415.1,Pieridae,Papilionoidea
Pieris_napi.GCA_905475465.2,Pieridae,Papilionoidea
Macrosoma_leucophasiata.GCA_036365455.1,Hedylidae,Papilionoidea
Calephelis_nemesis.GCA_002245505.1,Riodinidae,Papilionoidea
Bombyx_mori.GCA_030269925.2,Bombycidae,Bombycoidea
Cossus_cossus.GCA_965117205.1,Cossidae,Cossoidea
Epermenia_chaerophyllella.GCA_964235415.1,Epermeniidae,Epermenioidea
Bryotropha_terrella.GCA_964059635.1,Gelechiidae,Gelechioidea
Pyralis_farinalis.GCA_947507595.1,Pyralidae,Pyraloidea
")
```

Check for name mismatches before plotting. Extra rows in `metadata` are okay
when the tree contains only a subset of the listed species; those species are
left out of the plot. A tree tip without a matching metadata row
stops the script so that a label problem is not hidden.

```r
missing_metadata <- setdiff(tree$tip.label, metadata$label)
if (length(missing_metadata) > 0) {
  stop("No species-table metadata for tree tip(s): ",
       paste(missing_metadata, collapse = ", "))
}

not_in_tree <- setdiff(metadata$label, tree$tip.label)
if (length(not_in_tree) > 0) {
  message("Species in the table but not this tree: ",
          paste(not_in_tree, collapse = ", "))
}

metadata <- metadata[match(tree$tip.label, metadata$label), ]
```

### Root and plot

Root the tree based on the outgroup. Here the metadata can help us, as we
have the superfamily and know all non-Papilionoidea species in this set are
intended as the outgroup:

```r
outgroups <- metadata[metadata$superfamily != "Papilionoidea", "label"]

tree <- root(tree, outgroup = outgroups, resolve.root = TRUE)
```

Now, take a look at the tree with ape's plotting function with node labels:

```r
plot(tree, show.node.label = TRUE)
```

When passed a `phylo` object, ape's `plot.phylo` function takes over, opening
up some tree specific options. Here we enable node labels. ape also kindly
removes the underscores in our species names. We have the accession still
appended, but if our tips were only species names, this would already be in
quite a good format. You can see [more options for ape plotting here](
https://www.rdocumentation.org/packages/ape/versions/5.8-1/topics/plot.phylo).

While we can get quite far with ape, many of you may be familiar with how to
make plots in ggplot. If this is the case, you may be interested in ggtree.
Let's use ggtree to make a plot that ladderizes our tree and colors the tips
using the families in the metadata table. Our outgroup species all come from
different families, so to limit the number of colors we need, let's just
change their families to 'Outgroup' so they share a color:

```r
metadata[metadata$label %in% outgroups, "family"] <- "Outgroup"

ggtree(tree) %<+% metadata +
  geom_tippoint(aes(color = family), size = 3) +
  geom_tiplab()
```

Notice that there are a few things that we probably need to clean up that
were handled automatically by ape's plot function: species names are not
italicized, underscores remain in the labels, and long tip labels may get
cut off. We can make a display label from the species name while keeping
the full species-plus-accession label for matching metadata:

```r
metadata$display_label <- sub("\\.GCA_.*$", "", metadata$label)
metadata$display_label <- gsub("_", " ", metadata$display_label)

ggtree(tree) %<+% metadata +
  geom_tippoint(aes(color = family), size = 3) +
  geom_tiplab(
    aes(label = display_label, color = family),
    fontface = "italic",
    offset = 0.01,
    size = 3
  ) +
  scale_x_continuous(expand = expansion(mult = c(0, 0.1))) +
  theme_tree2()
```

Finding values for offsets and other adjustments for ggtree can be
challenging, as the axes are a bit odd. Generally, the y-axis for a
vertical tree is in number of tips, but the x-axis is in branch length
units. Play around with this a bit, we've added a scale with the tree
theme you can use to experiment. For instance, use `geom_point()` to
add a point and see if it ends up where you think.

Once you have a tree you like, you can save it as a PDF:

```r
dir.create("outputs/figures", recursive = TRUE)
ggsave(
  "outputs/figures/papilionoidea_tree.pdf",
  width = 10,
  height = 8
)
```

`ggsave()` will save the last plot you plotted by default. Play with the
width and height settings to get a good set of dimensions for your specific
tree.

There are a lot of ways you can use ggtree, including adding other data to
the tree. Take a look at [the guide for some ideas](https://guangchuangyu.github.io/ggtree-book/chapter-ggtree.html).
There is also [phytools](https://blog.phytools.org/), which has many cool
plotting features like plotting phylogenies next to each other (say to look
at co-evolution or mito-nuclear discordance) or on a map.
