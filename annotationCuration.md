# Post annotation curation/analysis

## NOTE
If you are using the Ohio Super Computer and have access to PAS1046, you should be able to use the necessary custom python libraries by following the directions in my scripts folder `README.md`. The path to my scripts will be denoted as `$SCRIPTS`.

<br />

## Prerequisites
- Miniconda3 installation
- Xonq python libraries

<br />

## Curation

#### Gene/Protein headers
`<$SCRIPTS>/curateFunannotate.py -n <NAMECODE> -g <GFF3> -p <PROTEOME> -t <TRANSCRIPTS>`

<br />

## Analysis

#### Annotation statistics
`<$SCRIPTS>/annotationStats.py <GFF3>`
