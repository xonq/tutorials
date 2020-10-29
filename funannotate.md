# Funannotate *de novo* annotation software setup and use

## NOTE 
- [Funannotate documentation](https://funannotate.readthedocs.io/en/latest/install.html)
- [Docker container](https://hub.docker.com/r/xonq/funannotate_mask/tags) - requires exporting dynamic variables upon startup

If you are using OSC and have access to PAS1046, you should be able to run Funannotate after acquiring a GeneMark use key.

<br />

## Prerequisites
- [Assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md) - `scaffolds.fasta`
- [Repeat library fasta](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md) - `consensi.fa`

<br />

## INSTALLING
Skip to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use) if you are using the Ohio SuperComputer

#### Installing GeneMark

**- Accept the license for [GeneMark-ES/ET/EP](https://topaz.gatech.edu/GeneMark/license_download.cgi), download the program, and transfer the use key to your home directory as `.gm_key`**

#### Pulling the Funannotate Container
 
**- Use [Singularity](https://gitlab.com/xonq/tutorials/-/blog/master/containers.md) (or Docker) to pull the prebuilt Funannotate container**
```
singularity pull docker://xonq/funannotate_mask:1.8.1
```

**- Install databases**
```
singularity run <CONTAINER.sif>
funannotate setup -d <DATABASE/DIRECTORY>
```
NOTE - only use the container to run Funannotate commands, press CTRL + D or type `exit` to exit

**- Create an environment source file that points Funannotate to the appropriate directories and save as source.sh**
`source.sh`:
```
# export path dependencies for databases, genemark, and augustus/busco species databases
export FUNANNOTATE_DB=		# path to funannotate database
export GENEMARK_PATH=		# path to genemark program directory
#export AUGUSTUS_CONFIG_PATH=	# path to augustus species configuration (only specify if using a non-default one)
export PATH=$PATH:		# path to genemark program directory
```
NOTE - whenever you want to use Funannotate, you must run `source source.sh` to export these variables

**- Continue to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use) and follow instructions, changing path references when necessary**

<br />

## OSC USE
#### Accessing GeneMark
Accept the license for [GeneMark-ES/ET/EP](http://topaz.gatech.edu/GeneMark/license_download.cgi), download the 64-bit key (NOT the program), and transfer to your OSC home directory. 

**- uncompress and rename**
```
gunzip gm_key_64.gz
mv gm_key_64 ~/.gm_key
```

NOTE - Keys expire in 400 days and will cause GeneMark errors.

<br />

#### Activating Funannotate Container

**- activate container then source the Funannotate directories to your path**
```
singularity run /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
source /fs/project/PAS1046/software/containers/funannotate/source.sh
```

NOTE - Only use to run the container software. To deactivate press CTRL + D or run `exit`.

<br />

### 1. Clean assembly
Clean by removing contigs < 1000 bp and with 95% identity to any contig less than the N50.

**- create a plain text (UTF-8) file with the clean command, save as an `.sh` file, and transfer to OSC**
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate clean -i <YOUR/ASSEMBLY> -o <OUTPUT/FILENAME> -m 1000
```

**- edit and submit a job to Torque to run that file in the funannotate container**
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMDFILE>' | qsub -l walltime=30:00:00 -l nodes=1:ppn=6 -A PAS<####> -N clean
```

<br />

### 2. Soft-mask assembly 

**- soft-mask the assembly by using RepeatMasker to lowercase masked nucleotides**
```
funannotate mask -i <YOUR/ASSEMBLY> -m repeatmodeler -l <YOUR/REPEAT_LIBRARY> -o <OUTPUT/FILENAME>
```
NOTE - the container needs to be active and sourced

<br />

### 3. Predict genes
Download/compile necessary data and information:
- transcript/EST evidence from organism(s) in the same genus
- protein evidence from at least 10 closely related organisms (separate by spaces in command)
- run `funannotate species` to find the most closely related BUSCO species database; funannotate will generate a BUSCO species database for your species; funannotate will create and add a BUSCO species database for your organism

**- create a file with the predict command, save as an `.sh`, and transfer to OSC:**
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

funannotate predict -i <YOUR/MASKED_ASSEMBLY> -s “<OME_RUN#>” --transcript_evidence <YOUR/TRANSCRIPT_AND_EST_EVIDENCE> \
--protein_evidence <YOUR/PROTEIN_EVIDENCE> /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
-–cpus 8 --busco_seed_species <BUSCO_SPECIES> -o <OUTPUT/FOLDER>
```

**- edit and submit a job to Torque to run that file in the funannotate container**
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMDFILE>' | qsub -l walltime=60:00:00 -l nodes=1:ppn=8 -A PAS<####> -N funannotate
```
NOTE - you do not need to submit with the container active

<br /><br />

### All at once
Here is the skeleton of a text file that can run through all the previous steps
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

# remove contigs < 1000 bp and duplicates < N50 size
funannotate clean -i <YOUR/ASSEMBLY> -m 1000 -o <CLEAN/ASSEMBLY>


# soft-mask nucleotides
funannotate mask -i <CLEAN/ASSEMBLY> -m repeatmodeler -l <YOUR/REPEAT-LIBRARY.fa> -o <MASKED/ASSEMBLY>


# gene prediction
funannotate predict -i <MASKED/ASSEMBLY> -s "<OME_RUN#>" --transcript_evidence <YOUR/EVIDENCE> \
--protein_evidence <YOUR/EVIDENCE1> <YOUR/EVIDENCEn> /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
--cpus 8 --busco_seed_species <BUSCO_SPECIES> -o <OUTPUT/FOLDER>
```

Save as an `.sh`, transfer to OSC, and submit the script as previously described (allocate enough resources)
