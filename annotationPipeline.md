# Slot Lab Annotation Pipeline
## GETTING STARTED
If you are using the Ohio Supercomputer (OSC) and have access to PAS1046, you should be able to skip all installation and proceed to the OSC Use sections of the following tutorials. All you need to do is run this command to set up your profile:
```
echo 'export SINGULARITY_BINDPATH="/opt:/mnt"' >> ~/.bash_profile
```

<br />

## SUMMARY
#### 1) Acquire Reads
#### 2) [Prepare & assemble reads with SPAdes](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md)
#### 3) [Generate a repeat library with RepeatModeler](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md)
#### 4) [Predict genes with Funannotate](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md)
#### 5) Fill annotation with OrthoFiller
#### 6) [Curate annotation](https://gitlab.com/xonq/turotials/-/blob/master/annotationCuration.md)

![Slot Lab Annotation Pipeline flowchart](https://gitlab.com/xonq/tutorials/-/raw/master/image/annotationPipeline.png "Flowchart")

Flowchart originally authored by Kelsey Scott and Emile Gluck-Thaler; modified by Zachary Konkel from Maker to Funannotate and Orthofiller
