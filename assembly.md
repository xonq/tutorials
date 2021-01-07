# SPAdes *de novo* genome assembly

## GETTING STARTED 
Assembly software takes next generation reads and stitches them into contiguous sequences (contigs). Some software go a step further and attempt to stich the contigs themselves (scaffolds).

If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip installation to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md#osc-use) once you have read the [pipeline introduction](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#getting-started).

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
### 0. Keep it clean
Imagine a pile of dishes from that shitty roommate in undergrad... don't let your folder get like that. Prepare a directory where you will make your output. If you think about your directory organization before the analysis you will have an easier time returning and disseminating your data. I personally will create one directory for the project, one for the organism, then one directory for each step of the pipeline. e.g. (use whatever folder you like as the `<ROOT_DIR>`, just remember it)
```
cd <ROOT_DIR>
mkdir -p <ORGANISM_CODENAME>/spades
mkdir <ORGANISM_CODENAME>/trimmomatic
```
The `-p` there tells `mkdir` to make a directory and if the parent directory does not exist then make that too. Whatever you choose to do, just remember the directories to input in the output portion of commands. You can always find the absolute path to your current directory by running `pwd`.

<br />

### 1. Trimming Illumina paired-end reads
Find the adapter relevant to your sequence data - these are located in the trimmomatic software folder; on OSC this is the same directory as the argument following `ILLUMINACLIP` in the command below. Once ready, edit the following job submission command with your information - remember to pipe the output somewhere useful and informative, like the `trimmomatic` folder we created above:
```
echo -e 'cd </OUTPUT/FOLDER> && java -jar /fs/project/PAS1046/software/Trimmomatic-0.36/trimmomatic-0.36.jar \
PE -phred33 </PATH/TO/READ_1> </PATH/TO/READ_2> <OUTPUT_NAME_fpaired.fq.gz> <OUTPUT_NAME_funpaired.fq.gz> \
<OUTPUT_NAME_rpaired.fq.gz> <OUTPUT_NAME_runpaired.fq.gz> ILLUMINACLIP:/fs/project/PAS1046/software/Trimmomatic-0.3.6/adapters/TruSeq3-PE.fa:2:30:10:11 \
HEADCROP:10 CROP:145 SLIDINGWINDOW:50:25 MINLEN:100' | sbatch --time=01:00:00 --nodes=1 --ntasks-per-node=12 -A PAS#### --job-name=trimmomatic
```

<br />

### 2. Assembling
#### Accessing SPAdes assembler
When using SPAdes, you have to activate the SPAdes singularity container, which gives you access to SPAdes software. You do not need the container active for the rest of this tutorial, but we will use a special job submission command to invoke the container later on. For now, try running the following to introduce yourself to containers: 
```
singularity run /fs/project/PAS1046/software/containers/spades/spades_3.14.1.sif
spades.py --help
```

You should see a help menu with possible arguments for spades (if you get a `command not found error` contact me). Only use the container to run the software within it, otherwise you will experience weird errors. Also, the container should NOT be active when submitting jobs that use the container - the procedure for job submission is explained below. To exit the container, press `CTRL` + `D`, or type `exit`. Now try running `spades --help`. You'll probably get an error, do you see why? If not, no worries, you will get more familiar as you go.

Now we will prepare a job submission command in a "shell" file. Keeping commands in shell files is a good way to keep things organized and is ncessary for submitting jobs with containers. Create a plain text (UTF-8) file with either of the following commands if they fit your use case (check `spades.py --help` if these do not fit your criteria). Once finished, save it as an `.sh` file and transfer to OSC:

#### EXAMPLE1: Illumina 150 bp paired-end

```
spades.py --pe-1 1 </PATH/TO/TRIMMED/F_paired.fq.gz> --pe-2 1 </PATH/TO/TRIMMED/R_paired.fq.gz -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir </PATH/TO/SCRATCH> -t 6 -m 144 -o </PATH/TO/OUTPUT>
```

#### EXAMPLE2: Illumina 150 bp paired-end + Nanopore long read

```
spades.py --pe-1 1 </PATH/TO/TRIMMED/F_paired.fq.gz> --pe-2 1 </PATH/TO/TRIMMED/R_paired.fq.gz> --nanopore </PATH/TO/NANOPORE_READS> -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir </PATH/TO/SCRATCH> -t 6 -m 144 -o </PATH/TO/OUTPUT>
```

<br />

#### SUBMITTING A JOB SCRIPT
Now we submit the job by telling the supercomputer to invoke our singularity container and call upon the shell script you made above. Edit the command below and submit a job - do not have the container active when submitting the job.
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/spades/spades_3.14.1.sif bash <YOUR/CMD>' | sbatch --time=100:00:00 --nodes=1 --ntasks-per-node=6 -A PAS<####> --job-name=assembly
```

<br /><br />

## OUTPUT
SPAdes will output many files, but ultimately the `scaffolds.fa` is what you want to move on to [gene prediction](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#3.-predict-genes). Before moving ahead, let's keep it organized and make a directory for important output files to consolidate them and get them out of their different analyses folders:

```
mkdir <ORGANISM_CODENAME>/results
mv <SPADES/OUTPUT>/scaffolds.fa <ORGANISM_CODENAME>/results/<ORGANISM_CODENAME>.scaffolds.fa
```
