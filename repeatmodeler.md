# RepeatModeler *de novo* repeat library generation

## Getting Started
You will need [a *de novo* assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md). If you are using the Ohio Super Computer (OSC) and have access to PAS1046 you can skip installation to [OSC use](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md#osc-use).


<br /><br />

## INSTALLING
 
Use [Singularity](https://gitlab.com/xonq/tutorials/-/blog/master/containers.md) (or Docker) to pull the prebuilt container
```
singularity pull docker.io://xonq/funannotate_mask:1.8.1
```

<br />

## OSC USE
#### Accessing repeat masking software
To use RepeatMasker software you will have to activate the container. Press CTRL + D or run `exit` to exit. Only run the container for repeat masking commands otherwise you will experience unexpected problems.

```
singularity run /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
```

<br />

### Repeat modeling
build an NCBI database for `RepeatModeler` to reference. Make sure you activate the container as explained above.

```
BuildDatabase -name <NAME> -engine ncbi <YOUR/ASSEMBLY>
```

Make an output directory  in your scratch folder:

```
mkdir <YOUR/SCRATCH/DIR>
```

create a plain text (UTF-8) file with the following command, save it as an `.sh` file, and transfer to OSC. NOTE - reference the `NAME` after `-database`, *no file extensions*


```
cd <YOUR/SCRATCH/FOLDER/OUTPUT>
RepeatModeler -pa 8 -database <YOUR/NCBI/DATABASE/NAME> -engine ncbi 2>&1 | tee repeatmodeler.log
```

Edit the command below and submit the job. *Do not activate the container to submit a job using it*

```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMD>.sh' | qsub -l walltime=100:00:00 -l nodes=1:ppn=8 -A PAS<####> -N repeatmodeler
```

<br />

## OUTPUT
RepeatModeler outputs a ton of files, which is why output needs to be directed to a scratch folder. You will not need most of the files - instead, the `-consensi.fa` is the *de novo* repeat library you reference when [masking](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#2.-soft-mask-assembly) your assembly for gene prediction.