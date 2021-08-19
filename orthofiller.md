# OrthoFiller annotation completion

## Getting Started
[OrthoFiller](https://gitlab.com/xonq/OrthoFiller) fills missing genes in annotations by building profile models of orthogroups from closely related species and searching for these models in nonannotated regions of the assembly.

You will need at least one [annotation](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md). If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip installation to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/orthofiller.md#osc-use).


<br /><br />

## INSTALLING
 
Use [Singularity](https://gitlab.com/xonq/tutorials/-/blob/master/containers.md) (or Docker) to pull the prebuilt container. Alternatively, build a container from the [Dockerfile](https://gitlab.com/xonq/recipes/orthofiller).
```
singularity pull docker.io://xonq/orthofiller_latest.sif
```

Proceed to OSC USE and change hard coded paths as necessary.

<br /><br />

## OSC USE
#### Accessing OrthoFiller software
To use OrthoFiller software you will have to activate the software container. Press CTRL + D or run `exit` to exit. Only run the container for commands related to OrthoFiller or you may experience unexpected problems. NOTE - you will only have access to your home folder and shared project folders while in a container on OSC.

```
/fs/project/PAS1046/software/containers/orthofiller/orthofiller_latest.sif
source /fs/project/PAS1046/software/containers/orthofiller/source.sh
```

<br />

#### 1) Procure raw data
Gather at least 10 species' `.gff3` gene coordinate files and their corresponding assemblies. These can be downloaded from NCBI, JGI, or any other genome portal or you can grab them using the [database](https://gitlab.com/xonq/mycotools_scripts/-/blob/master/USAGE.md#dbfilespy). NOTE - this conversion works for NCBI, JGI, and Funannotate `.gff`s, but files from other sources, like Maker, may be incompatible. If the following command fails, try running `clean_gff.py <GFF3> > <CLEAN_GFF3>` in the container and reference `<CLEAN_GFF3>` in next command. If both fail, you will have to implement or find a solution.

remove spaces from all assembly fastas:
```
sed -r "s/>([^ ]+*) .*/>\1/g" <ASSEMBLY>.fa > <ASSEMBLY>.clean.fa
```

Activate the container and convert each `.gff3` to a `.gtf` compatible with
OrthoFiller. 
```
/fs/project/PAS1046/software/containers/orthofiller/orthofiller_latest.sif
gff_to_gtf_safe.py <YOUR/GFF> <OUTPUT/GTF>
```

NOTE - If you have a python3 environment, such as miniconda, you will have to deactivate it (`conda deactivate`) *AFTER* activating the container.

<br />

#### 2) Prepare input coordinate files
We will start OrthoFiller from scratch, however you can start/resume
OrthoFiller from a completed OrthoFinder run as well. OrthoFiller inputs a plain text
"targets" `.tsv` of organisms you want to *fill* and optionally a
reference `.tsv` of organisms you just want to reference and *not* fill. *DO
NOT INCLUDE REFERENCE ORGANISMS IN THE TARGET FILE*. Including more organisms than necessary in the target file will roughly double the computation time per organism added. Therefore, if you are finishing the annotation of one organism, then that organism should be the only one in the targets file and all others should be in the reference.

Create a plain text file for both the reference and target input (reference clean assemblies):
```
#gtf	genome
<YOUR/GTF1>	<YOUR/ASSEMBLY1>
<YOUR/GTF2>	<YOUR/ASSEMBLY2>
<YOUR/GTFn>	<YOUR/ASSEMBLYn>
```

<br />

#### 3) Run OrthoFiller
Create a plain text `.sh` file to execute OrthoFiller with the following information. The source command is necessary for Augustus, otherwise OrthoFiller will run to completion and not output results.
```
source /fs/project/PAS1046/software/containers/orthofiller/source.sh
orthofiller.py -r <YOUR/reference.tsv> -i <YOUR/target.tsv> -c 28 --fulloutput -o <SCRATCH/OUTPUT/DIRECTORY>
```
Allot 10-20 hours per target genome and invoke the container to execute the `.sh` file as a job:
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/orthofiller/orthofiller_latest.sif /bin/bash <YOUR/.sh>' | sbatch --time=20:00:00 --nodes=1 --ntasks-per-node=28 -A <PROJECT> --job-name=orthofiller
```

<br /><br />

## OUTPUT / CITATION INFO
OrthoFiller results are located in `<YOUR/OUTPUT>/results` and are fairly self explanatory. These data can be curated into a `.gff3` compatible with most downstream analyses by using the [`annotationCuration.py` script](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#curate-annotation).

This installation of OrthoFiller is derived from an update I made to modernize the software. Please cite the original OrthoFiller publication as well as the link to the updated software (gitlab.com/xonq/OrthoFiller).
