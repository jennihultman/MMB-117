## OTU analysis
As this will take some time, we will start by downloading Silva 132 database. Download it to your /wrk/$USER/DONOTREMOVE folder so you can use it in the future. https://mothur.org/wiki/Silva_reference_files. Use `wget`command. 

# OTU table
We have now OTUs but how abundant are they in your samples? For this we need to make OTU table. The usearch command `otutab` requires plenty of memory so we need to run it as batch job. 

Do this in the folder where you have `otus.fasta` from Tuesday.
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

After that time to assign taxonomy.

```
module load biokit
```
type
```
mothur
```
to activate mothur program. In mothur type

```
classify.seqs(fasta=otus.fasta, reference=/wrk/yourusername/DONOTREMOVE/silva.nr_v132.align, taxonomy=/wrk/yourusername/DONOTREMOVE/silva.nr_v132.tax, cutoff=60, processors=4)
```
# Combination of OTU table and taxonomy

I do this usually in excel as I inspect the contaminants manually. So move `otutab_raw.txt` and ´otus.nr_v132.wang.taxonomy´to your computer and open in excel or other spredsheet program. Save as `otutable.txt` (tab delimited format).

# Removal of contaminants

There is no golden standar for this but I remove the abundant contaminants and not the ones found in one sample only. **it is important to write how contaminants were removed to the report/manuscript/thesis**

# Make metadata file

For R we'll need metadata table. Make this for your own samples and for these test samples. 

# Make biom file for R
Move the `otutable.txt` to Taito, MMB117 folder.

```
biom convert -i otutable.txt -o table.from_txt_json.biom --table-type="OTU table" --to-json
summarize_taxa.py -i table.from_txt_json.biom 
```
