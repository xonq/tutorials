# Slot Lab *de novo* Gene Annotation Pipeline
## GETTING STARTED
This pipeline and tutorials are tailored toward the annotation of fungal genomes without direct transcript evidence. Before each command in each tutorial, I have tried to explain what you will be doing within the command itself. If you are using the Ohio Supercomputer (OSC) and have access to PAS1046, you should be able to skip all installation and proceed to the OSC Use sections of the following tutorials. All you need to do is add this line to your `~/.bash_profile` file in your OSC home directory:
```
export SINGULARITY_BINDPATH="/opt:/mnt"
```

If you are having trouble with this pipeline, refer to the [troubleshooting](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#troubleshooting) section at the bottom of this page.

<br />

## SUMMARY
#### 1) [Prepare & assemble reads with SPAdes](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md)
#### 2) [Generate a repeat library with RepeatModeler](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md)
#### 3) [Predict genes with Funannotate](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md)
#### 4) [Fill annotation with OrthoFiller](https://gitlab.com/xonq/tutorials/-/blob/master/orthofiller.md)

![Slot Lab Annotation Pipeline flowchart](https://gitlab.com/xonq/tutorials/-/raw/master/image/annotationPipeline.png "Flowchart")

<br />

## TROUBLESHOOTING
Many common problems can be addressed by asking the following:


<br />

#### Is everything spelled correct and do the PATH(s) exist?
- most of my errors are due to spelling mistakes or using non-existent directories

- sometimes flags, like `-o / --output`, need directories and sometimes they need files; these are made clear in the tutorials


##### Common errors
These are typically made clear


##### Solutions
- meticulously scan your command and check if you PATHs are real

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
