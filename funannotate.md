# Funannotate *de novo* annotation software setup and use

## NOTE 
- More information: https://funannotate.readthedocs.io/en/latest/install.html. 
- Fully capitalized paths, like `CONDA/INSTALLATION/PATH` need to be manually edited by the user.
- Step 3+ require the `funannotate` environment is active - `source activate funannotate`. 

<br /><br />
## INSTALL
### 1. Install miniconda and setup download channels. 
Miniconda is an *environment manager*, which keeps software in isolated environments. Miniconda uses software *channels* with prepackaged software - this makes the installation of complex programs as simple as `conda install X`. Alas, it is not always this easy.

#### a) Here we install miniconda3 and make our profile aware of its executable files:
```
wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p /CONDA/INSTALLATION/PATH/miniconda3
echo -e 'export PATH="/CONDA/INSTALLATION/PATH/miniconda3/bin:$PATH"' >> ~/.bash_profile
```
#### b) We then add the package channels that contain software *in this order*
```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```  
 
<br />

### 2. Create and downoad Funannotate environment. 
Create a new environment, `funannotate`, and download all software in the package list `funannotate`. Once downloaded, if you wish to interface with these software you must activate the environment.

```
conda create -n funannotate python=2.7 funannotate
source activate funannotate
```

<br />

### 3. Install external software. 
Note that when you install software with an environment activated, you may create a dependency on your environment – in other words, the new software may rely on your environment, so it needs to be active to use the new software. If you wish to deactivate your environment or activate another, run `conda deactivate`. 

#### GeneMark
I recommend using someone else’s installation. You need to accept the licensing at http://topaz.gatech.edu/GeneMark/license_download.cgi and then run the code below. If you wish to install your own, you must follow the instructions on the website.

`cp /users/PAS1046/osu10393/.gm_key ~/`

`echo “export GENEMARK_PATH=/users/PAS1046/osu9696/Software/gm_et_linux_64/gmes_petap” >> \ /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/activate.d/funannotate.sh`


#### gmap
As of April 2020, the conda packaged gmap does not work, so you will have to install it manually and overwrite the existing version.

```
wget http://research-pub.gene.com/gmap/src/gmap-gsnap-2020-03-12.tar.gz
tar -xzf gmap-gsnap-2020-03-12.tar.gz
cd gmap-2020-03-12
```
```
./configure –-prefix /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate --exec-prefix \
/CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate
make check
make install
```

#### RepeatModeler and RepeatMasker
Installing RepeatModeler/RepeatMasker requires a tutorial of their own. For solid repeat masking, these software will need to be installed beforehand.

<br />

### 4. Install databases. 
Funannotate relies on a variety of databases. We need to download them and set our environment up to use them.

`funannotate setup -d /CONDA/INSTALLATION/PATH/miniconda3/databases`
```
echo “export FUNANNOTATE_DB=/CONDA/INSTALLATION/PATH/miniconda3/databases” >> \
/CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/etc/conda/activate.d/funannotate.sh
echo “unset FUNANNOTATE_DB” > /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/deactivate.d/funannotate.sh
```

<br />

### 5. Check installation. 
Funannotate provides tools to check installation software and test the installation. *It’s okay* if `emapper.py`, `ete3`, and `signalp` are not installed as we don’t need them now.

`funannotate check --show-versions`
`funannotate test -t all --cpus 8`

<br /><br />

## USE
### 0. Beforehand. 
You will need an assembly via SPAdes, or Funannotate’s assembly pipeline as well as a RepeatModeler library for your organism - `$OME-families.fa`

<br />

### 1. Soft-mask assembly. 
This will create a new assembly with repeat nucleotides lower-cased.

`funannotate mask -i YOUR/ASSEMBLY -o OUTPUT/MASKED_ASSEMBLY_NAME -l YOUR/REPEATMODELER/$OME-families.fa`

<br />

### 2. Gene prediction. 
Congratulations on making it this far. Here is where we run the annotation. Use the most closely related BUSCO species. Options can be found in `/CONDA/INSTALLATION/PATH/envs/funannotate/config/species`. If you want to use a species parameter file the lab generated previously, you must copy it to the above directory. Cite the exact name of the species parameter folder below

```
echo -e ‘source activate funannotate && funannotate predict -i YOUR/MASKED_ASSEMBLY -s “$OME_$RUN#” \
--transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE --protein_evidence YOUR/PROTEIN_EVIDENCE \
MORE/PROTEIN_EVIDENCE /CONDA/INSTALLATION/PATH/miniconda3/envs/funannotate/databases/uniprot_sprot.fasta –cpus 6 \
--busco_seed_species MOST_CLOSELY_RELATED_BUSCO_SPECIES -o OUTPUT/FOLDER | qsub -l walltime=72:00:00 \
-l nodes=1:ppn=6 -o OUTPUT/FOLDER -N LOG_FILE_NAME -A PAS####
```
