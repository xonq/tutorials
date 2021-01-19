# Slot Lab *de novo* Gene Annotation Pipeline | 2021/01/11
This pipeline and tutorials are tailored toward the annotation of fungal genomes without direct transcript evidence. Please read thoroughly, sometimes rereading to understand - what is not necessary to read will be made clear. Make sure to read the [GETTING STARTED](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#getting-started) section. If you are having trouble with this pipeline, refer to the [TROUBLESHOOTING](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#troubleshooting) section.


![Slot Lab Annotation Pipeline flowchart](https://gitlab.com/xonq/tutorials/-/raw/master/image/annotationPipeline.png "Flowchart")

<br />

## SUMMARY
#### 1) [Prepare & assemble reads with SPAdes](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md)
#### 2) [Generate a repeat library with RepeatModeler](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md)
#### 3) [Predict genes with Funannotate](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md)
#### 4) OPTIONAL: [Fill annotation with OrthoFiller](https://gitlab.com/xonq/tutorials/-/blob/master/orthofiller.md)

<br />

## GETTING STARTED
#### How to read tutorials
Commands in this tutorial will be placed in obvious command boxes `like this` or
```
like this
```

The information required to put into shell - `.sh` files - will be in their own box and made clear in the writing before each command where I've attempted to explain what the command(s) do. You will also find parts within commands that you need to edit inside these `<>` brackets; for example, if you see `<OUTPUT/DIRECTORY>` then you may change that to `~/athelia/` if that is where you are piping your output. 

#### Useful information
To get started, you will want to be familiar with interacting in the command line to some extent: knowing what `mv`, `cd`, `rm`, `mkdir`, and `ls` do is a good start. Dr. Michael Sovic has some [good videos](https://www.youtube.com/playlist?list=PLxhIMi78eQehzRgd1C6wkJaaf0_nEnmvH) on these. Other important info:

1) If a line of code ends with EXACTLY (even the space before) ` \` then it is telling the command line that you want to write on the next line... this keeps things readable and clean - which is a theme this tutorial puts forth. If you do not add the `\`, then each line will be executed as an individual line of code. A clean lab bench is best practice.

2) The `\` means completely different things than `/`, as the latter is typically 
involved in delineating paths. Paths point to files, e.g. `/fs/scratch/` (or even this URL!); however, there are multiple ways to delineate paths. The full/
absolute path always points to the exact location of a file and is therefore always safe to use, however the *relative* path 
is only relative to your current position in the file system. 

Say my current folder is: `/user/zach/` (which I found via `pwd`)

The absolute path for a directory could be `/users/zach/this/is/my/path/`. It is "absolute" because the path begins at the start of the file system. Can you guess what the starting folder of the file system is? ... it is simply `/`. Absolute paths can also start with `~` (your home directory) or `$` (beyond the scope of this tutorial). 

So if my current directory is `/user/zach/` and the absolute path is `/users/zach/this/is/my/path/`, then a path relative to my position in the file system, or *the relative path* is: `this/is/my/path`. Notice what it starts with... the directory `this/`. If instead you wrote `/this/` it would be pointing you at an absolute path for a directory "this" at the start of the file system. You may also see `./this/is/my/path` (the `.` indicates current directory).

Now, if I changed to an existing folder with a relative path like this: `cd this/is/` it would work. I then print my current directory: `pwd`, and it shows I'm now in `/user/zach/this/is/`... But what if I now tried the relative path I used before: `this/is/my/path/`? Will it exist? ... No. Because the absolute path for that is actually `/user/zach/this/is/this/is/my/path/` *relative* to my position. TL;DR, if you point to the same relative path in a different spot, it won't exist!  

<br />

#### Actually getting started
With all that out of the way, if you are using the Ohio Supercomputer (OSC) and have 
access to PAS1046, you should be able to skip all installation and proceed to the OSC Use 
sections of the following tutorials. All you need to do to get started is add this line 
to your `~/.bash_profile` file in your OSC home directory to circumnavigate OSC container
mount point restrictions:
```
export SINGULARITY_BINDPATH="/opt:/mnt"
```

<br /><br />

## TROUBLESHOOTING
Many common problems can be addressed by asking the following:


<br />

#### Is everything spelled correct and do the PATH(s) exist?
- most of my errors are due to spelling mistakes or using non-existent directories

- are your paths or relative? if relative, are you in the correct directory?

- sometimes flags, like `-o / --output`, need directories and sometimes they need files; these are made clear in the tutorials


##### Common errors
These are typically made clear


##### Solutions
- meticulously scan your command and check if you PATHs are real; convert relatives to
absolute

- make sure the correct virtual environment (miniconda) is loaded or the environment is appropriately sourced

- run `man $CMD` or `$CMD --help` to check what the command wants (sometimes `-h / -help`)


<br />

#### Is the container active/inactive appropriately?
- most of the pipeline software is contained within [*containers*](https://gitlab.com/xonq/tutorials/-/blob/master/containers.md) 

- containers should only be activated to run the software within them

- when active, a container will change your shell prompt (e.g. for Owens@OSC `-bash-4.2$` to `<Singularity>`)

- submitting jobs in a container is different than interacting with it on the login node

- do not activate the container when submitting a job that calls on it


##### Common errors
`command not found` ONLY if you know for sure the command should be available (i.e. loaded into your PATH)


##### Solutions
- to deactivate the container press `CTRL` + `D` or run `exit`


<br />

#### Are your `.sh` files formatted in plain text UTF-8 and free of hidden characters?
- shell files must be written in UTF-8 format, which requires the use of a plain text editor

- copying and pasting commands can inadvertently introduce hidden characters 


##### Common errors
An error stating the flag/command does not exist, though you have checked the `--help` menu and confirmed the flag is spelled correct. 


##### Solutions
- try a command-line text editor like `nano` (easiest), `vim`, or `emacs`

- rewrite the command / suspect portion manually in a plain text editor instead of copying and pasting


<br />

#### Are your files in accessible folders?
- containers can prevent access to some directories for safety

- OSC only grant access to your home folder, scratch folder, and shared project folder


##### Common errors
`file not found` or `directory / folder not found` and the path does not fit the above criteria


<br />

#### None of these fit my error.
Researching and tracing back errors is a large part of computational work. I venture to say most of my conceptual computation knowledge comes from independent research of countless errors... it really is a great way to learn, but if you cannot find a solution, please contact me or raise an issue.

## CREDIT
Flowchart originally authored by Kelsey Scott and Emile Gluck-Thaler; modified by Zachary Konkel from Maker to Funannotate and Orthofiller

Tutorials produced by Zachary Konkel
