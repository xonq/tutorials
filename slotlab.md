# Slot Lab Ohio Supercomputer Center Best Practices
## ACCESSING SOFTWARE
Primary software containers are stored in `/fs/project/PAS1046/software/containers`. If you wish to use one of these software, add the binaries to your `$PATH` by adding the following line to your `~/.bash_profile`:

```
#Singularity containers
export PATH=$PATH:/fs/project/PAS1046/software/containers/.bin
```

Then, identify the `.sif` file for the software you want and call it open to access the software:

```
<CONTAINER>.sif
```

To close, press `CTRL` + `D` or command `exit`.

<br /><br />

## INSTALLING SOFTWARE
 
There are multiple methods for installing software, but a singularity container is the easiest when available. Quay.io is a website that has prepackaged bioinformatics containers, and there is a solid chance your software of interest is located there. Your first option is to use a search engine, such as Qwant/google, to search for your software of interest and `quay.io`. If the software exists open the quay.io URL, click the tags section, click "fetch tag" on the far right of the latest version, and pull down "Docker Pull (by tag)". Take note of the `quay.io/biocontainers....` URL.

Now, on OSC, make a directory in `/fs/project/PAS1046/software/containers/` with your software name (preferably lower case). Navigate into the directory you created and run the following command with the URL you just retrieved then activate it to test:

```
#pull the container
singularity pull <URL>

#active it
./<CONTAINER_NAME>.sif
```

<br /><br />

## ADDING CONTAINERS TO PATH

You can add all containers to your path manually by adding the following line of code to your `~/.bash_profile`:

```
export PATH=$PATH:/fs/project/PAS1046/software/containers/.binlinks
```

New containers will not show up until you run `bash /fs/project/PAS1046/software/containers/.addlinks.sh` or contact Zach to do it.

Use [Singularity](https://gitlab.com/xonq/tutorials/-/blog/master/containers.md) (or Docker) to pull the prebuilt containe
```
singularity pull docker.io://xonq/funannotate_mask:1.8.1
```

<br />

## OSC USE
#### Accessing repeat masking software
Repeat masking software is *contained* within a software *container*. To access RepeatMasker software you will have to activate the container:
```
/fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
```
Press CTRL + D or run `exit` to exit it. Only run the container for repeat masking commands otherwise you will experience unexpected problems.

<br />

### Repeat modeling
#### 1) build a database
RepeatModeler outputs more files than you could ever deal with, so to circumnavigate project and user folder file limits, we will make an output directory in your scratch folder (`/fs/scratch/<PAS####>/<USER>`). Once done, we prepare an NCBI database for `RepeatModeler` to reference when actually running RepeatModeler. Make sure you have the container active as explained above.

```
mkdir <SCRATCH/OUTPUT/DIRECTORY>
cd <SCRATCH/OUTPUT/DIRECTORY>
BuildDatabase -name <NAME> -engine ncbi <YOUR/ASSEMBLY>
```

#### 2) run repeatmodeler
Now, deactivate the container, create a plain text (UTF-8) shell file with the following command, save it with a `.sh` extension, and transfer to OSC. This file is for job submission. NOTE - reference the `<NAME>` used after `-name` in the previous command, *no file extensions*. So, if in the above command you used 'psicub1' for `<NAME>` you will input `<DATABASE/PATH>/psicub1`.


```
cd </SCRATCH/OUTPUT/DIRECTORY>
RepeatModeler -pa 8 -database <SCRATCH/OUTPUT/DIRECTORY/NAME> -engine ncbi 2>&1 | tee repeatmodeler.log
```

Edit the command below and submit the job. Remember, you do not want the container active when submitting the job - the job itself calls upon the container. Confusing, yes, but what you are doing is 1. telling the computer to submit a command to the supercomputer job scheduler, 2. telling the job itself to invoke singularity, and 3. telling singularity to run your shell file prepared above.

```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMD>.sh' | sbatch --time=100:00:00 --nodes=1 --ntasks-per-node=8 -A PAS<####> --job-name=repeatmodeler
```

<br />

## OUTPUT
Keep it clean - if you know `cd`, `mv`, `ls` and `rm` then you know everything you need to move files around and create a clean directory structure with informative directory names and file paths. You will not need most of the files - instead, the `<NAME>-families.fa` is the *de novo* repeat library you reference when [masking](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#2-soft-mask-assembly) your assembly for gene prediction. Let's move that file into your organism(s)' folder(s) that we created when starting assembly.

```
mkdir <ORGANISM>/repeatmodeler
mv <REPEATMODELER>/<NAME>-families.fa <ORGANISM>/repeatmodeler
```

## CITATION INFO
This installation of RepeatModeler references the Dfam 3.0 repeat library. In addition to citing RepeatMasker software, cite the Dfam library as well.
