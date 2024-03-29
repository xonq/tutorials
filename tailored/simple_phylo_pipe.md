The point of the project is to familiarize and implement what you've learned in
class. Therefore, look to the Jelmer's project announcements and descriptions
to determine what to emphasize. e.g. the focus is on good documentation, data
handling practices, version control via git, building a pipeline, perhaps writing a bash
script, or creating a pipeline to run the separate parts of an analysis etc etc.
You don't need the output data to be the highest quality
possible, but if this is relevant to your research interests then by all means
make it high quality.

<br />

## What is MycoDB?

MycoDB is a database of JGI and NCBI genomes that's uniformly curated with
systematic naming schemes. That way the sequence data will play nice with
software. 

The second benefit of MycoDB is that it is modular and scalable. In other
words, you can run an analysis referencing the entire database ("master MycoDB") 
of fungi, or you can "abstract" a specific set of organisms into an smaller
MycoDB .db file for your analysis. This is what is demonstrated in the 
[USAGE guide for
`abstractDB.py`](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#creating-modular-databases).

<br />

## What is MycoTools?

MycoTools is a set of python tools designed to make interfacing with MycoDB
easy and to expedite routine bioinformatic tasks.

<br />

## Getting MycoTools running

You essentially need to create a `conda` environment at OSC as discussed in
class, then activate the environment and install MycoTools. This is touched on
[here](https://gitlab.com/xonq/mycotools/-/tree/master/mycotools#install).
Since you've already created a python3 environment for class, you can simply
activate it, then run `pip install mycotools`.

Once installed, you will also need to add the following to your
`~/.bash_profile`; if you already see these lines in your `~/.bash_profile`,
then remove them:
```
export MYCOGFF3=/fs/ess/PAS1855/mycodb/data/gff3/
export MYCOFAA=/fs/ess/PAS1855/mycodb/data/faa/
export MYCOFNA=/fs/ess/PAS1855/mycodb/data/fna/
export MYCODB=/fs/ess/PAS1855/mycodb/mycodb/
```

You will have to restart your console once all this is done the first time.
To use MycoTools in the future, activate the environment. 

I recommend you test it is working after a restart by running `mycodb` and
seeing if it executes without error. If it does, it will output the path to 
the master MycoDB `.db` file.

<br />

## Inputting data into MycoTools

Each script will have its own input requirements and you can find out what the script
inputs by running, for example, `db2blast.py -h`, which will pull up the help
menu. If you fail to put in the required inputs, the script will let you know.
For example, say I run `db2blast.py` without the required inputs:

`db2blast.py`

This returns the following error:

`db2blast.py: error: the following arguments are required: -b/--blast,
-q/--query`

<br />

## Running `db2blast.py`

What you'll find if you run `db2blast.py -h` is that this script "Blasts a query against a db and
compiles output fastas for each query." There are several required inputs for
this script:

- A MycoDB `-d`: by default this script will reference the master MycoDB of all
  organisms. If you want to reference the master MycoDB then you don't have to
  input anything for `-d`. However, if you only want to reference a subset of organisms, you
  can [abstract a 
  MycoDB](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#creating-modular-databases)
  to contain the lineages of interest.

- A query fasta `-q`: this fasta file will contain the sequences you want to blast
  search the database for. A fasta can either contain amino acid sequences or
  nucleic acid sequences. You can acquire this fasta for your gene of interest
  online (say at a database like UniProt), or mix several genes of interest. 
  Each sequence within a fasta is denoted with a header
  line that starts with `>` followed by lines of the sequence. If you run a
  `db2blast.py` search with multiple sequences (a "multifasta"), then it will output
  a results file for each individual query sequence within the fasta. Here is
  an example fasta that will output two results fastas:

```
>SEQUENCE 1
AWAYWJWMWJW
AAYYYZYYJJJ
>SEQUENCE_n
AAYYYAAZZYY
JJJAAAYYJJJ
```

- The type of blast search `-b`: there are multiple flavors of blast - `blastp`,
  `blastn`, `tblastn`, and `blastx`. Each one has its specific use case.
  `blastp` blasts a protein fasta against protein sequences, `blastn` blasts a
  nucleotide fasta against genome sequences, `tblastn` reverse translates a protein
  fasta into all possible translations and then blasts against nucleotide
  sequences. I highly recommend you use `blastp` or `tblastn` because they are
  more sensitive than `blastn`.

- Finally, there are several nonrequired inputs. You can specify the e value
  maximum (`-e`), the bitscore minimum (`-s`), the minimum percent identity
  (`-i`), the maximum number of top hits you want per organism (`-m`), an output directory (defaults
  to your current directory, `-o`), and the number of CPU cores (increases
  throughput to a point, `--cpu`). Most of these are ways to filter your
  dataset to include reasonable hits.

I highly recommend you implement an e-score cutoff or bitscore cutoff to reduce
your output size. For the e-score a good arbitrary cutoff is 10^-2, which can
be specified via `-e 2`. `--cpu` will use all cores by default, but it has to
match the cores you input on your slurm job submission script when you run it.
In other words, if you tell slurm to run 5 cores, your CPU argument would be 
`--cpu 5`.

Here is a good example `db2blast.py` if you want to do a phylogenetic analysis
on a gene that references the master MycoDB

```
db2blast.py -q <QUERY_FASTA.aa.fa> -b blastp --cpu 12
-e 2
```

You can also use an abstracted MycoDB that only contains your lineages of
interest:

```
db2blast.py -q <QUERY_FASTA.aa.fa> -b blastp -d <ABSTRACTED_MYCODB.db> --cpu 12
-e 2
```

This will generate an output directory like this:
```
20210409_db2blast/
----fastas/
--------<QUERY_1.fa>
--------<QUERY_2.fa>
----reports/
--------<ORGANISM_1.out>
--------<ORGANISM_n.out>
````

You will only be concerned with the fasta outputs. They contain all the
significant hits for your query sequence(s). If you don't have blast 
installed, run `conda install blast` in your environment.

<br />

## Running `fa2tree.py`

There are dependencies you'll need installed to run this script. In your
conda environment, run `conda install iqtree fasttree mafft trimal`.

If you run `fa2tree.py -h`, you'll find that it "Takes in a multifasta or
directory of multifastas, then aligns, trims, and runs treebuilding."

Therefore, all you need is a fasta or directory of fastas, like what
`db2blast.py` outputs! You can also tell `fa2tree.py` which tree building
software you want to use. IQTree is more robust, but it takes much longer than
fasttree. Since this is a class project, fasttree is sufficient, but if you
really care about the data run IQTree. You specify this with `-t fasttree`.

The other cool thing about `fa2tree.py` is that it will
submit the jobs for you because building a tree is three steps - aligning the
sequences, trimming them, then building the tree. You can tell it to submit to
slurm by adding the `-s` flag as well as a `-A PAS1855` flag for the class
project number. 

Here is an example of running a job submission on constructing a tree from you
blast results:

```
fa2tree.py -i <DB2BLAST_OUTPUT/fastas> -t fasttree -s -A PAS1855 -o
<YOUR_OUTPUT>
```

This will output all the trees for each fasta into their own subfolder. If you
end up making a tree let me know and I can help you visualize it.   
