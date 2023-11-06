# Funannotate *de novo* annotation software setup and use

## GETTING STARTED
[Bakta](https://bakta.readthedocs.io/en/latest/) is a genome
annotation "wrapper" for prokaryotes and plasmids that outperforms other
software, such as Prokka, in annotation comprehensitivity and applying
functional annotations.

You will need a prokaryote genome assembly to proceed. If you are using the
Ohio Super Computer (OSC) and have access to PAS1568, skip installation to [OSC
USE](https://github.com/xonq/tutorials/blob/master/bakta.md#osc-use).

---

<br /><br /><br />

## INSTALLING
#### Pull the Bakta Container
 
Use
[Singularity](https://github.com/xonq/tutorials/blob/master/containers.md)
(or Docker) to pull the Bakta container.

```
singularity pull docker://quay.io/biocontainers/bakta:latest
```

Install databases.
```
<CONTAINER.sif>
bakta_db download --output <DB_DIRECTORY>
```

<br />

Continue to 
[OSC USE](https://github.com/xonq/tutorials/blob/master/bakta.md#osc-use).

<br /><br /><br />

## OSC USE
#### Keep it clean (create a directory)

It is best practice to create a directory for a particular analysis. I
personally prefer to create a directory for every genome annotation, e.g.:

```
mkdir <OUTPUT_DIR>/<GENOME_CODENAME>/bakta
```

<br />

#### Preparing Bakta commands

For high performance computing environments, such as OSC, it is necessary to
create TWO scripts: 1 script will call on `Bakta` to run the annotation
command, while another script will invoke `Singularity`, the container
software, to call Bakta.


##### Bakta script

Create a `.sh` script file in UTF-8 using a plain-text editor. 
Copy and fill in the following commands' arguments (`<>`)
with your genome's metadata, and omit `--gram` if unknown. There are several
notes to consider:

- If your genome is a plasmid, then append `--plasmid <PLASMID_NAME>`
- If your genome is complete (to chromosome/plasmid level), then append
  `--complete`
- If your genome is a metagenoome, then append `--meta`

```
bakta --db /fs/ess/PAS1568/databases/bakta/ --genus <GENUS> \
--species <SPECIES> --gram <+/-/?> --strain <STRAIN> -t 8 \
-o <YOUR/OUTPUT/DIRECTORY> <YOUR/ASSEMBLY/PATH>
```

Once complete, save your script as a `.sh` file, such as `bakta.sh`, then
transfer it to a folder in OSC.


##### Singularity job script

The `Singularity` job script will call on the script you just wrote to launch
the job on the high performance computing environment. For the SLURM job
scheduler, which OSC uses, create another UTF-8 formatted file using a
plain-text editor; make sure the path `<YOUR/BAKTA_SCRIPT.sh>` references the
OSC path of the script you previously made above:

```
#!/bin/bash
#SBATCH --job-name=<YOUR_JOB_NAME>
#SBATCH --output=<YOUR_JOB_NAME>
#SBATCH --error=<YOUR_JOB_NAME>
#SBATCH --time=24:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH -A PAS1568

singularity exec /fs/ess/PAS1568/software/bakta/bakta.sif /bin/bash <YOUR/BAKTA_SCRIPT.sh>
```

Once complete, save your script as a `.sh` file, such as `bakta_execution.sh`,
then transfer it to a folder in OSC. 

<br />

#### Executing Bakta command

```
sbatch <bakta_execution.sh>
```

<br /><br />


## CITATION INFO

Bakta is the `wrapper` that packages many other software. In addition to
[citing Bakta](https://pubmed.ncbi.nlm.nih.gov/34739369/), you should cite the
included software, such as Prodigal, Aragorn, tRNAScan, Infernal, Rfam, Blast,
Diamond, Hmmer, Pfam, etc, depending on the software used in your analysis.
