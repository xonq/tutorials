# Singularity containers

## NOTE
Singularity is a software that manages and executes software *containers*, like docker. Containers pack a program and its dependencies in an isolated environment that is easily disseminated. This not only makes bioinformatic software installation *much* easier, but it also allows multiple users to access the same installation and promotes reproducible computational science by freezing an image of published software and allowing reviewers to access that software with relative ease. If you are using OSC or another super computer, Singularity may be enabled out of the box. Just run `singularity --help` to check.

<br />

## INSTALLING

##### - [General Linux](https://singularity.lbl.gov/install-linux)
##### - Arch Linux:
```
git clone https://aur.archlinux.org/singularity-container.git
cd singularity-container
makepkg -si
```
##### - [MacOS](https://singularity.lbl.gov/install-mac)
##### - [Windows](https://singularity.lbl.gov/install-windows)

<br /><br />

## USE
#### Pulling premade software containers
Many bioinformatics software have containers built from "conda recipes" hosted at [bioconda](https://bioconda.github.io/) or elsewhere. Simply find the software recipe page on bioconda.
##### - copy the docker/singularity link and pull:
```
singularity pull docker://ONLINE/CONTAINER:<version>
```
NOTE - `docker://` is necessary when the links are docker containers

<br />

#### Running containers
##### - once pulled, activate the container
```
singularity run /YOUR/CONTAINER.sif
```
NOTE - only use the container for its specific commands; press CTRL + D or run `exit` to exit

<br />

#### Submitting jobs to Torque (OSC)
##### - create a text file with the command to run in the container, save it as an `.sh` file, and transfer to OSC:

##### - edit and submit a job to Torque referencing the command file
```
echo -e 'singularity exec /YOUR/CONTAINER.sif bash YOUR/FILE.sh' | qsub -l walltime=##:##:## -l nodes=#:ppn=# -A PAS####
```
NOTE - you do not need to submit with the container active; `exec` tells singularity to run the command and exit