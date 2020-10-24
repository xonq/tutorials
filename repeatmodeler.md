# RepeatModeler *de novo* repeat library generation

## NOTE
If you are using OSC and have access to PAS1046, you should be able to repeat mask out of the box.

<br />

## Prerequisites
- [Assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md) - `scaffolds.fasta`

<br />

## OSC USE
#### Accessing repeat masking software
##### - activate singularity container
```
singularity run /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
```

<br />

### Repeat modeling
##### - build an NCBI database for `RepeatModeler` to reference
```
BuildDatabase -name <NAME> -engine ncbi <YOUR/ASSEMBLY>
```

##### - create a plain text (UTF-8) file with the following command, save it as an `.sh` file, and transfer to OSC:

```
cd YOUR/SCRATCH/FOLDER
RepeatModeler -pa 8 -database <YOUR/NCBI/DATABASE/NAME> -engine ncbi 2>&1 | tee repeatmodeler.log
```
NOTE - reference the `NAME` after `-database`, *no file extensions*

##### - edit and submit a job to Torque
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash YOUR/CMD.sh' | qsub -l walltime=100:00:00 -l nodes=1:ppn=8 -A PAS<####> -N repeatmodeler
```
NOTE - you do not need to submit with the container active
