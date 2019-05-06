Now we have the quality trimmed sequence reads. Remember, garbage in, garbage out. 

# Joining forward and reverse reads

Next we join the paired end reads with program Pear. First you'll need to install it. What was the best location for programs?

Pear can be found here: https://cme.h-its.org/exelixis/web/software/pear/doc.html

```
**change "sample" to your sample name ja path to pear to __your__ path**

/homeappl/home/hultman/appl_taito/pear-0.9.10-bin-64/pear-0.9.10-bin-64 -y 400M -j 2 -f sample_R1.adapter_trim.fastq -r sample_R2.adapter_trim.fastq -o sample.pear
```

After this lets check the length distribution of the reads with program Prinseq

```
prinseq-lite.pl -fastq *fastq  -stats_all
```

## Filtering reads and transforming them to fasta format

```
usearch --fastq_filter sample1_pear.assembled.fastq --threads 2 --fastq_maxee 1 --relabel @ --fastq_minlen 350 --fastaout sample1_pear.assembled.usearch.trimmed.fasta -threads 2
```
## Add sample name to sequences
In order to analyze OTUs in samples we need to combine all of the sequence data. before this let's add sample identiier to each sequence file (pear assembled, trimmed sequences) with command `sed`

```

sed "s/>@*/>barcodelabel=sample1;read=/g"  sample1_pear.assembled.vsearch.trimmed.fasta >  sample1_renamed
```
