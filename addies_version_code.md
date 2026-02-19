## Login in OSC, either by using SSH in your terminal, a command line instanc ein your web browser, or using VSCode from OSC
## Check where you are and make sure you create your working directory in the right place
```shell
cd /fs/scratch/PAS3260/Addie/
mkdir Lab_4
```

## Copy files to your working directory, for example:
```shell
cp /fs/scratch/PAS3260/Jonathan/Lab_4/ ?
```

## Go to your working directory and execute:
```shell
cd Lab_4
ls -l
ls -Llh
ls -lh
```
### What do you see?
## the commands list the files in Lab_4 in different ways. -l adds the long listing format which includes some additional
## information, -h makes the file sizes human-readable, and -L references the linked file rather than the symbolic link itself

## There is a md5 file, what are md5 files for? Explain
## md5 files are for checking data integrity

## Let's work with this file:
```shell
md5sum -c Lab5.md5 > Checking.txt
grep "FAIL" Checking.txt
## Parenthesis on integrity checking
touch myDNA.fasta
nano myDNA.fasta
md5sum myDNA.fasta
# Inspect the files
```
## Now, let's explore the files a little:
```shell
head ref_Run1CB3334.fasta
head Rice.gff3
head Run1CB3334.fastq
head Run1CB3334.vcf
head Run1CB3334.fastq.gz
head Run1CB3334.bam
```
### Do you see something weird?
## The fastq.gz and bam files are not human readable

## For `Run1CB3334.fastq.gz` try:
```shell
zcat Run1CB3334.fastq.gz | head
```

## Let's check the fastq file in a little more detail
```shell
head -100 Run1CB3334.fastq | less
```

### How would you count the number of reads in the fastq file?
## You could count the number of lines and divide by 4
```shell
wc -l Run1CB3334.fastq
```
## 7560972

# Is it? Try:
```
zcat Run1CB3334.fastq.gz | echo "$((`wc -l` / 4))"
```
## 1890243

### Why the difference?
## The first time we did not divide by 4. We also used the compressed file this time which is more computationally efficient.
## The $(()) part is telling the computer that we are doing math and to interpret the / as division isntead of part of a file path,
## and the backticks tell the function to evaluate wc -l separately and pass the result to the math part.

### Let's download some data from NCBI
```
module load sratoolkit/3.0.2
```
# What is sratoolkit?
## A set of tools for managing and analyzing sequencing data. SRA stand for sequence read archive

```
fastq-dump --gzip --split-files --readids --origfmt ERR3638927
```
## (a faster option is fasterq-dump, how can you look for information about this command?)
## fasterq-dump --help

```shell
zcat ERR3638927_1.fastq.gz | head
```
## Let's download a SAM file
```shell
sam-dump --output-file ERR3638927.sam ERR3638927
```
To manipulate the SAM file we will need samtools:
```shell
module load samtools/1.21
```
Then, we can make a sorted BAM file:
```shell
samtools view -bS ERR3638927.sam | samtools sort -o ERR3638927_sorted.bam
```
Index it:
```shell
samtools index ERR3638927_sorted.bam
```
Make it a FASTA file:
```shell
samtools fasta ERR3638927_sorted.bam > ERR3638927.fasta
head ERR3638927.fasta
```
And index it:
```shell
samtools faidx ERR3638927.fasta
```
## Finally, let's check the VCF file
```shell
head -100 Run1CB3334.vcf | less
module load vcftools/0.1.16
vcftools --vcf Run1CB3334.vcf
vcftools --vcf Run1CB3334.vcf --maf 0.05
vcftools --vcf Run1CB3334.vcf --maf 0.05 --recode --out Run1CB3334_Filtered_maf05
vcftools --vcf Run1CB3334_Filtered_maf05.recode.vcf
```
## Can we check the VCF file using a container as in Lab 3?
## Yes
```shell
module unload vcftools/0.1.16
vcftools --help
apptainer pull vcftools.sif docker://quay.io/biocontainers/vcftools:0.1.16--pl5321hdcf5f25_9
apptainer exec vcftools.sif vcftools --help
apptainer exec vcftools.sif vcftools --vcf Run1CB3334.vcf --maf 0.05 --recode --out Run1CB3334_Filtered_maf05_from_container
apptainer exec vcftools.sif vcftools --vcf Run1CB3334_Filtered_maf05_from_container.recode.vcf
```
## How could we use a container of samtools?
```shell
module unload samtools/1.21
samtools --help
apptainer pull samtools.sif docker://community.wave.seqera.io/library/samtools:1.21--0d76da7c3cf7751c
apptainer exec samtools.sif samtools --help
```
## Following the same structure as VCFtools, define the commands to sort and index fasta and BAM files using the samtools container
# sort: sort
# index: index
```shell
# make and sort BAM file
apptainer exec samtools.sif samtools view -bS ERR3638927.sam | apptainer exec samtools.sif samtools sort -o ERR3638927_sorted_2.bam
# index BAM file
apptainer exec samtools.sif samtools index ERR3638927_sorted_2.bam
# make and sort fasta file
apptainer exec samtools.sif samtools fasta ERR3638927_sorted_2.bam > ERR3638927_2.fasta
# index BAM file
apptainer exec samtools.sif samtools faidx ERR3638927_2.fasta
```
