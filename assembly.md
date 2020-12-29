# SPAdes *de novo* genome assembly

## GETTING STARTED 
Assembly software takes next generation reads and uses models to guide stitching the reads into contiguous sequences, or contigs. Some software go a step further and attempt to stich the contigs themselves into scaffolds.

If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip installation to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md#osc-use) once you have added the line to your profile from [here](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#getting-started).

<br />

## INSTALLATION
Adapter trimming software has to be installed independently. For Illumina, `trimmomatic` works well and Nanopore base calling software seems to do a decent enough job these days.

#### Pulling SPAdes container
```
singularity pull docker://quay.io/biocontainers/spades:3.14.1--h2d02072_0
```

Proceed to OSC USE and adjust paths and job submission commands as necessary

<br /><br />

## OSC USE
### 1. Trimming Illumina paired-end reads
Find the adapter relevant to your sequence data and edit the following job submission command with your information:
```
echo -e 'cd </OUTPUT/FOLDER> && java -jar /fs/project/PAS1046/software/Trimmomatic-0.36/trimmomatic-0.36.jar \
PE -phred33 </PATH/TO/READ_1> </PATH/TO/READ_2> <OUTPUT_NAME_fpaired.fq.gz> <OUTPUT_NAME_funpaired.fq.gz> \
<OUTPUT_NAME_rpaired.fq.gz> <OUTPUT_NAME_runpaired.fq.gz> ILLUMINACLIP:/fs/project/PAS1046/software/Trimmomatic-0.3.6/adapters/TruSeq3-PE.fa:2:30:10:11 \
HEADCROP:10 CROP:145 SLIDINGWINDOW:50:25 MINLEN:100' | sbatch --time=01:00:00 --nodes=1 --ntasks-per-node=12 -A PAS####
```

<br />

### 2. Assembling
#### Accessing SPAdes assembler
When using SPAdes, you have to activate the SPAdes singularity container, which gives you access to SPAdes software. Only use the container to run the software within it, otherwise you will experience weird errors. Also, the container should NOT be active when submitting jobs that use the container - the procedure for job submission is explained below. To exit the container, press `CTRL` + `D`, or type `exit`.
```
singularity run /fs/project/PAS1046/software/containers/spades/spades_3.14.1.sif
```

Activate the container and run `spades.py --help` for more information relevant to your use case. You do not need the container active for the rest of the tutorial.

#### EXAMPLE1: Illumina 150 bp paired-end
Create a plain text (UTF-8) file with the following command, save it as an `.sh` file, and transfer to OSC:

```
spades.py --pe-1 1 </PATH/TO/TRIMMED/F_paired.fq.gz> --pe-2 1 </PATH/TO/TRIMMED/R_paired.fq.gz -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir </PATH/TO/SCRATCH> -t 6 -m 144 -o </PATH/TO/OUTPUT> --isolate
```

#### EXAMPLE2: Illumina 150 bp paired-end + Nanopore long read
Create a plain text (UTF-8) file with the following command, save it as an `.sh` file and transfer to OSC:

```
spades.py --pe-1 1 </PATH/TO/TRIMMED/F_paired.fq.gz> --pe-2 1 </PATH/TO/TRIMMED/R_paired.fq.gz> --nanopore </PATH/TO/NANOPORE_READS> -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir </PATH/TO/SCRATCH> -t 6 -m 144 -o </PATH/TO/OUTPUT> --isolate
```

<br />

#### SUBMITTING A JOB SCRIPT
Edit the below and submit a job to Torque job management - you do not need the container active to submit the job.
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/spades/spades_3.14.1.sif bash <YOUR/CMD>' | sbatch --time=100:00:00 --nodes=1 --ntasks-per-node=6 -A PAS<####>
```

<br /><br />

## OUTPUT
SPAdes will output many files, but ultimately the `scaffolds.fa` is the assembly that attempts to stitch contigs together as best as possible and is what is used for [gene prediction](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#3.-predict-genes). 
