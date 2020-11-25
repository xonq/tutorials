# Supercomputer Wisdom
Getting started with job submission

<br />

## Slurm
Slurm and Torque/PBS are job scheduling systems. Essentially super computers use scheduling systems to allocate
resources to many software in a systematic way. Torque used to be the default job scheduler for Ohio Super
Computer, which includes the `qsub qstat qdel #! PBS` commands, but as of 2021 most clusters schedule using Slurm.

### Job submission
You can submit jobs through the command line or by creating a plain text shell `.sh` file. I recommend using this
as it serves as a log, keeps complex commands readable, and is easily edited via a command line text editor.

shell file, lines that are optional can be removed:
```
#!/bin/bash
#SBATCH --ntasks=<REQUIRED_THREADS/CORES>
#SBATCH --time=<REQUIRED_WALLTIME>
#SBATCH --mem-per-cpu=<OPTIONAL_MB_RAM_PER_CPU>
#SBATCH --job-name=<OPTIONAL_JOB_NAME>
#SBATCH --output=<OPTIONAL_LOG_OUTPUT_FILENAME>

<YOUR_CMD>
```

<br />

command line submission:
```

