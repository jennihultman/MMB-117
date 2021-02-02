Let's repeat pear and vsearch for fungi. To keep things clear, move the trimmed fungal reads to your *trimmed* folder. Bring also your fungal mapping file here! All tasks today will be performed there. Remember where the folder is? If you have already moved them, you can continue.


# Joining forward and reverse reads

Next we join the paired end reads with program Pear. Pear is already installed so just load bioconda 

```
module load bioconda/3


```
And your `pear`environment:
```
source activate pear
```

As you already know how pear works, use the script right away. Copy script `batch_pear.py` from Jenni's folder `/scratch/project_2003853/JENNI`. 

run script with
```
./batch_pear.py mapping.txt
```
How much of the reads assembled? How about the negative control?

Next check the size dostribution. Lot more variation than in bacteria right?
```
prinseq-lite.pl -fastq *fastq  -stats_all
```

## Filtering reads and transforming them to fasta format
From now on we will use vsearch tools wich are free version of usearch from Robert Edgar. Read more at https://drive5.com/usearch/ and https://manpages.debian.org/testing/vsearch/vsearch.1.en.html. Vsearch work in biokit environment which you can load by typing


```
module load biokit
```
then run vsearch as batch for all of your samples with the script below. Only difference to bacteria in the ```--fastq-minlen``` which is 200 bp. Copy the script to nano and save. Make sure you have mapping file copied to the filder where you run it. Run program after giving access (remmeber how chmod worked?) by typing ./whatevernameyouhadforyourscript

```
#!/bin/bash

while read i
do
        arr=($i)
        vsearch --fastq_filter ${arr[0]}_pear.assembled.fastq --fastq_maxee 1 --relabel @ --fastq_minlen 200 --fastaout ${arr[0]}_pear.assembled.vsearch.trimmed.fasta
done < mapping.txt
```

## Add sample name to sequences
In order to analyze OTUs in samples we need to combine all of the sequence data. before this let's add sample identiier to each sequence file (pear assembled, trimmed sequences) with command `sed`. NB! Instead of sample1 have an unique sample identifier for each sample. 

```
sed "s/>@*/>barcodelabel=sample1;read=/g"  sample1_pear.assembled.vsearch.trimmed.fasta >  sample1_renamed
```
Finally we can join our sequences together

```
cat *_renamed > all.fungi.trimmed.renamed.fasta
```


## Find unique read sequences and abundances
```
vsearch --derep_fulllength all.fungi.trimmed.renamed.fasta --threads 2 --minuniquesize 2 --sizeout --output fun_uniques.fa
```

View the output file `fun_uniques.fasta` with less. What does the `size=` refer to? What is the largest size? Exit less with `q`

## Remove chimera

Number of chimeric sequences can be even 25% in amplicon studies. There are tens of different algoritms available and on this course we will use de novo chimera detection program uchime by Robert Edgar. Read more here https://drive5.com/usearch/manual/cmd_uchime3_denovo.html. 

```
vsearch --uchime3_denovo fun_uniques.fa --nonchimeras ITS_nochimeras.fasta --uchimeout ITS_uchime.out --threads 2

```
How much did you have chimeras?

## Make OTUs

```
vsearch --cluster_fast ITS_nochimeras.fasta --id 0.97 --centroids ITS_OTUs.fasta --relabel OTU --uc ITSclusters.uc
```
How many OTUs did you have? 

## Map trimmed reads to OTUs

Last we want to know in which samples the OTUs were present and in which abundance. For this we need to map the reads back to the OTUs. 


```
vsearch --usearch_global all.fungi.trimmed.renamed.fasta --db ITS_OTUs.fasta --strand plus --id 0.97 --uc ITS_OTUtab.uc --otutabout ITS_OTUs.txt
```
