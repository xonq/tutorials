# RepeatModeler *de novo* repeat library generation

## NOTE
- This installation uses the Dfam database. Further modification is necessary to use RepBase.

If you are using OSC and have access to PAS1046, you should be able to repeat mask out of the box.

<br />

## OSC USE
#### Accessing repeat masking software
##### - activate singularity container
```
singularity run /fs/project/PAS1046/software/containers/repeat/repeat_tools.sif
```

<br />

### Repeat modeling
Compile an [assembly](https://gitlab.com/xonq/tutorials/-/blob/master/spades.md)

##### - build an NCBI database for `RepeatModeler` to reference
```
BuildDatabase -name OME -engine ncbi YOUR/ASSEMBLY.fasta
```

##### - create a text file with the following command, save it as an `.sh` file, and transfer to OSC:

```
cd YOUR/SCRATCH/FOLDER
RepeatModeler -pa 8 -database YOUR/NCBI/DATABASE/OME -engine ncbi 2>&1 | tee repeatmodeler.log
```
NOTE - reference the `OME` after `-database`, *no file extensions*

##### - edit and submit a job to Torque
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/repeat/repeat_tools.sif bash YOUR/FILE.sh' | qsub -l walltime=100:00:00 -l nodes=1:ppn=8 -A PAS#### -N repeatmodeler
```
NOTE - you do not need to submit with the container active
