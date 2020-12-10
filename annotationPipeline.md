# Slot Lab Annotation Pipeline
## GETTING STARTED
If you are using the Ohio Supercomputer (OSC) and have access to PAS1046, you should be able to skip all installation and proceed to the OSC Use sections of the following tutorials. All you need to do is add this line to your `~/.bash_profile` file in your OSC home directory:
```
export SINGULARITY_BINDPATH="/opt:/mnt"
```

<br />

## SUMMARY
#### 1) Acquire Reads
#### 2) [Prepare & assemble reads with SPAdes](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md)
#### 3) [Generate a repeat library with RepeatModeler](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md)
#### 4) [Predict genes with Funannotate](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md)
#### 5) [Fill annotation with OrthoFiller](https://gitlab.com/xonq/tutorials/-/blob/master/orthofiller.md)
#### 6) [Curate annotation](https://gitlab.com/xonq/tutorials/-/blob/master/annotationCuration.md)

![Slot Lab Annotation Pipeline flowchart](https://gitlab.com/xonq/tutorials/-/raw/master/image/annotationPipeline.png "Flowchart")

<br />

## TROUBLESHOOTING
Many common problems can be addressed by asking the following:

<br />

#### Is the container active/inactive appropriately?
This software is contained within a *container* that needs to be activated to have access. HOWEVER, containers should only be active when running commands that are specific to the container software. If you have the container active when you are trying to run other commands you will potentially get weird errors, or `command not found`. To deactivate the container, press `CTRL` + `D` or run `exit`. It is also important to note, there is a special procedure for submitting jobs using containers that is explained in each tutorial that needs it. CONTAINERS SHOULD NOT BE ACTIVE DURING JOB SUBMISSION. 

<br />

#### Are your `.sh` files formatted in plain text UTF-8 / have hidden characters?
Invoking a container during job submission requires a shell, or `.sh`, file. These files MUST be formatted in UTF-8 format, which requires the use of a plain text editor. Additionally, copying and pasting commands, particularly if you are using Windows, can inadvertently introduce hidden characters - these will often manifest in an error that says a flag does not exist, even though you have checked the `--help` menu and confirmed the flag is spelled correct. I have separated code in these tutorials into command boxes, which should remove hidden characters, however odd things can happen. I recommend getting familiar with a command-line text editor like `nano` (easiest), `vim`, or `emacs`, which will allow you to write new files directly from the command line and make ensure they are plain-text. Most of the time I have observed hidden character errors from Windows.

<br />

#### Are your paths inputted correctly?
Are your commands pointing to the correct files?

<br />

#### Are your files in accessible folders?
Running containers prevents access to other users' folders. You are only given access to your home folder, scratch folder, and shared project folder. Therefore, even during job submission commands, you CANNOT reference other users' files; they must be copied and referenced in an accessible folder or you will get a `file not found` error.

<br />

#### Are you specifying the right output type?
Many commands use an output flag, typically `-o` or `--output`. However, some programs want a file name and some want a directory. Frustratingly, some programs want a directory that exists, some will make it if it does not exist, and some want to make it. I have tried to make these nuances clear in the tutorials by denoting output files as something similar to `<OUTPUT/FILE>` and directories as `<OUTPUT/FOLDER>` so please abide by the tutorials. Most commands also come equiped with a `-h`, `-help`, and/or `--help` flag, so you can also run that and check what it wants for the output flag.

<br />

#### None of these fit my error.
Computing is a lot of troubleshooting and you can only learn how to overcome those errors on your own if you familiarize yourself with interpreting the errors, tracing back the error to the source, and researching solutions. Most of my conceptual knowledge about computers comes from countless failures and errors. Please give an effort to solve your issues on your own. If you cannot find a solution, please contact me. Most, if not all, commands have been verified to be correct on here; however, there could be a cryptic issue I can help you work out.

## CREDIT
Flowchart originally authored by Kelsey Scott and Emile Gluck-Thaler; modified by Zachary Konkel from Maker to Funannotate and Orthofiller

Tutorials produced by Zachary Konkel
