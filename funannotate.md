# Funannotate *de novo* annotation software setup and use

## GETTING STARTED
[Slot Lab Annotation Pipeline](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md)

[Funannotate](https://funannotate.readthedocs.io/en/latest/install.html) is an all-in-one genome annotation software that wraps and curates output from multiple *ab initio* gene prediction software.

Only two external inputs are necessary: [a *de novo* assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md) and [a *de novo* Repeat library fasta](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md). If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip installation to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use) once you have read the [pipeline introduction](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#getting-started).

<br /><br /><br />

## INSTALLING
#### Install GeneMark

Accept the license for [GeneMark-ES/ET/EP](https://topaz.gatech.edu/GeneMark/license_download.cgi), download the program, and transfer the use key to your home directory as `.gm_key`

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
GeneMark is one of the *ab initio* gene prediction software Funannotate calls. Upon first use or every 400 days, accept the license and download the 64-bit key (NOT THE PROGRAM) at this link for [GeneMark-ES/ET/EP](http://topaz.gatech.edu/GeneMark/license_download.cgi). Then we will transfer the key to your home directory. These keys expire every 400 days.

Once the key is uploaded, uncompress it and rename it in your home folder as `.gm_key`:
```
gunzip gm_key_64.gz
mv gm_key_64 ~/.gm_key
```

Let's get started with gene prediction. First we have to interface with Funannotate. Like other parts of the pipeline, Funannotate is in *container* of software. Furthermore, Funannotate also requires that we run a `source` command upon activating the container - this adds software to our `PATH` which essentially is telling the computer the default places to look for files. Only use the container to run Funannotate commands, otherwise you will experience unexpected behavior. To deactivate the container, press CTRL + D or run `exit`. 
```
singularity run /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate --help
```

If you entered it all correct and got an error above then stop and contact me.

<br /><br />

### 0. Keep it clean
I am once again reminding you how important it is to keep your directory structures clean. Following what we have done in previous tutorials, we are ultimately going to output to our `<ORGANISM_CODENAME>` directory.

```
mkdir <ORGANISM_CODENAME>/funannotate_prep
```

<br /><br />

### 1. Sort assembly
Sort your assembly: this renames contigs to `scaffold`, removes contigs below a minimum length (1 kb), and sorts from largest to smallest. Remember to [activate and source](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#activating-funannotate-container) the container to have access to Funannotate. This command can be run without submitting a job. If you followed the previous tutorial, your assembly will be in `<ORGANISM_CODENAME>/results`:
```
funannotate sort -i <YOUR/ASSEMBLY> -o <ORGANISM_CODENAME>/funannotate_prep/ASSEMBLY.sort.fa> --minlen 1000
```

<br /><br />

### 2. Soft-mask assembly 

Soft-mask the assembly: this references your [*de novo* repeat library](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md) to lowercase masked nucleotides. If you followed the previous tutorials' directory structure, the file will be in `<ORGANISM_CODENAME>/results`. 

We are now going to submit a job that invokes the singularity container, so we need to write the shell, `.sh` execution script in plain-text UTF-8 format and save it as something meaningful like `<ORGANISM>_mask.sh`.
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate mask -i <YOUR/SORTED_ASSEMBLY> -m repeatmodeler \
-l <YOUR/REPEAT_LIBRARY>-o <ORGANISM_CODENAME/funannotate_prep/ASSEMBLY.masked.fa --cpus 6
```

If you make the `.sh` file on your local computer, you'll have to upload it to the supercomputer. Then submit to the job node by invoking singularity to reference your `.sh` file like so:
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif /bin/bash <YOUR/SH_FILE>' | sbatch --time=10:00:00 --nodes=1 --ntasks-per-node=6 -A <PROJECT> --job-name=softmask
```

<br /><br />

### 3. Predict genes
Download/compile necessary data and information:
- transcript/EST evidence from organism(s) in the same genus (JGI/GenBank)
- protein evidence from at least 10 closely related organisms (separate by spaces in command)
- run `funannotate species` (container must be active) to find the most closely related BUSCO species database; funannotate will generate a BUSCO species database for your species; funannotate will create and add a BUSCO species database for your organism

Once compiled, create an output directory in your scratch folder to avoid file limits:
```
mkdir /fs/scratch/<PAS###>/<USER>/funannotate_<NAME>
```

Now we are ready for gene prediction. Create a UTF-8 file with the predict command, save as an `.sh`, and transfer to OSC. This command first sources the environment for the container, then tells funannotate to input your masked assembly, a name for the annotation run, transcript evidence, protein evidence, the number of CPU cores you use, the BUSCO species database, and the output folder you made above:
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

funannotate predict -i <YOUR/MASKED_ASSEMBLY> -s “<OME_RUN#>” --transcript_evidence <YOUR/TRANSCRIPT_AND_EST_EVIDENCE> \
--protein_evidence <YOUR/PROTEIN_EVIDENCE> /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
-–cpus 8 --busco_seed_species <BUSCO_SPECIES> -o <OUTPUT/FOLDER>
```

<br />

Edit the below to submit the `.sh` file as a job in the funannotate container. Remember, you don't need to activate the container to submit a job that uses the container.
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMDFILE>' | sbatch --time=60:00:00 --nodes=1 --ntasks-per-node=8 -A PAS<####> --job-name=funannotate
```

<br /><br />

## OUTPUT
Funannotate will output many working files and an organized folder of results. Let's just move those to your clean directory.
```
mv /fs/scratch/<PAS####>/<USER>/funannotate/predict_results <OUTPUT>/<ORGANISM_CODENAME>/results
```

The naming scheme of the output directory is intuitive: the gene coordinate file is the `gff3`, the protein fasta is `proteins.aa.fasta`, etc etc. You can obtain a summary of your annotation statistics using the `annotationStats.py` [script](https://gitlab.com/xonq/scripts).

<br /><br />

### All at once
Once you have tested the individual steps of Funannotate and are familiar with this portion of the pipeline, you can submit all the commands in one `.sh` file to move a genome through all steps at once. Here is the skeleton of such a file:
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

# sort and remove contigs < 1000 bp
funannotate sort -i <YOUR/ASSEMBLY> --minlen 1000 -o <SORTED/ASSEMBLY>


# soft-mask nucleotides
funannotate mask -i <SORTED/ASSEMBLY> -m repeatmodeler -l <YOUR/REPEAT-LIBRARY.fa> -o <MASKED/ASSEMBLY>


# gene prediction
funannotate predict -i <MASKED/ASSEMBLY> -s "<OME_RUN#>" --transcript_evidence <YOUR/EVIDENCE> \
--protein_evidence <YOUR/EVIDENCE1> <YOUR/EVIDENCEn> /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
--cpus 8 --busco_seed_species <BUSCO_SPECIES> -o <OUTPUT/FOLDER>
```

Save as an `.sh`, transfer to OSC, and submit the script as previously described (allocate enough resources)

<br /><br />

## CITATION INFO

Funannotate is the `wrapper` that packages many other software. In addition to citing Funannotate, you should cite `GlimmerHMM`, `GeneMark ES`, `Snap`, `BUSCO`, and `Augustus`.
