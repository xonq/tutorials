# RepeatModeler *de novo* repeat library generation

## Getting Started
You will need [a *de novo* assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md). If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip installation to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md#osc-use) once you have added the line to your profile from [here](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#getting-started).


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
Make an output directory in your scratch folder, then build an NCBI database for `RepeatModeler` to reference. Make sure you activate the container as explained above.

```
mkdir <SCRATCH/OUTPUT/DIRECTORY>
cd <SCRATCH/OUTPUT/DIRECTORY>
BuildDatabase -name <NAME> -engine ncbi <YOUR/ASSEMBLY>
```

Deactivate the container, create a plain text (UTF-8) file with the following command, save it as an `.sh` file, and transfer to OSC. NOTE - reference the `<NAME>` used previously after `-database`, *no file extensions*


```
cd </SCRATCH/OUTPUT/DIRECTORY>
RepeatModeler -pa 8 -database <SCRATCH/OUTPUT/DIRECTORY/NAME> -engine ncbi 2>&1 | tee repeatmodeler.log
```

Edit the command below and submit the job. *Do not activate the container to submit a job that invokes the container*

```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMD>.sh' | qsub -l walltime=100:00:00 -l nodes=1:ppn=8 -A PAS<####> -N repeatmodeler
```

<br />

## OUTPUT
RepeatModeler outputs a ton of files, which is why output needs to be directed to a scratch folder. You will not need most of the files - instead, the `<NAME>-families.fa` is the *de novo* repeat library you reference when [masking](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#2-soft-mask-assembly) your assembly for gene prediction. Simply move the main files to a different directory so they do not get deleted in the scratch folder and either compress or delete the working directory.
