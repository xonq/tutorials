# OrthoFiller annotation completion

## Getting Started
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
To use OrthoFiller software you will have to activate the container. Press CTRL + D or run `exit` to exit. Only run the container for commands related to OrthoFiller or you may experience unexpected problems. NOTE - you will only have access to your home folder and shared project folders when in a container on OSC.

```
singularity run /fs/project/PAS1046/software/containers/orthofiller/orthofiller_latest.sif
```

<br />

#### 1) Procure raw data
Gather at least 10 species' `.gff3` gene coordinate files and their corresponding assemblies. These can be downloading from NCBI, JGI, or any other genome portal or you can grab them using the [database](https://gitlab.com/xonq/scripts/-/blob/master/README.md). NOTE - this conversion works for JGI `.gff` as well, but due to the incongruent nature of `.gff`, files from other sources, like Maker, may be incompatible.

Activate the container and convert each `.gff3` to a `.gtf` compatible with OrthoFiller.
```
singularity run /fs/project/PAS1046/software/containers/orthofiller/orthofiller_latest.sif
gff_to_gtf.py <YOUR/GFF> > <OUTPUT/GTF>
```

<br />

#### 2) Prepare reference files
We will start OrthoFiller from scratch, however you can start/resume OrthoFiller from a completed OrthoFinder run as well. OrthoFiller requires two plain text input files: a reference `.tsv` and a target/input `.tsv` for the annotations that you wish to "fill". Including more organisms than necessary in the target file will roughly double the computation time per organism added.

Create a plain text file for both the reference and target input:
```
#gtf	genome
<YOUR/GTF1>	<YOUR/ASSEMBLY1>
<YOUR/GTF2>	<YOUR/ASSEMBLY2>
<YOUR/GTFn>	<YOUR/ASSEMBLYn>
```

<br />

#### 3) Run OrthoFiller
Create a plain text `.sh` file to execute OrthoFiller with the following information:
```
orthofiller.py -r <YOUR/reference.tsv> -i <YOUR/target.tsv> -c 28 --fulloutput -o <YOUR/SCRATCH/OUTPUT>
```
Allot 10-20 hours per target genome and invoke the container to execute the `.sh` file as a job:
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/orthofiller/orthofiller_latest.sif /bin/bash <YOUR/.sh>' | qsub -l walltime=20:00:00 -l nodes=1:ppn=28 -A <PROJECT> -o orthofiller
```

<br /><br />

## OUTPUT
OrthoFiller results are located in `<YOUR/OUTPUT>/results` and are fairly self explanatory. These data can be curated into a `.gff3` compatible with most downstream analyses by using the [`annotationCuration.py` script](https://gitlab.com/xonq/scripts).
