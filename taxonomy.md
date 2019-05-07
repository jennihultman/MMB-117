## OTU analysis

# OTU table
We have now OTUs but how abundant are they in your samples? For this we need to make OTU table. The usearch command `otutab` requires plenty of memory so we need to run it as batch job. 

Do this in your `trimmed_data` folder
```
nano otutab.batch
```

```
#!/bin/bash -l
#SBATCH -J otutab
#SBATCH -o otutab_out_%j.txt
#SBATCH -e otutab_err_%j.txt
#SBATCH -t 01:00:00
#SBATCH -n 1
#SBATCH -p serial
#SBATCH --mem=100
#


usearch -otutab all.assembled.trimmed.renamed.fasta -otus otus.fasta -otutabout otutab_raw.txt

```
# How about taxonomy? 

What is the closest match to these OTUs? Bacteria or archaea? We will use naive abyesian classifier from mothur package and Silva SSU rRNA database.

Before assigning taxonomy you need to download the SIlva database to your DONOTREMOVE folder. https://mothur.org/wiki/Silva_reference_files. Use `wget`command. 

```
module load biokit
```

```
mothur
```

classify.seqs(fasta=otus.fasta, reference=/wrk/yourusername/DONOTREMOVE/silva.nr_v132.align, taxonomy=/wrk/yourusername/DONOTREMOVE/silva.nr_v132.tax, cutoff=60, processors=4)

