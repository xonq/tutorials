# Funannotate *de novo* annotation software setup and use

## NOTE 
- More information: https://funannotate.readthedocs.io/en/latest/install.html. 
- Fully capitalized paths, like `CONDA/INSTALLATION/PATH` need to be manually edited by the user.
- Step 3+ require the `funannotate` environment is active - `source activate funannotate`. 

Miniconda is an *environment manager*, which keeps software in isolated environments. Miniconda uses software *channels* with prepackaged software - this makes the installation of complex programs as simple as `conda install X`. Alas, sometimes - as evidenced here - the installation requires some minor adjustments. RepeatMasker, for example, cannot be installed by miniconda due to licensing and will need to be installed independently. Note that when you install software with an environment activated, you may create a dependency on your environment – in other words, the new software may rely on your environment, so it needs to be active to use the new software. If you wish to deactivate your environment or activate another, run `conda deactivate`. 



<br /><br />
## INSTALL
### 1. Install miniconda and setup download channels. 

##### - install miniconda3 and make your profile aware of its executable files:
```
wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p /CONDA/INSTALLATION/PATH/miniconda3
echo -e 'export PATH="/CONDA/INSTALLATION/PATH/miniconda3/bin:$PATH"' >> ~/.bash_profile
```
##### - add the package channels that contain software *in this order*:
```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```  
 
<br />

### 2. Create and downoad Funannotate environment. 
##### - create a new environment, `funannotate`, and download all software from the channels:
```
conda create -n funannotate python=2.7 funannotate
```
##### - *IF this does not complete*, create an empty environment and download software by submitting to OSC (EDIT The PAS#### with the project number):
```
conda create -n funannotate
echo -e 'source activate funannotate && conda install -y python=2.7 funannotate' | qsub -l walltime=10:00:00 -l nodes=1:ppn=1 -A PAS####
```
##### - Once installed, activate the environment
```
source activate funannotate
```

<br />

### 3. Install external software. 
#### GeneMark
I recommend using someone else’s installation. You need to accept the licensing at http://topaz.gatech.edu/GeneMark/license_download.cgi and then run the code below. If you wish to install your own, you must follow the instructions on the website.

##### - copy a permissions key:
```
cp /users/PAS1046/osu10393/.gm_key ~/
```

##### - add installed GeneMark to your funannotate conda environment:
```
echo “export GENEMARK_PATH=/users/PAS1046/osu9696/Software/gm_et_linux_64/gmes_petap” >> \ /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/activate.d/funannotate.sh
```


#### gmap
As of April 2020, the conda packaged gmap does not work, so we download it here

##### - download, extract, and navigate into the gmap folder:
```
wget http://research-pub.gene.com/gmap/src/gmap-gsnap-2020-03-12.tar.gz
tar -xzf gmap-gsnap-2020-03-12.tar.gz
cd gmap-2020-03-12
```
##### - install the new gmap to your funannotate conda environment:
```
./configure –-prefix /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate --exec-prefix \
/CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate
make check
make install
```

<br />

### 4. Install databases. 

##### - install databases:
```
funannotate setup -d /CONDA/INSTALLATION/PATH/miniconda3/databases
```
##### - add databases to funannotate conda environment configuration:
```
echo “export FUNANNOTATE_DB=/CONDA/INSTALLATION/PATH/miniconda3/databases” >> \
/CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/activate.d/funannotate.sh
echo “unset FUNANNOTATE_DB” > /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/deactivate.d/funannotate.sh
```

<br />

### 5. Check installation. 
*It’s okay* if `emapper.py`, `ete3`, and `signalp` are not installed/cause errors as we don’t need them now.

##### - check your installation:
```
funannotate check --show-versions
funannotate test -t all --cpus 8
```

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
- find the most closely related BUSCO database to your species by searching `/CONDA/INSTALLATION/PATH/envs/funannotate/config/species`. To use a lab BUSCO database you must copy it to the above directory. Cite the exact name of the species parameter folder, *not the full directory`* in the use command below

##### - Edit the command below and submit the annotation job to OSC:
```
echo -e ‘source activate funannotate && funannotate predict -i YOUR/MASKED_ASSEMBLY -s “$OME_$RUN#” \
--transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE --protein_evidence YOUR/PROTEIN_EVIDENCE \
MORE/PROTEIN_EVIDENCE /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/databases/uniprot_sprot.fasta –cpus 6 \
--busco_seed_species MOST_CLOSELY_RELATED_BUSCO_SPECIES -o OUTPUT/FOLDER | qsub -l walltime=72:00:00 \
-l nodes=1:ppn=6 -o OUTPUT/FOLDER -N LOG_FILE_NAME -A PAS####
```
