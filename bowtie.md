###BOWTIE & BOWTIE2
##NOTES
bowtie and bowtie2 are two different programs and one needs to be specified over the other. They each have their advantages in speed depending on the length of the read.

##Build database

`bowtie2-build ~/database/assembly/fibshu1_scaffolds.fasta fibshu1`

##Paired-end alignment

Took about 8 hours. In build directory:
`bowtie2 -x fibshu1 -1 /fs/project/PAS1046/projects/Atheliaceae/fibsh1-oh/illumina_raw/Fibu_USD16090670L_HKFJKDSXX_L1_1.fq.gz -2 /fs/project/PAS1046/projects/Atheliaceae/fibsh1-oh/illumina_raw/Fibu_USD16090670L_HKFJKDSXX_L1_2.fq.gz -S fibshu1.sam`

##Long read alignment

In build directory:
`bowtie2 --local -x fibshu1 -U /fs/project/PAS1046/projects/Atheliaceae/fibsh1-oh/minion/longreads.fq -S fibshu1.long.sam`

##Compress output to parse

Had to submit as a job. Completes in roughly a half hour
`samtools view -bS fibshu1.sam > fibshu1.bam`

##Sort BAM (compresses more)

Took about 5 minutes, but I had to submit the job (-@ specifies additional threads, so request the appropriate number)
`samtools sort fibshu1.bam -o fibshu1.sort.bam -@ 7`

##BAM to nt by nt coverage

`bedtools genomecov -ibam fibshu1.sort.bam -d > fibshu1.ill_cov.bed`

##BAM to BCF conversion for viewing

samtools mpileup -uf ~/database/assembly/fibshu1_scaffolds.fasta fibshu1.sort.bam | bcftools view -Ov - > fibshu1.raw.bcf
