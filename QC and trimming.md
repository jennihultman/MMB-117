# Quality control and filtering

Log into Taito, either with ssh (Mac/Linux) or PuTTy (Windows)

open interactive node
```
sinteractive
```

## Data download
First set up the course directory, make some folders and then download the data.  
Move to work directory (wrk) at CSC and make a course directory there. Or a subfolder under a previous course folder.  
```
cd $WRKDIR
mkdir MMB117
cd MMB117
mkdir raw_data
mkdir scripts
mkdir trimmed_data
```

Download the training data (takes few minutes)  
```
cd raw_data
cp /wrk/hultman/shared/mmb117.tar.gz .
```

The md5 sum for the file is 1d77f950aff0ccb4773498e7ac5df689. Check that the md5 um for the file you downloaded matches by typing

```
md5sum mmb117.tar.gz
```
And then unpack the tar.gz file
```
tar -xzvf mmb117.tar.gz
``` 
**FastQC & MultiQC**  
Two programs for sequence data quality control. Both will be installed using Bioconda package management tool that can be found from CSC.  
When using Bioconda at CSC, everything needs to be installed in virtual enviroments. You can create the virtual environment called `QC_env` and install the packages with one command.  
```
module load bioconda/3
conda create -n QC_env multiqc fastqc
```

The environment can be activate with the command `source activate QC_env`. And deactivated with `source deactivate`.  
For now, just create the environment, we will need it soon.


## QC and trimming
QC for the raw data (takes few min, depending on the allocation).  
Go to the folder that contains the raw data and make a folder called e.g. `FASTQC` for the QC reports.  
Then run the QC for the raw data and combine the results to one report using `multiqc`.  
Can be done on the interactive nodes using `sinteractive`. 

## activate the QC environment
```
module load bioconda/3
source activate QC_env
```

## Run fastqc
```
fastqc ./*.fastq.gz -o FASTQC/ -t 4
```

## Then combine the reports with multiqc
```
multiqc ./ --interactive
```

## deactivate the virtual env
```
source deactivate

```

Copy the resulting HTML file to your local machine with `scp` from the command line (Mac/Linux) or *WinSCP* on Windows.  
Have a look at the QC report with your favorite browser.  

After inspecting the output, it should be clear that we need to do some trimming.  

Make a folder for the trimmed data (trimmed_data).  

Specify the adapter sequences that you want to trim after `-a` and `-A`. What is the difference with `-a` and `-A`?

Option `-q` is for quality trimming (PHRED score).  
Check that the paths are correct.  
Cutadapt [manual.](http://cutadapt.readthedocs.io)  

## Run Cutadapt to all of your files (see below option for batch job)
adapter sequences are in this case the four reverse and four forward primers. Make two fasta files with Nano

forward.fasta

>llum_341F_1
CCTACGGGNGGCWGCAG
>Illum_341F_2
gtCCTACGGGNGGCWGCAG
>Illum_341F_3
agagCCTACGGGNGGCWGCAG
>Illum_341F_4
tagtgtCCTACGGGNGGCWGCAG

reverse.fasta
>Illum_785R_1
GACTACHVGGGTATCTAATCC
>Illum_785R_2
aGACTACHVGGGTATCTAATCC
>Illum_785R_3
tctGACTACHVGGGTATCTAATCC
>Illum_785R_4
ctgagtgGACTACHVGGGTATCTAATCC

Save Nano with `ctrl-o` and exit with `ctrl-x`
 
```
cutadapt -m 1 -e 0.2 -O 15 -g file:forward.fasta -G file:reverse.fasta-q 25 your_R1_001.fastq.gz your_R2_001.fastq.gz -o your_R1_001_adapter_trimmed.fastq -p your_R2_001_adapter_trimmed.fastq > log_cutadapt.txt
```
Then let's check the results from the trimming. Go to the folder containing the trimmed reads and make a new folder for the QC files.  
Run FASTQC and MultiQC again.  

```

# activate the QC environment
module load bioconda/3
source activate QC_env
# run QC on the trimmed reads
fastqc ./*.fastq -o FASTQC/ -t 4
 multiqc ./ --interactive
# deactivate the virtual env
source deactivate
# log out from the computing node
exit
# and free the resources after the job is done
exit
```

Copy it to your local machine as earlier and look how well the trimming went.  

## joining forward and reverse reads
Finally we join the paired end reads with program Pear. First you'll need to install it. What was the best location for programs?

```
#change "sample" to your sample name
/homeappl/home/hultman/appl_taito/pear-0.9.10-bin-64/pear-0.9.10-bin-64 -y 400M -j 2 -f sample_R1.adapter_trim.fastq -r sample_R2.adapter_trim.fastq -o sample.pear
```


##Optional: run cutadapt as a batch job (script from Antti Karkman). For this you'll need list of your sample names
```
ls *.fastq.gz |awk -F "-" '{print $2}'|uniq > ../sample_names.txt
```

```

#!/bin/bash

while read i
do
        cutadapt  -a CTGTCTCTTATACACATCTCCGAGCCCACGAGAC -A CTGTCTCTTATACACATCTGACGCTGCCGACGA -q 28 -O 10 \
        -o ../trimmed_data/$i"_R1_trimmed.fastq" -p ../trimmed_data/$i"_R2_trimmed.fastq" \
        *$i*_R1*.fastq.gz *$i*_R2*.fastq.gz > ../trimmed_data/$i"_trim.log"
done < $1
```
Then we need a batch job file to submit the job to the SLURM system. More about CSC batch jobs here: https://research.csc.fi/taito-batch-jobs  
Make another file with text editor.
```
#!/bin/bash -l
#SBATCH -J cutadapt
#SBATCH -o cutadapt_out_%j.txt
#SBATCH -e cutadapt_err_%j.txt
#SBATCH -t 01:00:00
#SBATCH -n 1
#SBATCH -p serial
#SBATCH --mem=50
#

module load biokit
cd $WRKDIR/BioInfo_course/raw_data
bash ../scripts/cutadapt.sh ../sample_names.txt
```
After it is done, we can submit it to the SLURM system. Do it from the course main folder, so go one step back in your folders.  

`sbatch scripts/cut_batch.sh`  

You can check the status of your job with:  

`squeue -l -u $USER`  

After the job has finished, you can see how much resources it actually used and how many billing units were consumed. `JOBID` is the number after the batch job error and output files.  

`seff JOBID`  

