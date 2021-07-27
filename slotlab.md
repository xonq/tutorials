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
