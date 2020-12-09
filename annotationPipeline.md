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
The software used here is contained within a *container* that needs to be activated to have access to the software. HOWEVER, it should only be active when running commands interactively in the login node. If you have the container active when you are trying to run other commands, you will potentially get weird errors, or `command not found` for commands you often run. To deactivate the container, press `CTRL` + `D` or run `exit`. It is also important to note, there is a special procedure for submitting jobs using containers that is explained in each tutorial that needs it. 

<br />

#### Are your `.sh` files formatted in plain text UTF-8 / have hidden characters?
As stated above, invoking a container during job submission requires a special procedure that makes use of a shell, or `.sh`, file. These files MUST be formatted in UTF-8 format, which requires the use of a plain text editor. Additionally, copying and pasting commands, particularly if you are using Windows, can inadvertently introduce hidden characters that you will not see in a text editor. These characters lead to obscure errors. I have separated the code you run in these tutorials in command boxes, that should remove hidden characters, however odd things can happen. 
So what do I recommend? If you are adventurous enough for this pipeline, you may benefit from using a command-line text editor like `nano` (easiest), `vim`, or `emacs`, which will allow you to write new files directly on the supercomputer and make sure they are plain-text. For hidden characters... Windows, man...

<br />

#### Are your paths inputted correctly?
Are your commands pointing to the correct files?

<br />

#### Are your files in accessible folders?
Running containers prevents access to other users' folders. You are only given access to your home folder, scratch folder, and shared project folder. Therefore, even during job submission commands, you CANNOT reference other users' files; they must be copied and referenced in an accessible folder or you will get a `file not found` error.

<br />

If none of these fit your issues, please contact Zach. Most, if not all, commands have been verified to be correct on here; however, there could be a cryptic issue I can help you work out.

## CREDIT
Flowchart originally authored by Kelsey Scott and Emile Gluck-Thaler; modified by Zachary Konkel from Maker to Funannotate and Orthofiller

