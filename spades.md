# SPAdes *de novo* genome assembly

## NOTE
If you are using OSC and have access to PAS1046, you should be able to assemble out of the box.

<br />

## OSC USE
#### Accessing SPAdes assembler
##### - activate singularity container
```
singularity run /fs/project/PAS1046/software/containers/spades/spades.sif
```

<br />

### Assembling
Edit the presented commands with the appropriate assembler, kmers, etc. Activate the container and run `spades.py --help` for more information.

### EXAMPLE: Illumina 150 bp paired-end
##### - create a text file with the following command, save it as an `.sh` file, and transfer to OSC:

```
spades.py --pe1-1 /PATH/TO/TRIMMED/F_READS -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir /PATH/TO/SCRATCH -t 6 -m 144 -o /PATH/TO/OUTPUT
```

### EXAMPLE: Illumina 150 bp paired-end + Nanopore long read
##### - create a text file with the following command, save it as an `.sh` file and transfer to OSC:

```
spades.py --pe1-1 /PATH/TO/TRIMMED/F_READS.fq.gz --pe1-2 /PATH/TO/TRIMMED/R_READS.fq.gz --nanopore /PATH/TO/NANOPORE_READS -k 21,33,55,77,99,121 --cov-cutoff auto --tmp-dir /PATH/TO/SCRATCH -t 6 -m 144 -o /PATH/TO/OUTPUT
```

<br />

##### - edit and submit a job to Torque
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/spades/spades.sif bash YOUR/FILE.sh' | qsub -l walltime=100:00:00 -l nodes=1:ppn=8 -A PAS#### -N spades
```
NOTE - you do not need to submit with the container active
