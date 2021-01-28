
# OTU analysis

First, open inteactive node with `sinteractive`


We have now OTUs but how abundant are they in your samples? For this we need the OTU table you made yesterday. In your folder you should have 16S_OTUs.fasta and ITS_OTUs.fasta which are the files with the centroid sequences from each OTU (ie. representative seuqnce) and 16S_OTUs.txt and ITS_OTUs.txt which are OTU tables. View OTU table with less. Are there some OTUs that are really abundant in all samples? Some that are in some samples alone?

## How about taxonomy? 

What is the closest match to these OTUs? Which bacteria and fungi? We will use naive abyesian classifier from mothur package and Silva SSU rRNA database for bacteria (https://www.arb-silva.de/documentation/release-138/) and UNITE database for fungi (https://unite.ut.ee/).

So time to assign taxonomy. Mothur is in biokit environment, open it with:

```
module load mothur
```
### Bacteria

Go to first to your training/bacteria/trimmed folder. Remember how that was done? Then type
```
mothur
```
to activate mothur program. In mothur type

```
classify.seqs(fasta=16S_OTUs.fasta, reference=/scratch/project_2003853/JENNI/silva.nr_v138.align, taxonomy=/scratch/project_2003853/JENNI/silva.nr_v138.tax, cutoff=60, processors=4)
```
exith with ```quit()```

### Fungi
Go to first to your training/fungi/trimmed folder. Then type
```
mothur
```
to activate mothur program. 

Get more memory for the task and the open mothur

```
sinteractive --account project_2003853 --time 12:00:00 --mem 3000


classify.seqs(fasta=ITS_OTUs.fasta, reference=/scratch/project_2003853/JENNI/Unite_ITS_02/UNITEv6_sh_99.fasta, taxonomy=/scratch/project_2003853/JENNI/Unite_ITS_02/UNITEv6_sh_99.tax, cutoff=60, processors=4)

```
Again, exit with ```quit()```

## Combination of OTU table and taxonomy

I do this usually in excel as I inspect the contaminants manually. So move `16S_OTUs.txt`, `ITS_OTUs.txt` and `16S_OTUs.nr_v138.wang.taxonomy` as well as `ITS_OTUs.unite.wang.taxonomy` to your computer and open in excel or other spredsheet program. Save as `otutable_16S.txt` and `otutable_ITS.txt` (tab delimited format).

## Removal of contaminants

There is no golden standar for this but I remove the abundant contaminants and not the ones found in one sample only. **it is important to write how contaminants were removed to the report/manuscript/thesis**

# Make metadata file (only for real samples from Viikki)

For R we'll need metadata table. Make this for your own samples and for these test samples. 

# Make biom file for R
Move the `otutable.txt` to Puhti, to your folder. For biom-tool we'll need to activate biokit 

```
module load biokit
```

```
biom convert -i otutab_fromexcel.txt -o otutable.biom --table-type="OTU table" --process-obs-metadata taxonomy

```
