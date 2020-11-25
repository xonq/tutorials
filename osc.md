# Supercomputer Wisdom
Getting started with job submission

<br />

## Slurm
Super computers use scheduling systems to allocate resources to many software in a systematic way. Torque used to 
be the default job scheduler for Ohio Super Computer, which includes the `qsub qstat qdel #! PBS` commands, but 
as of 2021 most clusters schedule using Slurm.

### Job submission
You can submit jobs through the command line or by creating a plain text shell `.sh` file. I recommend a shell 
file as it serves as a log, keeps complex commands readable, and is easily edited via a command line text editor.

#### shell file 

OPTIONAL lines are not required
```
#!/bin/bash
#SBATCH --nodes=<REQUIRED typically 1>
#SBATCH --ntasks-per-node=<REQUIRED_THREADS/CORES>
#SBATCH --time=<REQUIRED hh:mm:ss>
#SBATCH --account=<REQUIRED_ACCOUNT>
#
#SBATCH --mem=<OPTIONAL_RAM>
#SBATCH --job-name=<OPTIONAL_JOB_NAME>
#SBATCH --output=<OPTIONAL_LOG_OUTPUT_PATH>
#SBATCH --error=<OPTIONAL_LOG_ERROR_PATH>

<YOUR_CMD>
```

Once the script is created and transferred, run via:
```
sbatch <YOUR_SCRIPT>
```

<br />

#### command line submission
```
echo -e "<YOUR_CMD>" | sbatch --nodes=<NODES> --ntasks-per-node=<THREADS/CORES> -A <ACCOUNT>
```

<br /><br />

### Job monitoring

#### monitor by user
```
squeue -u <USERNAME>
```

#### monitor by job
```
sstat -j <JOB_ID> 
```

#### expected start time
```
squeue --start --jobs=<JOB_ID>
```

#### delete a job
```
scancel <JOB_ID>
```
