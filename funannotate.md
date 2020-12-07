# Funannotate *de novo* annotation software setup and use

## GETTING STARTED
[Funannotate documentation](https://funannotate.readthedocs.io/en/latest/install.html)

[Slot Lab Annotation Pipeline](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md)


Only two external inputs are necessary: [a *de novo* assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md) and [a *de novo* Repeat library fasta](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md). If you are using the Ohio Super Computer (OSC) and have access to PAS1046, you can skip installation and jump straight to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use).

<br /><br /><br />

## INSTALLING
#### Install GeneMark

Accept the license for [GeneMark-ES/ET/EP](https://topaz.gatech.edu/GeneMark/license_download.cgi), download the program, and transfer the use key to your home directory as `.gm_key`**

<br />

#### Pull the Funannotate Container
 
Use [Singularity](https://gitlab.com/xonq/tutorials/-/blog/master/containers.md) (or Docker) to pull the prebuilt Funannotate container
```
singularity pull docker://xonq/funannotate_mask:1.8.1
```

Install databases and exit the container
```
singularity run <CONTAINER.sif>
funannotate setup -d <DATABASE/DIRECTORY>
exit
```

Create a UTF-8 text file with the following information, fill it in, and save it in the folder with the container `.img` as `source.sh`. Everytime you wish to use the container, you must run `source <PATH/TO>/source.sh` to point Funannotate to the binaries.
```
# activate funannotate software
. /opt/conda/etc/profile.d/conda.sh
conda activate base

# export path dependencies for databases, genemark, and augustus/busco species databases
export FUNANNOTATE_DB=		# path to funannotate database
export GENEMARK_PATH=		# path to genemark program directory
#export AUGUSTUS_CONFIG_PATH=	# path to augustus species configuration (only specify if using a non-default one)
export PATH=$PATH:		# path to genemark program directory
```

<br />

Continue to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use) and follow instructions, changing path references when necessary

<br /><br /><br />

## OSC USE
#### First use
First, run this command to make your profile compatible with Funannotate:
```
echo -e 'export SINGULARITY_BINDPATH="/opt:/tmp"' >> ~/.bash_profile
```

Next, accept the license for [GeneMark-ES/ET/EP](http://topaz.gatech.edu/GeneMark/license_download.cgi), download the 64-bit key (NOT the program), and transfer to your OSC home directory. These keys expire every 400 days.

uncompress the key and place it in your home folder as `.gm_key`
```
gunzip gm_key_64.gz
mv gm_key_64 ~/.gm_key
```

Relogin and test Funannotate as described below: To interface with Funannotate in the login node you activate a *container* of software and then run a `source` command to add the software to our `PATH`. Only use the container to run Funannotate commands, otherwise you will experience unexpected behavior. To deactivate the container, press CTRL + D or run `exit`. 
```
singularity run /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
source /fs/project/PAS1046/software/containers/funannotate/source.sh
```

<br /><br />

### 1. Sort assembly
Sort the assembly: this renames contigs to `scaffold`, removes contigs below a minimum length (1 kb), and sorts from largest to smallest. Remember to [activate and source](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#activating-funannotate-container) the container to have access to Funannotate. This command can be run without submitting a job.
```
funannotate sort -i <YOUR/ASSEMBLY> -o <OUTPUT/SORTED_ASSEMBLY> --minlen 1000
```

<br /><br />

### 2. Soft-mask assembly 

Soft-mask the assembly: this references your [*de novo* repeat library](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md) to lowercase masked nucleotides. This will be submitted as a job, which is slightly different for `singularity` containers. 
NOTE - *You do not want the container to be activated when submitting a job*. Instead, you first create a `UTF-8` plain-text file with the information below and save it as a `.sh` file, like `funmask.sh`. 
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate mask -i <YOUR/ASSEMBLY> -m repeatmodeler -l <YOUR/REPEAT_LIBRARY> -o <OUTPUT/FILENAME>
```

If you make the `.sh` file on your local computer, you'll have to upload it to the supercomputer. Then submit to the job node by invoking singularity to reference your `.sh` file like so:
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif /bin/bash <YOUR/SH_FILE>' | qsub -l walltime=10:00:00 -l nodes=1:ppn=1 -A <PROJECT>
```

<br /><br />

### 3. Predict genes
Download/compile necessary data and information:
- transcript/EST evidence from organism(s) in the same genus
- protein evidence from at least 10 closely related organisms (separate by spaces in command) and the uniprot database.
- run `funannotate species` to find the most closely related BUSCO species database; funannotate will generate a BUSCO species database for your species; funannotate will create and add a BUSCO species database for your organism

create a UTF-8 file with the predict command, save as an `.sh`, and transfer to OSC:
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

funannotate predict -i <YOUR/MASKED_ASSEMBLY> -s “<OME_RUN#>” --transcript_evidence <YOUR/TRANSCRIPT_AND_EST_EVIDENCE> \
--protein_evidence <YOUR/PROTEIN_EVIDENCE> /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
-–cpus 8 --busco_seed_species <BUSCO_SPECIES> -o <OUTPUT/FOLDER>
```

<br />

Edit and submit a job to run that file in the funannotate container. Remember, you don't need to activate the container to submit a job that uses the container.
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMDFILE>' | qsub -l walltime=60:00:00 -l nodes=1:ppn=8 -A PAS<####> -N funannotate
```

<br /><br />

### All at once
Once you have tested the individual steps of Funannotate and are familiar with the pipeline, you can submit all the commands in one `.sh` file to move a genome through all steps at once. Here is the skeleton of such a file:
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

# sort and remove contigs < 1000 bp
funannotate sort -i <YOUR/ASSEMBLY> --minlen 1000 -o <SORTED/ASSEMBLY>


# soft-mask nucleotides
funannotate mask -i <CLEAN/ASSEMBLY> -m repeatmodeler -l <YOUR/REPEAT-LIBRARY.fa> -o <MASKED/ASSEMBLY>


# gene prediction
funannotate predict -i <MASKED/ASSEMBLY> -s "<OME_RUN#>" --transcript_evidence <YOUR/EVIDENCE> \
--protein_evidence <YOUR/EVIDENCE1> <YOUR/EVIDENCEn> /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
--cpus 8 --busco_seed_species <BUSCO_SPECIES> -o <OUTPUT/FOLDER>
```

Save as an `.sh`, transfer to OSC, and submit the script as previously described (allocate enough resources)
