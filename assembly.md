# SPAdes *de novo* genome assembly

## NOTE
If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md#osc-use).

<br />

## INSTALLATION
#### Pulling SPAdes container
```
singularity pull docker://quay.io/biocontainers/spades:3.14.1--h2d02072_0
```

proceed to OSC USE and adjust paths and job submission commands as necessary

<br /><br />

## OSC USE
#### Accessing SPAdes assembler
##### - activate singularity container
```
singularity run /fs/project/PAS1046/software/containers/spades/spades_3.14.1.sif
```
NOTE - only run SPAdes commands with the container active, press CTRL + D or run `exit` to deactivate

<br />

### 1. Trimming Illumina paired-end reads
##### - edit the following command with your information:
```
echo -e 'cd </OUTPUT/FOLDER> && java -jar /fs/project/PAS1046/software/Trimmomatic-0.36/trimmomatic-0.36.jar \
PE -phred33 </PATH/TO/READ_1> </PATH/TO/READ_2> <OUTPUT_NAME_fpaired.fq.gz> <OUTPUT_NAME_funpaired.fq.gz> \
<OUTPUT_NAME_rpaired.fq.gz> <OUTPUT_NAME_runpaired.fq.gz> ILLUMINACLIP:/fs/project/PAS1046/software/Trimmomatic-0.3.6/adapters/TruSeq3-PE.fa:2:30:10:11 \
HEADCROP:10 CROP:145 SLIDINGWINDOW:50:25 MINLEN:100' | qsub -l walltime=01:00:00 -l nodes=1:ppn=12 -A PAS#### \
-N trimmomatic
```
NOTE - you may have to use a different adapter set in the adapters folder

<br />

### 2. Assembling
Activate the container and run `spades.py --help` for more information relevant to your use case.

#### EXAMPLE1: Illumina 150 bp paired-end
##### - create a plain text (UTF-8) file with the following command, save it as an `.sh` file, and transfer to OSC:

```
spades.py --pe1-1 </PATH/TO/TRIMMED/F_READS> -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir </PATH/TO/SCRATCH> -t 6 -m 144 -o </PATH/TO/OUTPUT>
```

#### EXAMPLE2: Illumina 150 bp paired-end + Nanopore long read
##### - create a plain text (UTF-8) file with the following command, save it as an `.sh` file and transfer to OSC:

```
spades.py --pe1-1 </PATH/TO/TRIMMED/F_READS.fq.gz> --pe1-2 </PATH/TO/TRIMMED/R_READS.fq.gz> --nanopore </PATH/TO/NANOPORE_READS> -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir </PATH/TO/SCRATCH> -t 6 -m 144 -o </PATH/TO/OUTPUT>
```

<br />

##### - edit and submit a job to Torque
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/spades/spades_3.14.1.sif bash <YOUR/CMD>' | qsub -l walltime=100:00:00 -l nodes=1:ppn=6 -A PAS<####> -N spades
```
NOTE - you do not need to submit with the container active
