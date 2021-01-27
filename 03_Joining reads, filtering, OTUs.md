Now we have the quality trimmed sequence reads. Remember, garbage in, garbage out. To keep things clear, move the trimmed reads to your *trimmed* folder. All tasks today will be performed there. Remember where the folder is? If you have already moved them, you can continue.


# Joining forward and reverse reads

Next we join the paired end reads with program Pear. First you'll need to install it. 

```
module load bioconda/3

conda create -n pear pear
```
Now you have created `pear`environment. Activate it with 
```
source activate pear
```

EXAMPLE of pear command (don't run this):
```
pear -f R1read.fastq_trim -r R2read.fastq_trim -o sample_pear
```

```
./batch_pear.py mapping.txt
```

After this lets check the length distribution of the reads with program Prinseq

```
prinseq-lite.pl -fastq *fastq  -stats_all
```
And this is so much easier with scripts. Copy script `batch_pear.py` from Jenni's folder `/scratch/project_2003853/JENNI`
. How did you change rights to execute the file?
/
run script with


How much of the reads assembled? How about the negative control?

## Filtering reads and transforming them to fasta format
From now on we will use vsearch tools wich are free version of usearch from Robert Edgar. Read more at https://drive5.com/usearch/ and https://manpages.debian.org/testing/vsearch/vsearch.1.en.html. Vsearch work in biokit environment which you can load by typing


```
module load biokit
```
then run vsearch:

```
vsearch --fastq_filter sample_pear --fastq_maxee 1 --relabel @ --fastq_minlen 350 --fastaout sample1_pear.assembled.vsearch.trimmed.fasta
```
You can also run it with script for all of you samples. Copy the script to nano and save. Make sure you have mapping file copied to the filder where you run it. Run program after giving access (remmeber how chmod worked?) by typing ./whatevernameyouhadforyourscript

```
#!/bin/bash

while read i
do
        arr=($i)
        vsearch --fastq_filter ${arr[0]}_pear.assembled.fastq --fastq_maxee 1 --relabel @ --fastq_minlen 350 --fastaout ${arr[0]}_pear.assembled.vsearch.trimmed.fasta
done < mapping.txt
```

## Add sample name to sequences
In order to analyze OTUs in samples we need to combine all of the sequence data. before this let's add sample identiier to each sequence file (pear assembled, trimmed sequences) with command `sed`. NB! Instead of sample1 have an unique sample identifier for each sample. 

```
sed "s/>@*/>barcodelabel=sample1;read=/g"  sample1_pear.assembled.vsearch.trimmed.fasta >  sample1_renamed
```
Finally we can join our sequences together

´´´
cat `*_renamed` > all.bacteria.trimmed.renamed.fasta
´´´

## Length trimming
Next we will remove the reads that are too short or long

```
vsearch --sortbylength all.bacteria.trimmed.renamed.fasta --maxseqlength 530 --minseqlength 350 --output all.bacteria.trimmed.350-530.fasta

```

## Find unique read sequences and abundances
```
vsearch --derep_fulllength all.bacteria.trimmed.350-530.fasta --threads 2 --minuniquesize 2 --sizeout --output bac_uniques.fa
```

In case this is too memory consuming we can run its as batch job. 

Nano fastx.batch

```
#!/bin/bash -l
#SBATCH --job-name=vsearch
#SBATCH --account=project_2003853
#SBATCH --partition=small
#SBATCH --output=vsearch_out_%j.txt
#SBATCH --error=vsearch_err_%j.txt
#SBATCH --time=01:00:00
#SBATCH --mem-per-cpu=100
#SBATCH --nodes=1  
#SBATCH --cpus-per-task=6
#

module load biokit

vsearch --derep_fulllength all.bacteria.trimmed.350-530.fasta --threads 2 --minuniquesize 2 --sizeout --output bac_uniques.fa
```
After it is done, you can submit it to the SLURM system with `sbatch` command

```
sbatch fastx.batch
```
You can check the status of your job with:

```
squeue -l -u $USER
```
View the output file `bac_uniques.fasta` with less. What does the `size=` refer to? What is the largest size? Exit less with `q`

## Remove chimera

Number of chimeric sequences can be even 25% in amplicon studies. There are tens of different algoritms available and on this course we will use de novo chimera detection program uchime by Robert Edgar. Read more here https://drive5.com/usearch/manual/cmd_uchime3_denovo.html. 

```
vsearch --uchime3_denovo bac_uniques.fa --nonchimeras 16S_nochimeras.fasta --uchimeout 16S_uchime.out --threads 2

```
How much did you have chimeras?

## Make OTUs

```
vsearch --cluster_fast 16S_nochimeras.fasta --id 0.97 --centroids 16S_OTUs.fasta --relabel OTU --uc 16Sclusters.uc
```
How many OTUs did you have? 

## Map trimmed reads to OTUs

Last we want to know in which samples the OTUs were present and in which abundance. For this we need to map the reads back to the OTUs. 


```
vsearch --usearch_global all.bacteria.trimmed.350-530.fasta --db 16S_OTUs.fasta --strand plus --id 0.97 --uc 16S_OTUtab.uc --otutabout 16S_OTUs.txt
```
