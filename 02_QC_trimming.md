# Quality control and filtering

Log into Puhti, either with ssh (Mac/Linux) or PuTTy (Windows)

open interactive node
```
sinteractive
```

you should see something like this
```
Interactive batch job is launched with following resources:
  Maximum run time (hours:min:sec): 24:00:00
  Maximum memory (MB): 2000 
  $TMPDIR size (GB): 32 
  Number of cores/threads: 1 
  Accounting project: project_2003853
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
cp /scratch/project_2003853/JENNI/MMB117.tar.gz .
```

The md5 sum for the file is 61127d6f03beca35aadcbe587a09cff8. Check that the md5 sum for the file you downloaded matches by typing
/MMB117.tar.gz
```
md5sum MMB117.tar.gz
```
And then unpack the tar.gz file. After that check where the files are and move to raw_data if needed.
```
tar -xzvf MMB117.tar.gz
``` 

## Make mapping file 
For the sequence analysis pipeline we need a mapping file with information on the sample name and corresponding fastq-files. Use Nano to make the file. You can get some help from these this oneliners. What do they do?
```
ls *fastq.gz > sequence_files

awk 'BEGIN {FS = "-Hultman" } ; {print $1}'  sequence_files | sort | uniq > sample_names
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

Copy the resulting HTML file to your local machine with `scp` or Cyberduck/Fugu from the command line (Mac/Linux) or *WinSCP* on Windows.  
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
cutadapt -g file:forward.fasta -G file:reverse.fasta -q 25 -m 50 your_R1_001.fastq.gz your_R2_001.fastq.gz -o your_R1_001_adapter_trimmed.fastq -p your_R2_001_adapter_trimmed.fastq > log_cutadapt.txt
```
Then let's check the results from the trimming. Go to the folder containing the trimmed reads and make a new folder for the QC files.  
Run FASTQC again.  

```

# run QC on the trimmed reads
fastqc ./*.fastq -o FASTQC/ -t 4
```

Copy it to your local machine as earlier and look how well the trimming went.


# Next same for fungi

Remeber that we have both bacteria and fungi. What was the diffeence how they were prepped in the lab? ANd what was the region used and was there something that differed in fundi (hint. what were we supposed to see on the gel electrophoresis?)

go to

```
/scratch/project_2003853/yourname/training/

```
and make folder called fungi. 

Download the training data (takes few minutes)  
```

cd fungi
mkdir raw_data
mkdir trimmed_data
cd raw_data
cp /scratch/project_2003853/JENNI/MMB117_fungi.tar.gz .
```

The md5 sum for the file is 9a50dbcc998d90fafd055ea755911b7b. Check that the md5 sum for the file you downloaded matches by typing
/MMB117_fungi.tar.gz
```
md5sum MMB117_fungi.tar.gz
```
And then unpack the tar.gz file. After that check where the files are and move to raw_data if needed.
```
tar -xzvf MMB117_fungi.tar.gz
``` 

## Make mapping file 
For the sequence analysis pipeline we need a mapping file with information on the sample name and corresponding fastq-files. Use Nano to make the file. You can get some help from these this oneliners. What do they do?
```
ls *fastq.gz > sequence_files

awk 'BEGIN {FS = "-Heinonsalo" } ; {print $1}'  sequence_files | sort | uniq > sample_names
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

Copy the resulting HTML file to your local machine with `scp` or Cyberduck/Fugu from the command line (Mac/Linux) or *WinSCP* on Windows.  
Have a look at the QC report with your favorite browser.  

After inspecting the output, it should be clear that we need to do some trimming. 

Adapter sequences are in this case the four reverse and four forward primers. Make two fasta files with Nano

forward.fasta
```

>ITS4_F1
TCCTCCGCTTATTGATATGC
>ITS4_F2
NTCCTCCGCTTATTGATATGC
>ITS4_F3
agtRRTCCTCCGCTTATTGATATGC
```
reverse.fasta
```
>fITS7_R1
GTGARTCATCGAATCTTTG
>fITS7_R2
WGTGARTCATCGAATCTTTG
>fITS7_R3
ccaGTGAnTCATCGAATCTTTG

```

also for reverse complement

forward_rc.fasta
```
>ITS4_F1_rc
GCATATCAATAAGCGGAGGA
>ITS4_F2
GCATATCAATAAGCGGAGGAN
>ITS4_F3
GCATATCAATAAGCGGAGGAnnact
```

reverse_rc.fasta

```
>fITS7_R1_rc
CAAAGATTCGATGAnTCAC
>fITS7_R2_rc
CAAAGATTCGATGAnTCACn
>fITS7_R3_rc
CAAAGATTCGATGAnTCACtgg
```
Save Nano with `ctrl-o` and exit with `ctrl-x`
 
```
cutadapt -g file:forward.fasta -b file:reverse_rc.fasta -G file:reverse.fasta -B file:forward_rc.fasta --max-n 0 --minimum-length 50 -n 2 your_R1_001.fastq.gz your_R2_001.fastq.gz -o your_R1_001_adapter_trimmed.fastq -p your_R2_001_adapter_trimmed.fastq > log_cutadapt.txt
```
Then let's check the results from the trimming. Go to the folder containing the trimmed reads and make a new folder for the QC files.  
Run FASTQC again.  

```

# run QC on the trimmed reads
fastqc ./*.fastq -o FASTQC/ -t 4
```

Copy it to your local machine as earlier and look how well the trimming went.  

  


## running Cutadapt as batch job

#### for bacteria

Make this in nano and save as cutadapt_bac.sh 
```
#!/bin/bash

while read i
do
  	arr=($i)
        cutadapt -m 50 -q 25 -g file:forward.fasta -G file:reverse.fasta -o ${arr[1]}_trim -p ${arr[2]}_trim ${arr[1]} ${arr[2]}
done < mapping.txt
```
You do not have right for the task, so give them with chmod

```
chmod 777 cutadapt_bac.sh

```
After that make sure you are in the folder with your mapping file (names as *mapping.txt*), primer files and raw reads. 

Then run the task in the interactive node

```
./cutadapt_bac.sh

```

### For fungi

name as cutadapt_fun.sh
```
#!/bin/bash

while read i
do
  	arr=($i)
        cutadapt -g file:forward.fasta -b file:reverse_rc.fasta -G file:reverse.fasta -B file:forward_rc.fasta --max-n 0 --minimum-length 50 -n 2 -o ${arr[1]}_trim -p ${arr[2]}_trim ${arr[1]} ${arr[2]}
done < mapping.txt
```

Run as you did for bacteria. NB! Mapping file for fungi is NOT the same as for bacteria. Why?
