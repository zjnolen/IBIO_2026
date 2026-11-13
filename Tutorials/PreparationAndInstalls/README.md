# Tutorial 0: Preparing your working directory and installing software with Conda

## Background

Phylogenomic analyses often involve a mixture of command line and graphical
tools that you will need to install either on your own or a remote computer.
Here we will go over how you can set up your working directory to keep your
project organized and install specific versions of bioinformatic software.

## Working directory

We suggest setting up a folder you will work in for this course and have a
suggested structure to keep things organized:

```bash
~/
└──phylogenomics_course/
    ├── data/
    ├── outputs/
    └── scripts/
```

Here, we have the home folder `~`, which has your working directory,
`phylogenomics_course`, in it. Inside your working directory, you have
the `data` folder where you can store raw data, `outputs` folder where
you can organize your analysis outputs, and `scripts` where you can
store the scripts you write, which are needed to reproduce your work.

Open the Terminal app and create these directories with the `mkdir`
command:

```bash
# from inside your home (~) folder:
mkdir phylogenomics_course
# enter the phylogenomics_course folder:
cd phylogenomics_course
# now make the others
```

For this course, you will need to use the terminal to run commands like
this and a plain text editor to edit scripts. On the machines you have, you
can use Visual Studio Code to combine these two apps into one, but there's
many different text editors and terminal emulators you can use for these
purposes.

## Setting up a software environment with Conda

[Conda](https://github.com/conda/conda) is a package manager that lets you
set up isolated environments with a set of specific software. You can use
it to set up an environment to run your project in where you have all of the
software you need installed and set to the specific versions you want (making
it easier to run different projects on different versions by using a project-
specific environment).

Check if your machine has Conda installed, and if it doesn't install it:

```bash
# check if conda is installed, it should tell you the version if so:
conda -V

# IF it is not found, you can install it, skip this if its installed:
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

Once installed, we can prepare an environment for this course. We will
use an environment file (`environment.yaml`) to define the software to
include in the environment, set their versions, and what repositories
they should come from. Create an `environment.yaml` file in your working
directory with the following contents:

```yaml
channels:
  - conda-forge
  - bioconda
  - nodefaults
dependencies:
  - aliview=1.33
  - aster=1.25
  - bioconductor-ggtree=4.0.5
  - busco=6.1.0
  - fastp=1.4.0
  - figtree=1.4.4
  - getorganelle=1.7.7.1
  - iqtree=3.1.4
  - mafft=7.526
  - mitos=2.1.10
  - ncbi-datasets-cli=18.41.1
  - r-ape=5.8_1
  - r-base=4.5.2
  - r-ggplot2=4.0.3
  - spades=4.3.0
  - sra-tools=3.4.1
  - trimal=1.5.1
```

Then, you can create the environment from this file with:

```bash
conda env create -f environment.yaml -n phylo
```

This will create a Conda environment with the name `phylo` that contains
all the software we will use in the course. When you open a new terminal
window, you'll need to activate this environment to use the software
installed to it. You'll see the currently active environment in your terminal
in parentheses. By default, your base environment activates, so you'll see
`(base)`. Activate the `phylo` environment:

```bash
conda activate phylo
```

You should see the terminal change to say `(phylo)`. Now you're ready!

Putting everything you need for a project into one environment may not be
ideal. In this case, these tools can form a compatible environment on this
system. However, you may want to make an environment for each tool and
activate it whenever you need the tool:

```bash
# you don't need to run this
conda create -n spades -c conda-forge -c bioconda "spades==v4.3.0"
```

This command would create an environment named (`-n`) spades that installs
just spades v4.3.0 and its dependencies. The `-c` options set the channels
that you saw in the `environment.yaml` and tells conda where to look for
the software. Bioconda is a repository of biology related software and
conda-forge contains a wide variaty of open-source software and is added so
that non-biology related dependencies can be grabbed (like python for spades).

## Saving your data processing as scripts

These tutorials will give examples of commands that you might run one by one.
Rather than running each command directly in your terminal, it is best to put
them in a script. This makes it easier to rerun the analysis and see exactly
which commands produced your results.

Create scripts in the `scripts/`, and you can organize them in subfolders, say
`scripts/assembly/`. Inside that folder, you can create a text file called
`scripts/assembly/download_ERR17038703.sh`, which contains the commands to
download the reads for a specific sample. You could also combine all steps for
assembly into one script called `scripts/assembly/assembly_ERR17038703.sh`.

Consider how best to make it so you can keep track of what you are running. You
can use comments in your script for this, lines starting with `#`, which aren't
run. Here's an example of what `scripts/assembly/download_ERR17038703.sh` might
look like (you don't need to run these yet, we will get to it in the assembly 
tutorial):

```bash
#!/usr/bin/env bash

# define environment variables for the sample accession and output directory
ACCESSION=ERR17038703
OUT="data/raw_reads"

# make the output directory if it doesn't exist
mkdir -p $OUT

# download the fastq files for the accession to the output directory
fasterq-dump "$ACCESSION" \
    --split-files \
    --outdir "$OUT"
```

Then, you can run everything in the script with the command:
`bash scripts/assembly/download_ERR17038703.sh`. From here on the tutorials
are written so that you run the commands directly, which will make it easier
to troubleshoot and learn each step in the course. However, try to build
scripts as you go, so that you have a documentation of what you've done and
can reproduce it.
