# Systematically truncating your phylogenetic tree
If you are trying to study the evolution of a gene, a good rule of thumb is to attempt to decrease your dataset to fewer than 1500 genes to make it manageable. A good way to wrangle your data from the get-go is to implement an evalue/bitscore cutoff at the blast stage because evalue/bitscore removes genes that likely are not closely related. *However, you do not want to implement a max hits per organism cutoff because that removes closely related sequences from your phylogeny.* 

If our dataset is too large, which it usually will be with the whole database, We face a problem: phylogenetic quality depends on sample size and quality, so we want our initial phylogenetic tree to contain the most comprehensive dataset as possible. We can truncate the dataset by only inputting a subsample of lineages - like the subphylum Agaricomycotina - but this assumes Agaricomycotina contains all the lineages in a well-supported clade of the larger tree. This assumption may hold-up conserved, near single-copy genes.

If we do not want to make this assumption and the dataset we want to operate on is too large, then we need to implement methods that can systematically remove genes that likely are not closely related. Mycotools implements a way to extract lineages of interest and two approaches for systematically attempting to remove distant homologs following the `blast` stage and prior to phylogenetic reconstruction. 

<br />

## 1. Truncating by taxonomy

I pull Agaricomycotina as a smaller dataset for a gene phylogeny. This assumption would be justified if I observed on a larger tree that indeed Agaricomycotina is contained in a single clade with high node support. It is not really a binary decision, sometimes excluding lineages will not affect the topology of the taxa we are interested in... but sometimes including a larger sample can completely shift where some genes are placed on the phylogeny. If we make this assumption, the steps are to use the [extractDB.py](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#extractdbpy) command and then use that `.db` file as the input (`-d`) for `db2blast.py`. To do this, we can use the extractDB.py command:

```
extractDB.py -r subphylum -l agaricomycotina > agaric.db
db2blast.py -d agaric.db ...
```
 
Here `-r` is for the taxonomic rank we are extracting, and `-l` is the taxonomic lineage. `extractDB.py` can operate accept a variety of arguments, check `-h/--help` for more.

<br />

## 2. Truncating an amino acid sequence phylogeny
I recommend using amino acid sequences to construct phylogenies whenever possible, though sometimes this is not possible (in the case of ITS for example). 1) blast is more sensitive with amino acid sequences and 2) the regions of genes vary in how they evolve and their rates of evolution, so we want to construct a phylogeny from the active domains of proteins - we cannot do this with nucleotide sequences. There are reasons why the ITS still makes a good barcoding region, but I will not elaborate on that here.

There is a database that contains models of active domains of proteins - the Pfam database. This database is derived from models of the evolution of groups of related sequences... hidden Markov models (hmm). These models are contained within an `.hmm` file and can then be used to search for sequences that contain the model domain. Furthermore, they can extract the region of sequences that contain the model domain.

Therefore, we can obtain a dataset for phylogenetic reconstruction by first using `blastp`, truncate this dataset for genes that contain the active Pfam domain via `hmmer`, and then obtain the regions of these genes that contain the Pfam domain. In terms of Mycotools, we can use `db2blast.py` to obtain all potential protein sequence homologs from MycotoolsDB, obtain the Pfam we are interested in (search the sequence on the [Pfam website](https://pfam.xfam.org/)), then use `fa2hmmer2fa.py` to extract the Pfam domains from these genes. As the name implies, `fa2hmmer2fa.py` takes in a fasta, then runs `hmmsearch` to look for Pfam domains, then returns a fasta of the regions of genes with significant hits. Here is an example where we are looking for the active domain of the psilocybin methyltransferase (PsiM):

```
fa2hmmer2fa.py -f psicub1_72373_psim.blast.fa -q PF05971 -a \
--hmm /fs/project/PAS1046/databases/funannotate/Pfam-A.hmm -e 2 -b hmmsearch
```

`-f` is the input fasta, `-q` is the query Pfam we want (this can also be a file with a
new-line delimited list of Pfams), `-a` specifies it is an accession
(necessary for querying with the Pfam database), `-e` is the evalue threshold (similar to blast),
and `-b hmmsearch` is specifying we are searching protein sequences (`nhmmer` is for nucleotides),
`--hmm` is specifying the path to the `Pfam-A.hmm` database. The output of this is a fasta with
only the regions of genes that are in the Pfam. This will likely be smaller than your blast fasta,
but more importantly, it only contains the active domain of your protein according to Pfam. We
may still need to dwindle your dataset as described below.

<br />

## 3. Truncating using hierarchical agglomerative clustering of sequence identity
Another systematic cutoff we can use is percent identity, which is a measure of how similar the sequences are. We can use percent identity as a cutoff on its own at the blast step, but we can also use `aggClus.py` from Mycotools to extract clusters of highly identical sequences from a distance matrix of percent identity. You will have to "tune" the thresholds in this script to obtain clusters of the size that you want.

`aggClus.py` works by using a dependency to calculate the percent identity of all pairwise sequence comparisons. In other words, it will take each sequence in your fasta, then align it to all other sequences, and calculate the percent identity of each comparison. This generates a distance matrix. We can then use hierarchical agglomerative clustering to group the sequences in this distance matrix into clusters of highly identical sequences. Here is an example:

```
aggClus.py -f psicub1_72373_psim.blast_PF05971.14.fa -d usearch -m 0.7 -x 0.7 -c 10
```

This script will output a `.clus` file of sequences and their cluster number, `.newick` file of a dendrogram of cluster relationships which can be opened in a phylogenetic tree viewer (it is not a phylogeny), and a `.dist` file of the distance matrix. You are interested in the cluster numbers/groups that contain your sequence of interest. Therefore, you can open the `.clus` file or the `.newick` file and search for your sequence. When you tune the cluster sizes to what you want, this can then be extracted as explained below. `-m` specifies the minimum percent identity for `usearch` to consider sequences as part of the same group - if sequences' percent identities are above this value then they will not be considered closely related. This is a value you will have to tune to change the size of your clusters. You will have to delete the `.dist` file to change `-m`. `-x` is the maximum distance between values to group them into a cluster - this is for the step after distance matrix calculation and can be tuned without deleting the `.dist` file. You will find that your cluster size is more sensitive to `-m`. `-d usearch` is specifying the distance matrix program. `aggClus.py` relies on a proprietary program, `usearch`, that you have to obtain a 32-bit license for (free). You will have to download it, obtain the license, and add it to your `~/.bash_profile` PATH... e.g. `export PATH=$PATH:/<PATH_TO_USEARCH_BINARY>`. 

Once you have the clusters that you want, you will have to extract the sequence names into a new-line delimited file that ONLY contains the sequences of interest. Think about using `cut` or `awk` here to do that automatically. Once you have this file of sequences, you can use `acc2fa.py` to obtain a fasta of them by referencing the MycotoolsDB:

`acc2fa.py -i <FILE_OF_SEQUENCE_HEADERS>`

<br />

Finally, you can proceed to phylogenetic construction via `fa2tree.py`.
