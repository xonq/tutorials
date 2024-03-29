# Funannotate *de novo* annotation software setup and use

## GETTING STARTED
[Slot Lab Annotation Pipeline](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md)

[Funannotate](https://funannotate.readthedocs.io/en/latest/install.html) is an all-in-one genome annotation software that wraps and curates output from multiple *ab initio* gene prediction software.

Only two external inputs are necessary: [a *de novo*
assembly](https://gitlab.com/xonq/tutorials/-/blob/master/assembly.md) and [a
*de novo* Repeat library
fasta](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md). If
you want to incorporate RNA-seq data, update your [Funannotate gff3](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#adding-transcript-reads). If you are using the Ohio Super Computer (OSC) and have access to PAS1046, skip installation to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use) once you have read the [pipeline introduction](https://gitlab.com/xonq/tutorials/-/blob/master/annotationPipeline.md#getting-started).

---

<br /><br /><br />

## INSTALLING
#### Install GeneMark

Accept the license for [GeneMark-ES/ET/EP](https://topaz.gatech.edu/GeneMark/license_download.cgi), download the program, and transfer the use key to your home directory as `.gm_key`

<br />

#### Pull the Funannotate Container
 
Use
[Singularity](https://gitlab.com/xonq/tutorials/-/blob/master/containers.md)
(or Docker) to pull the prebuilt Funannotate container. NOTE: The
`funannoate_mask` container referenced in the RepeatModeler step 
is NOT the same as the container pulled here. A future update will resolve
this. 

```
singularity pull docker://xonq/funannotate:latest
```

Create a UTF-8 text file with the following information, fill it in, and save
it in the folder with the container `.img` as `source.sh`. Everytime you wish
to use the container, you must run `source <PATH/TO>/source.sh` to point
Funannotate to the binaries. NOTE: if you do not have Augustus installed,
run `mkdir -p <INSTALL_LOC>/augustus/config/species` and set
`export AUGUSTUS_CONFIG_PATH=<INSTALL_LOC>/augustus/`

```
# activate funannotate software
. /opt/conda/etc/profile.d/conda.sh
conda activate base

# export path dependencies for databases, genemark, and augustus/busco species databases
export FUNANNOTATE_DB=		# path to funannotate database
export GENEMARK_PATH=		# path to genemark program directory
#export AUGUSTUS_CONFIG_PATH=	# path to augustus species configuration (only specify if using a non-default one)
export PATH=$PATH:		# path to genemark program directory
```



Install databases and exit the container
```
<CONTAINER.sif>
funannotate setup -d <DATABASE/DIRECTORY>
exit
```

<br />

Continue to [OSC USE](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#osc-use) and follow instructions, changing path references when necessary

<br /><br /><br />

## OSC USE
#### First use
GeneMark is one of the *ab initio* gene prediction software Funannotate calls. Upon first use or every 400 days, accept the license and download the 64-bit key (NOT THE PROGRAM) at this link for [GeneMark-ES/ET/EP](http://topaz.gatech.edu/GeneMark/license_download.cgi). Then we will transfer the key to your home directory. These keys expire every 400 days.

Once the key is uploaded, uncompress it and rename it in your home folder as `.gm_key`:
```
gunzip gm_key_64.gz
mv gm_key_64 ~/.gm_key
```

<br />

Let's get started with gene prediction. First we have to interface with Funannotate. Like other parts of the pipeline, Funannotate is in a *container* of software. Furthermore, Funannotate also requires that we run a `source` command upon activating the container - this adds software to our `PATH` which essentially is telling the computer the default places to look for files. Only use the container to run Funannotate commands, otherwise you will experience unexpected behavior. To deactivate the container, press CTRL + D or run `exit`. 
```
/fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate --help
```

If you entered it all correct and do not see the help menu then stop and contact me.

---

<br /><br />

### 0. Keep it clean
I am once again reminding you how important it is to keep your directory structures clean. Following what we have done in previous tutorials, we are ultimately going to output to our `<ORGANISM_CODENAME>` directory. If `<OUTPUT>/<ORGRANISM_CODENAME>` doesn't exist then choose an `<OUTPUT>` folder that already exists, then `mkdir <OUTPUT>/<ORGANISM_CODENAME>`.

```
mkdir <OUTPUT>/<ORGANISM_CODENAME>/funannotate_prep
```

<br /><br />

### 1. Sort assembly (~ 1 min)
Sort your assembly: this renames contigs to `scaffold`, removes contigs below a minimum length (1 kb), and sorts from largest to smallest. Remember to [activate and source](https://gitlab.com/xonq/tutorials/-/blob/master/funannotate.md#activating-funannotate-container) the container to have access to Funannotate. This command can be run without submitting a job. If you followed the previous tutorial, your assembly will be in `<ORGANISM_CODENAME>/results`:
```
funannotate sort -i <YOUR/ASSEMBLY> -o <ORGANISM_CODENAME>/funannotate_prep/<ASSEMBLY.sort.fa> --minlen 1000
```

<br /><br />

### 2. Soft-mask assembly  (~ 1-3 hrs)

Soft-mask the assembly: this references your [*de novo* repeat library](https://gitlab.com/xonq/tutorials/-/blob/master/repeatmodeler.md) to lowercase masked nucleotides. If you followed the previous tutorials' directory structure, the file will be in `<ORGANISM_CODENAME>/results`. 

We are now going to submit a job that invokes the singularity container, so we need to write the shell, `.sh` execution script in plain-text UTF-8 format and save it as something meaningful like `<ORGANISM>_mask.sh`:
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh
funannotate mask -i <YOUR/SORTED_ASSEMBLY> -m repeatmodeler \
-l <YOUR/REPEAT_LIBRARY> \
-o <ORGANISM_CODENAME/funannotate_prep/ASSEMBLY.masked.fa --cpus 6
```

If you make the `.sh` file on your local computer, you'll have to upload it to the supercomputer. Then submit to the job node by invoking singularity to reference your `.sh` file like so:
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif /bin/bash <YOUR/SH_FILE>' | sbatch --time=10:00:00 --nodes=1 --ntasks-per-node=6 -A <PROJECT> --job-name=softmask
```

<br /><br />

### 3. Generate a *preliminary* BUSCO database (~ 2-5 hrs)
BUSCO is used by Funannotate to train gene prediction software. We will create
a preliminary BUSCO database for your organism here. In step 4, Funannotate
will create a finalized BUSCO database. Only use this preliminary BUSCO
database you are making here for this species.


Create a plain text `.sh` file with the BUSCO
command (busco will make an output for you).
`<LINEAGE>` is `ascomycota` or `basidiomycota`, depending on your
organism. You can further refine an Ascomycete lineage dataset by running
`funannotate database --show-buscos` and choosing a more refined `<LINEAGE>`. 
```
source /fs/project/PAS1046/software/containers/funannotate/source.sh

cd <ORGANISM_OUTPUT>

python /venv/lib/python3.8/site-packages/funannotate/aux_scripts/funannotate-BUSCO2.py \
--local_augustus $AUGUSTUS_CONFIG_PATH --long --tarzip \
-i <YOUR/MASKED/ASSEMBLY.fa> -o <ORGANISM_CODE>_prelim --tmp <YOUR/SCRATCH> \
-l /fs/project/PAS1046/databases/funannotate/<LINEAGE> -m genome -c 8
```

Execute the command:
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash
<YOUR/BUSCO.sh>' | sbatch --time=20:00:00 --nodes=1 --ntasks-per-node=8 -A PAS<####> --job-name=busco
```

<br />

Once finished, we are going to capture the unique BUSCO tag for your organism
and print it out (take note of the name for later!). This will print a name like
`BUSCO_galpol1_prelim_3302707098`. If you have trouble, you could also navigate to `<OUTPUT>/run_<ORGANISM_CODE>_prelim/augustus_output/retraining_parameters` and grab the basename similar to the aforementioned name and replace `$busconame` in the following command.
```
busconame=$(find <OUTPUT>/run_<ORGANISM_CODE>_prelim | grep -Po "BUSCO.*(?=_exon_*)")
echo $busconame
```

Now, grant permissions and copy the resulting database to the lab BUSCO
database set.
```
output=<OUTPUT>/run_<ORGANISM_CODE>/augustus_output/retraining_parameters/
chmod -R 770 $output
cp -r $output /fs/project/PAS1046/software/augustus/config/species/$busconame
```

Finally, source and activate the container and update Funannotate so it can find your BUSCO db:
```
funannotate setup -u -w
chmod -R 774 $FUNANNOTATE_DB/outgroups/
```

NOTE: If you do not run the second part of the above command, then others will
encounter a `PermissionError` while running Funannotate. Please make sure to
execute these two commands together *everytime* you run `funannotate setup -u`

If you are getting a `PermissionError` while running `funannotate setup -u`, then it
indicates that someone did not run `chmod`. -_- Contact Zach or your container administrator.



<br />

### 4. Predict genes (~ 10-20 hrs)
This is the first prominent plug I'm going to make for installing [Mycotools](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/README.md). With Mycotools and the MycotoolsDB, curated genomic data needed for annotation can easily be copied or automatically downloaded.

Download/compile necessary data and information:

- transcript/EST evidence from the most closely related available organism(s) in the same genus (separate by spaces in command if > 1)
    * If using JGI, acquire the expressed sequence tags (EST) or refined transcripts (NOT allTranscripts)
    * The Mycotools scripts, `jgiDwnld.py`/`ncbiDwnld.py` can download these for you. Create an account at [MycoCosm](https://mycocosm.jgi.doe.gov/mycocosm/home) and/or [NCBI](https://www.ncbi.nlm.nih.gov/), [install mycotools](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/README.md), then follow the [usage guide](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#jgidwnldpy-ncbidwnldpy)
    
- protein evidence from 10-100 closely related organisms (separate by spaces in command)
    * These can be acquired from the lab MycotoolsDB via 
[db2files.py](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#db2filespy).
If acquired this way, the output will be a folder called `faa`; instead of
inputting the files one-by-one in the following command, simply specify
`<DB2FILESOUTPUT>/faa/*faa`.

<br />

Once compiled, create an output directory:
```
mkdir <ORGANISM_CODENAME>/funannotate
```

We are ready for gene prediction. Create a UTF-8 file with the predict command
and save as an `.sh`. This command first sources the environment for the
container, then tells funannotate to input your masked assembly, a name for the
annotation run, transcript evidence, protein evidence, the number of CPU cores
you use, the BUSCO database name and lineage from step 3, and the output folder you made above. It is also important to choose a brief, descriptive codename/name for your species in `<OME>_<RUN#>`:
```bash
source /fs/project/PAS1046/software/containers/funannotate/source.sh

funannotate predict -i <YOUR/MASKED_ASSEMBLY> -s <OME>_<RUN#> \
--transcript_evidence <YOUR/TRANSCRIPT_EVIDENCE1> <YOUR/TRANSCRIPT_EVIDENCEn> \
--protein_evidence <YOUR/PROTEIN_EVIDENCE1> <YOUR/PROTEIN_EVIDENCEn> \
/fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
--cpus 8 --busco_seed_species <BUSCONAME> --optimize_augustus \
--busco_db <LINEAGE> -o <ORGANISM_CODE>/funannotate
```

<br />

Edit the below to submit the `.sh` file as a job in the funannotate container. Remember, you don't need to activate the container to submit a job that uses the container.
```
echo -e 'singularity exec /fs/project/PAS1046/software/containers/funannotate/funannotate_mask.sif bash <YOUR/CMDFILE>' | sbatch --time=20:00:00 --nodes=1 --ntasks-per-node=8 -A PAS<####> --job-name=funannotate
```

<br /><br />

## OUTPUT
Funannotate will output many working files and an organized folder of results.
The naming scheme of the output directory is intuitive: the gene coordinate
file is the `gff3`, the protein fasta is `proteins.aa.fasta`, etc etc. The
finalized BUSCO results can be found in
`<FUNANNOTATE>/predict_misc/busco/run_<OME>_<RUN#>/short_summary*.txt`

You can obtain a summary of your annotation statistics using the [annotationStats.py](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#sequence-data-statistics): There really is not an objective metric to evaluate annotation quality, but a good check is to compare the the summary statistics, including % of the assembly covered by the total length of the annotation, to similar species. You can use `annotationStats.py` on related species' gffs in the MycotoolsDB.

If you are happy with your annotation quality and BUSCO scores, contribute to the lab and add it to our AUGUSTUS configuration files. Activate and source the container, then:
```
funannotate species -s <ORGANISM>_final -a <OUTPUT>/<ORGANISM_CODENAME>/funannotate/logfiles/<OME>_<RUN#>.parameters.json
```

If you are not running OrthoFiller, you can add your genome to the lab database via [predb2db.py](https://gitlab.com/xonq/mycotools/-/blob/master/mycotools/USAGE.md#predb2dbpy).


<br /><br />

### All at once
Once you have tested the individual steps of Funannotate and are familiar with this portion of the pipeline, you can submit all the commands in one `.sh` file to move a genome through all steps at once. Here is the skeleton of such a file:
```bash
set -e
source /fs/project/PAS1046/software/containers/funannotate/source.sh

# 0. make directories for your outputs
mkdir -p <OUTPUT>/sort_mask <OUTPUT>/funannotate


# 1. sort and remove contigs < 1000 bp
funannotate sort -i <YOUR/ASSEMBLY> --minlen 1000 \
-o <OUTPUT>/sort_mask/<OME>_sort.fa


# 2. soft-mask nucleotides
funannotate mask -i <OUTPUT>/sort_mask/<OME>_sort.fa -m repeatmodeler \
-l <YOUR/REPEAT-LIBRARY.fa> -o <OUTPUT>/sort_mask/<OME>_mask.fa


# 3a. run BUSCO
cd <OUTPUT>

python /venv/lib/python3.8/site-packages/funannotate/aux_scripts/funannotate-BUSCO2.py \
--local_augustus $AUGUSTUS_CONFIG_PATH --long --tarzip --tmp <YOUR/SCRATCH> \
-i <OUTPUT>/sort_mask/<OME>_mask.fa -o <OME>_prelim \
-l /fs/project/PAS1046/databases/funannotate/<LINEAGE> -m genome -c 8


# 3b. add BUSCO to database
busconame=$(find run_<ORGANISM>_prelim | grep -Po "BUSCO.*(?=_exon_*)"

cp -r <OUTPUT>/run_<ORGANISM>/augustus_output/retraining_parameters \
$AUGUSTUS_CONFIG_PATH/species/$busconame
funannotate setup -u -w
chmod -R 774 $FUNANNOTATE_DB/outgroups


# 4. gene prediction
funannotate predict -i <MASKED/ASSEMBLY> -s "<OME_RUN#>" --transcript_evidence <YOUR/EVIDENCE> \
--protein_evidence <YOUR/EVIDENCE1> <YOUR/EVIDENCEn> \
/fs/project/PAS1046/databases/funannotate/uniprot_sprot.fasta \
--cpus 8 --busco_seed_species $busconame -o <OUTPUT>/funannotate \
--optimize_augustus --busco_db <LINEAGE>
```

Save as an `.sh`, transfer to OSC, and submit the script as previously
described (allocate enough resources). If you encounter an error or failure
while running all at once then you will have to pinpoint which step failed,
then comment (add `#`) the out/delete the lines before the failure to begin at
the failure step.

<br /><br />

## ADDING TRANSCRIPT READS
Funannotate makes it extremely easy to update a de novo annotation with direct
transcript reads - no additional steps necessary. If you
have multiple transcript samples, concatenate the same numbered read sets for
each set of paired-end sequences; for other sequencing types, concatenate as
well:

```
zcat <SET1_READ_1>.fq.gz <SETn_READ_1>.fq.gz | gzip > <OME>_1.fq.gz
```

Repeat this for the other sets of reads and input the results (`<OME>_n.fq.gz`)
into `-l/-r` in the command below.

To update your annotation with the paired end transcript reads (use the standard singularity submission scripts to submit as a job):

```
funannotate update -f <SORTED_MASKED>.fa \
-g <FUNANNOTATE_OUTPUT>/predict_results/<OUTPUT>.gff3 \
--species "<GENUS> <SPECIES>" --jaccard_clip --cpus <NUM> \
-l <PAIRED_END_1>.fq.gz -r <PAIRED_END_2>.fq.gz
```

For other types of transcript reads and stranded RNA library preparations, refer to 
`funannotate update -h`. If you
would like to set your own output directory with `-o`, just make sure you use
the full absolute path or else you may encounter a PASA `no such file or
directory` error.


<br /><br />

## CITATION INFO

Funannotate is the `wrapper` that packages many other software. In addition to citing Funannotate, you should cite `GlimmerHMM`, `GeneMark ES`, `Snap`, `BUSCO`, and `Augustus`.
