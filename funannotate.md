# Funannotate *de novo* annotation software setup and use

## NOTE 
- More information: https://funannotate.readthedocs.io/en/latest/install.html. 
- Fully capitalized paths, like `CONDA/INSTALLATION/PATH` need to be manually edited by the user.

If you are accessing via OSC and have access to PAS1046, you should be able to run Funannotate out of the box.
To cleanly install Funannotate, we use a *Singularity container* (recommended) or an *environment manager*, Miniconda. Containers and environments keep software isolated so they do not interfere. Additionally, containers and miniconda allow one to install prepackaged software bundles - cutting out the time spent finding & installing the right program versions. Nevertheless, these processes often require some adjustment.

<br />

## OSC USE
#### Accessing GeneMark
Accept the licensing at http://topaz.gatech.edu/GeneMark/license_download.cgi, download a key, and transfer in your home directory as `~/.gm_key`. These expire every 400 days and will cause GeneMark errors.


#### Accessing Funannotate
Whenever you want to use Funannotate run the following. Only use the environment to run Funannotate commands. To deactivate press CTRL + D or run `exit`. To submit an OSC job you will have to create a `.sh` to run within the container. See 2. Gene Prediction.

##### - activate singularity container and setup environment
```
singularity run /fs/project/PAS1046/software/funannotate_1.7.4/funannotate_1.7.4.sif
source /fs/project/PAS1046/software/funannotate_1.7.4/source.sh
```

<br />

### 1. Soft-mask assembly. 
You will need an assembly as well as a RepeatModeler library for your organism - `$OME-families.fa`

##### - We take the repeat library and soft mask the assembly by lowercasing masked nucleotides:
```
funannotate mask -i YOUR/ASSEMBLY -o OUTPUT/MASKED_ASSEMBLY_NAME -l YOUR/REPEATMODELER/$OME-families.fa
```
NOTE - access the container to run the command; you will not need to submit an OSC job

<br />

### 2. Gene prediction. 
You will have to download/compile a few pieces of data and information:
- find transcript/EST evidence from organism(s) in the same genus
- find protein evidence from at least 10 closely related organisms (separate by spaces in command)
- find the most closely related BUSCO database to your species by examining the databases stored in `/usr/local/config/species` in the singularity container (REMEMBER: activate to use).  In the command below, cite the exact name of the species parameter folder, *not the full directory*

##### - create a text file with the following info, name it with `.sh`, and transfer to OSC:
```
source /fs/project/PAS1046/software/funannotate_1.7.4/source.sh

funannotate predict -i YOUR/MASKED_ASSEMBLY -s “$OME_$RUN#” --transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE \
--protein_evidence YOUR/PROTEIN_EVIDENCE PATH/TO/uniprot_sprot.fasta –cpus 6 --busco_seed_species \
MOST_CLOSELY_RELATED_BUSCO_SPECIES -o OUTPUT/FOLDER
```
NOTE - the source command exports the environment variables necessary for funannotate

#### - submit a job to Torque to run that file in the funannotate container
```
echo -e 'singularity exec /fs/project/PAS1046/software/funannotate_1.7.4/funannotate_1.7.4.sif bash YOUR/FILE.sh' | qsub -l walltime=60:00:00 -l nodes=1:ppn=8 -A PAS####
```

<br /><br /><br />

## IF INSTALLING ON YOUR OWN:
### INSTALL OPTION 1A. *singularity* - recommended
##### - make folder and pull funannotate singularity container:
```
mkdir -p ~/software/funannotate
singularity pull --name ~/software/funannotate/funannotate_1.7.4.sif docker://quay.io/biocontainers/funannotate:1.7.4--py27h864c0ab_1
```
##### - copy the setup script
```
cp /users/PAS1046/osu10393/program/singularity/funannotate/funannotate.sh ~/software/funannotate
```
##### - activate and interact with the funannotate container:
```
singularity run ~/software/funannotate/funannotate_1.7.4.sif 
source ~/software/funannotate/fuannotate.sh
```
NOTE - Only use the singularity container to run funannotate commands - to deactivate, simply press CTRL + D or use `exit`. 

<br /> 

### 2A. Install external software. 
#### GeneMark
We are using a local installation. Accept the licensing at http://topaz.gatech.edu/GeneMark/license_download.cgi and then run the code below. If you wish to install your own, you must follow the instructions on the website, and edit the genemark paths in `funannotate.sh`.

##### - copy a permissions key / download your own and place in home then copy GeneMark:
```
cp /users/PAS1046/osu10393/.gm_key ~/
cp /users/PAS1046/osu10393/program/gmes_petap ~/software/gmes_petap
```
NOTE - you cannot interact with other user folders when singularity is active, so remember to exit

<br />

### 3A. Check installation. 

##### - check your installation:
```
funannotate check --show-versions
```
NOTE - remember to activate the singularity container

NOTE - Errors for `emapper.py`, `ete3`, and `signalp` are okay for *de novo* annotation.

<br /><br />

### INSTALL OPTION 1B. miniconda (ERRORS: 09/24/2020)

##### - install miniconda3 and make your profile aware of its executable files:
```
wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p /CONDA/INSTALLATION/PATH/miniconda3
echo -e 'export PATH="/CONDA/INSTALLATION/PATH/miniconda3/bin:$PATH"' >> ~/.bash_profile
```
##### - add the package channels *in this order*:
```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```  
 
##### - create a new environment, `funannotate`, and download all software from the channels:
```
conda create -n funannotate python=2.7 funannotate
```
*IF this does not complete*, try OPTION 1.
##### - Once installed, activate the environment
```
source activate funannotate
```

<br />

### 2B. Install external software. 
#### GeneMark
Install via the 2A instructions and additionally enter the following commands.

#### - Add genemark to your environment path
```
echo “export GENEMARK_PATH=/users/PAS1046/osu9696/Software/gm_et_linux_64/gmes_petap” >> /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/activate.d/funannotate.sh
```

<br />

### 3B. Install databases. 
We bypass this in the singularity installation by using my installed databases

##### - install databases:
```
funannotate setup -d /CONDA/INSTALLATION/PATH/miniconda3/databases
```

##### - add databases to the environment configuration:
```
echo “export FUNANNOTATE_DB=/CONDA/INSTALLATION/PATH/miniconda3/databases” >> /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/activate.d/funannotate.sh
echo “unset FUNANNOTATE_DB” > /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/deactivate.d/funannotate.sh
```

<br />

### USING VIA MINICONDA
##### - follow the USE instructions above with the following command:
```
echo -e 'source activate funannotate && funannotate predict -i YOUR/MASKED_ASSEMBLY -s “$OME_$RUN#” --transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE --protein_evidence YOUR/PROTEIN_EVIDENCE /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/databases/uniprot_sprot.fasta –cpus 6 --busco_seed_species MOST_CLOSELY_RELATED_BUSCO_SPECIES -o OUTPUT/FOLDER' | qsub -l walltime=72:00:00 -l nodes=1:ppn=6 -o OUTPUT/FOLDER -N LOG_FILE_NAME -A PAS####
```
