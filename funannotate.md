# Funannotate *de novo* annotation software setup and use

## NOTE 
- More information: https://funannotate.readthedocs.io/en/latest/install.html. 
- Fully capitalized paths, like `CONDA/INSTALLATION/PATH` need to be manually edited by the user.

To cleanly install Funannotate, we use a *Singularity container* (recommended) or an *environment manager*, Miniconda. Containers and environments keep software isolated so they do not interfere. Additionally, containers and miniconda allow one to install prepackaged software bundles - cutting out the time spent finding & installing the right program versions. Nevertheless, these processes often require some adjustment. Note that when you install software with an environment activated, you may create a dependency on your environment – in other words, the new software may rely on the environment being active. You can see which environment is active by looking at the `($ENVIRONMENT)` that precedes your `bash` header in your terminal window. If you wish to deactivate your environment or activate another, run `conda deactivate`. 



<br /><br />
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
##### - to activate and interact with the funannotate container:
```
singularity run ~/software/funannotate/funannotate_1.7.4.sif 
source ~/software/funannotate/fuannotate.sh
```
NOTE - You should only use the singularity container to run funannotate commands - to deactivate, simply press CTRL + D or execute `exit`. 

<br /> 

### 2A. Install external software. 
#### GeneMark
We are using an existing installation. You need to accept the licensing at http://topaz.gatech.edu/GeneMark/license_download.cgi and then run the code below. If you wish to install your own, you must follow the instructions on the website, and edit the genemark paths in `funannotate.sh`.

##### - copy a permissions key or download your own and place in home:
```
cp /users/PAS1046/osu10393/.gm_key ~/
```
NOTE - you cannot interact with other user folders when singularity is active, so remember to exit

<br />

### 3A. Check installation. 
*It’s okay* if `emapper.py`, `ete3`, and `signalp` are not installed/cause errors as we don’t need them now.

##### - check your installation:
```
funannotate check --show-versions
```
NOTE - remember to activate the singularity container to run this command

<br />

## USING FUNANNOTATE
### 1. Soft-mask assembly. 
You will need an assembly as well as a RepeatModeler library for your organism - `$OME-families.fa`

##### - We take the repeat library and soft mask the assembly by lowercasing masked nucleotides:
```
funannotate mask -i YOUR/ASSEMBLY -o OUTPUT/MASKED_ASSEMBLY_NAME -l YOUR/REPEATMODELER/$OME-families.fa
```

<br />

### 2. Gene prediction. 
You will have to download/compile a few pieces of data and information:
- find transcript/EST evidence from organisms in the same genus
- find protein evidence from at least 10 closely related organisms (more increases computation time)
- find the most closely related BUSCO database to your species by examining the databases stored in `/usr/local/config/species` in the singularity container or `/CONDA/INSTALLATION/PATH/envs/funannotate/config/species` in the miniconda environment. In the command below, cite the exact name of the species parameter folder, *not the full directory*

##### - Edit the command and submit the annotation job to OSC:
```
echo -e 'singularity run YOUR/FUNANNOTATE.sif && source YOUR/FUNANNOTATE_SETUP.sh -i YOUR/MASKED_ASSEMBLY -s “$OME_$RUN#” --transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE --protein_evidence YOUR/PROTEIN_EVIDENCE PATH/TO/uniprot_sprot.fasta –cpus 6 --busco_seed_species MOST_CLOSELY_RELATED_BUSCO_SPECIES -o OUTPUT/FOLDER' | qsub -l walltime=72:00:00 -l nodes=1:ppn=6 -o OUTPUT/FOLDER -N LOG_FILE_NAME -A PAS####
```

<br /><br />

### INSTAL OPTION 1B. miniconda 

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
