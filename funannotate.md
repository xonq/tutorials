# Funannotate *de novo* annotation software setup and use

## NOTE 
- [Funannotate documentation](https://funannotate.readthedocs.io/en/latest/install.html)
- Fully capitalized paths, like `INSTALLATION/PATH`, need to be manually edited by the user.

If you are using OSC and have access to PAS1046, you should be able to run Funannotate after acquiring a GeneMark use key.

<br />

## OSC USE
#### Accessing GeneMark
Accept the license for [GeneMark-ES/ET/EP](http://topaz.gatech.edu/GeneMark/license_download.cgi), download the 64-bit key (NOT the program), and transfer to your OSC home directory. 

**- uncompress and rename**
```
gunzip gm_key_64.gz
mv gm_key_64 ~/.gm_key
```

NOTE - Keys expire in 400 days and will cause GeneMark errors.

<br />

#### Activating Funannotate Container
**- activate container then source the Funannotate directories to your path**
```
singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
source /fs/project/PAS1046/software/containers/funannotate/source.sh
```

NOTE - Only use to run the container's software. To deactivate press CTRL + D or run `exit`.

<br />

### OPTIONAL: "Clean" assembly
Compile an [assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md). Clean by removing contigs < 1000 bp and with 95% identity to any contig less than the N50.

** - create a text file with the clean command, save as an `.sh` file, and transfer to OSC**
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate clean -i YOUR/ASSEMBLY -o OUTPUT/ASSEMBLY.clean -m 1000
```

**- edit and submit a job to Torque to run that file in the funannotate container**
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash YOUR/CMD.sh' | qsub -l walltime=30:00:00 -l nodes=1:ppn=6 -A PAS#### -N clean
```

<br />

### 1. Soft-mask assembly. 
Compile an [assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md) and [RepeatModeler](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md) library fasta - `$NAME-families.fa`

**- soft-mask the assembly by using RepeatMasker to lowercase masked nucleotides**
```
funannotate mask -i YOUR/ASSEMBLY -m repeatmodeler -l YOUR/REPEAT_LIBRARY.fa -o OUTPUT/MASKED_ASSEMBLY
```
NOTE - the container needs to be active and sourced

<br />

### 2. Gene prediction. 
Download/compile necessary data and information:
- transcript/EST evidence from organism(s) in the same genus
- protein evidence from at least 10 closely related organisms (separate by spaces in command)
- run `funannotate species` to find the most closely related BUSCO species database; funannotate will generate a BUSCO species database for your species

**- create a file with the predict command, save as an `.sh`, and transfer to OSC:**
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

funannotate predict -i YOUR/MASKED_ASSEMBLY -s “OME_RUN#” --transcript_evidence YOUR/TRANSCRIPT_AND_EST_EVIDENCE \
--protein_evidence YOUR/PROTEIN_EVIDENCE /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
-–cpus 8 --busco_seed_species BUSCO_SPECIES -o OUTPUT/FOLDER
```

**- edit and submit a job to Torque to run that file in the funannotate container**
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash YOUR/FILE.sh' | qsub -l walltime=60:00:00 -l nodes=1:ppn=8 -A PAS#### -N funannotate
```
NOTE - you do not need to submit with the container active

<br /><br />

### All at once
Here is the skeleton of a text file that can run through all the previous steps
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

# OPTIONAL: remove contigs < 1000 bp and duplicates < N50 size
#funannotate clean -i YOUR/ASSEMBLY -m 1000 -o CLEAN/ASSEMBLY


# soft-mask nucleotides (specify CLEAN/ASSEMBLY if cleaned)
funannotate mask -i YOUR/ASSEMBLY -m repeatmodeler -l YOUR/REPEAT-LIBRARY.fa -o YOUR/MASKED-ASSEMBLY


# gene prediction
funannotate predict -i YOU/MASKED_ASSEMBLY -s "OME_RUN#" --transcript_evidence YOUR/EVIDENCE \
--protein_evidence YOUR/EVIDENCE1 YOUR/EVIDENCEn /fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
--cpus 8 --busco_seed_species BUSCO_SPECIES -o OUTPUT/FOLDER
```

Save as an `.sh`, transfer to OSC, and submit the script as previously described
