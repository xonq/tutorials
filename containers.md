# Singularity containers

## NOTE
- [Singularity documentation](https://sylabs.io/guides/3.6/user-guide/)

Singularity is a software that manages and executes software *containers*, like docker. Containers pack a program and its dependencies in an isolated environment that is easily disseminated. This not only makes bioinformatic software installation *much* easier, but it also allows multiple users to access the same installation. Furthermore, publishing container links promotes reproducible computational science by providing an image of published software and allowing reviewers to access that software with relative ease. 

<br />

## INSTALLING
If you are using OSC or another super computer, Singularity may be enabled out of the box. Just run `singularity --help` to check.

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

## Premade containers
#### Pulling containers
Many bioinformatics software have containers built from "conda recipes" hosted at [bioconda](https://bioconda.github.io/) or elsewhere. Simply find the software recipe page on bioconda.
##### - copy the docker/singularity link and pull:
```
singularity pull docker://<USER/CONTAINER>:<VERSION>
```
NOTE - `docker://` is necessary when the links are docker containers

<br />

#### Running containers
##### - once pulled, activate the container
```
singularity run </YOUR/CONTAINER.sif>
```
NOTE - only use the container for its specific commands; press CTRL + D or run `exit` to exit

<br />

#### Submitting jobs to Torque (OSC)
Using containers have some quirks - particularly, you do not have access to other users' folders and when submitting a job to a compute cluster the command must be executed in a bash script.

##### - create a text file with the command to run in the container, save it as an `.sh` file, and transfer to OSC:

##### - edit and submit a job to Torque referencing the command file
```
echo -e 'singularity exec </YOUR/CONTAINER.sif> bash <YOUR/CMD.sh>' | qsub -l walltime=<##:##:##> -l <nodes=#:ppn=#> -A PAS<####>
```
NOTE - you do not need to submit with the container active; `exec` tells singularity to run the command and exit

<br /><br />

## Building containers
Building containers requires root access. The general steps are described below, but there is too much nuance to go into detail here. If you cannot find an existing container in a trusted repository, consider creating an environment with `miniconda` and installing software into the environment.


### Choose base OS Dockerfile / Singularity file(s)
Find the most lightweight OS possible that can satisfy the dependencies of the software you want to install. E.g. Miniconda's preassembled container: `continuum.io/miniconda3`

### Edit Dockerfile / Singularity file(s) to install dependencies
Review build file formatting, install dependencies, and clean out excess material from the container. Dockerfiles are more common and once they are built and pushed, they can be pulled by Singularity as previously described.

### Build
Build the container: `docker build -i <DOCKER_USR/PROJECT>:<VERSION> .` Test it out by running `docker run -i -t <DOCKER_USR/PROJECT> /bin/bash`.

### Push
Push the container to docker.io: `docker push <DOCKER_USR/PROJECT>:<VERSION>`

### Pull
Pull the container at OSC: `singularity pull docker://<DOCKER_USR/PROJECT>:<VERSION>`
