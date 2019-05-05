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

The md5 sum for the file is xxxx. Check that the md5 um for the file you downloaded matches by typing

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

Detach from the installations screen with `Ctrl a + d`.  

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
 

```
cutadapt your_1_001.fastq your_2_001.fastq -a  AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC -A ATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT  -o your_1_001_adapter_trimmed.fastq -p your_2_001_adapter_trimmed.fastq
```
then run fastqc and multiqc to the trimmed data

```

fastqc *1_001_adapter_trimmed.fastq -o fastqc/.
fastqc *2_001_adapter_trimmed.fastq -o fastqc/.


multiqc fastqc/.*adapter_trimmed.zip -n multiqc_adapter_trimmed


multiqc * -n multiqc_all_files

```

Then let's check the results from the trimming. Go to the folder containing the trimmed reads and make a new folder for the QC files.  
Allocate some resources and then run FASTQC and MultiQC again.  
```
salloc -n 1 --cpus-per-task=6 --mem=3000 --nodes=1 -t 00:30:00 -p serial
srun --pty $SHELL
# activate the QC environment
module load bioconda/3
source activate QC_env
# run QC on the trimmed reads
fastqc ./*.fastq -o FASTQC/ -t 6
 multiqc ./ --interactive
# deactivate the virtual env
source deactivate
# log out from the computing node
exit
# and free the resources after the job is done
exit
```

Copy it to your local machine as earlier and look how well the trimming went.  



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

