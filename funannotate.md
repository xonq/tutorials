# Funannotate *de novo* annotation software setup and use

## NOTE 
- Funannotate documentation: https://funannotate.readthedocs.io/en/latest/install.html. 
- Fully capitalized paths, like `INSTALLATION/PATH` need to be manually edited by the user.

If you are using OSC and have access to PAS1046, you should be able to run Funannotate after acquiring a GeneMark use key.

<br />

## OSC USE
#### Accessing GeneMark
Accept the license for `GeneMark-ES/ET/EP` (http://topaz.gatech.edu/GeneMark/license_download.cgi), download the 64-bit key (NOT the program), and transfer to your OSC home directory. 
##### - uncompress the key, then place it where GeneMark looks
```
gunzip gm_key_64.gz
mv gm_key_64 ~/.gm_key
```

NOTE - These expire in 400 days and will cause GeneMark errors.

<br />

#### Accessing Funannotate
##### - activate singularity container then temporarily source the Funannotate directories to your path
```
singularity run /fs/project/PAS1046/software/containers/funannotate/funannotate_1.7.4.sif
source /fs/project/PAS1046/software/containers/funannotate/source.sh
```
NOTE - Use the environment to run Funannotate commands. To deactivate press CTRL + D or run `exit`. For job submission see 2. Gene Prediction.


##### - check the first time
```
funannotate check
```
NOTE - perl locale, `hisat2`, `ete3`, `singalp` and `emapper.py` errors are fine for annotation; `gmes_petap.pl` errors are not

<br />

### 1. Soft-mask assembly. 
Compile an assembly and RepeatModeler library fasta - `$OME-families.fa`

##### - Soft mask the assembly by lowercasing masked nucleotides from the repeat library:
```
funannotate mask -i YOUR/ASSEMBLY -o OUTPUT/MASKED_ASSEMBLY_NAME -l YOUR/REPEATMODELER/$OME-families.fa
```
NOTE - access the container to run the command; you will not need to submit an OSC job

<br />

### 2. Gene prediction. 
Download/compile necessary data and information:
- transcript/EST evidence from organism(s) in the same genus
- protein evidence from at least 10 closely related organisms (separate by spaces in command)
- find the most closely related BUSCO species database `/fs/project/PAS1046/databases/busco/config/`. Replace `BUSCO_SPECIES` below with the name of the BUSCO species parameter folder, *not the full path*.

##### - create a text file with the following funannotate command, save it as a `.sh` file, and transfer to OSC:
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

funannotate predict -i YOUR/MASKED_ASSEMBLY -s “$OME_$RUN#” --transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE \
--protein_evidence YOUR/PROTEIN_EVIDENCE /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
–cpus 8 --busco_seed_species BUSCO_SPECIES -o OUTPUT/FOLDER
```

##### - edit and submit a job to Torque to run that file in the funannotate container
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_1.7.4.sif bash YOUR/FILE.sh' | qsub -l walltime=60:00:00 -l nodes=1:ppn=8 -A PAS####
```
NOTE - you do not need to submit with the container active
