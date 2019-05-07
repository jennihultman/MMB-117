We have now OTUs but how abundant are they in your samples? For this we need to make OTU table. The usearch command `otutab` requires plenty of memory so we need to run it as batch job. 

nano otutab.batch

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
