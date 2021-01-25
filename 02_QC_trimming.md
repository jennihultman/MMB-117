# Quality control and filtering

Log into Puhti, either with ssh (Mac/Linux) or PuTTy (Windows)

open interactive node
```
sinteractive
```

## Data download
First set up the course directory, make some folders and then download the data.  
Move to course scratch folder at CSC and make an own directory there.  
```
cd /scratch/project_2003853/
mkdir _yourname_
cd _yourname_
mkdir training
mkdir scripts
mkdir own_data
```

Download the training data (takes few minutes)  
```
cd training
mkdir bacteria
cd bacteria
mkdir raw_data
mkdir trimmed_data
cd raw_data
cp /scratch/project_2002004/1_AMPLICONS/KALA_2016/FASTQ .
```

The md5 sum for the file is 61127d6f03beca35aadcbe587a09cff8. Check that the md5 um for the file you downloaded matches by typing

```
md5sum MMB117.tar.gz
```
And then unpack the tar.gz file
```
tar -xzvf MMB117.tar.gz
``` 

## Make mapping file 
For the sequence analysis pipeline we need a mapping file with information on the sample name and corresponding fastq-files. Use Nano to make the file. You can make this manually with help of this oneliner:
```
ls *fastq > sequence_files

awk 'BEGIN {FS = "-16S" } ; {print $1}'  sequence_files | sort | uniq > sample_names KORJAA!!!!
```

# Quality assessment with FastQC

FastQC is used for sequence read quality control and can be found from CSC under the #biokit environment. You get access to all of the programs under this installation by typing 
```
module load biokit
```

Then let's run FastQC for the raw data (takes few min, depending on the allocation).  
Go to the folder that contains the raw data and make a folder called e.g. `FASTQC` for the QC reports.  


## Run fastqc
```
fastqc ./*.fastq.gz -o FASTQC/
```

Copy the resulting HTML file to your local machine with `scp` from the command line (Mac/Linux) or *WinSCP* on Windows.  
Have a look at the QC report with your favorite browser.  

After inspecting the output, it should be clear that we need to do some trimming. 



# Run Cutadapt to all of your files (see below option for batch job)

Specify the adapter sequences that you want to trim after `-a` and `-A`. What is the difference with `-a` and `-A`?

Option `-q` is for quality trimming (PHRED score).  
Check that the paths are correct.  
Cutadapt [manual.](http://cutadapt.readthedocs.io)  

When looking at the cutadapt manual, which flags (=“-letter”) are for
```
Length trimming	 		
3’ adapter 			
Paired end 3’adapter	
Quality score			
Output name		
Paired end output	
```

Adapter sequences are in this case the four reverse and four forward primers. Make two fasta files with Nano

forward.fasta
```

>llum_341F_1
CCTACGGGNGGCWGCAG
>Illum_341F_2
gtCCTACGGGNGGCWGCAG
>Illum_341F_3
agagCCTACGGGNGGCWGCAG
>Illum_341F_4
tagtgtCCTACGGGNGGCWGCAG
```
reverse.fasta
```
>Illum_785R_1
GACTACHVGGGTATCTAATCC
>Illum_785R_2
aGACTACHVGGGTATCTAATCC
>Illum_785R_3
tctGACTACHVGGGTATCTAATCC
>Illum_785R_4
ctgagtgGACTACHVGGGTATCTAATCC
```

Save Nano with `ctrl-o` and exit with `ctrl-x`
 
```
cutadapt -m 1 -e 0.2 -O 15 -g file:forward.fasta -G file:reverse.fasta -q 25 your_R1_001.fastq.gz your_R2_001.fastq.gz -o your_R1_001_adapter_trimmed.fastq -p your_R2_001_adapter_trimmed.fastq > log_cutadapt.txt
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
```

Copy it to your local machine as earlier and look how well the trimming went.  




## running Cutadapt as batch job

```
#!/bin/bash

while read i
do
  	arr=($i)
        cutadapt -m 1 -e 0.2 -q 25 -O 15 -g file:forward.fasta -G file:reverse.fasta -o ${arr[1]}_trim -p ${arr[2]}_trim ${arr[1]} ${arr[2]}
done < mapping.txt
```


