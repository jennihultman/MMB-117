# MMB-117 Environmental Microbiology

Environmental microbiology course at Master's Programme in Microbiology and Microbial Biotechnology, University of Helsinki. Organizer Dr Jenni Hultman from Department of Microbiology (2019 and 2021). Teaching assistant Sirja Viitamäki (2021).


### Schedule  
 

| Time | Description |
| --- | --- |
| __Monday__ |  Intro to Bioinformatics, Linux and CSC (__Jenni__) |
| __Tuesday__ | Read QC and trimming [QC](https://github.com/jjholsa/MMB-117/blob/master/02_QC_trimming.md) (__Jenni__) |
| __Wednesday__ |  Read QC and trimming [QC](https://github.com/jjholsa/MMB-117/blob/master/02_QC_trimming.md)(__Jenni__) |
| __Thursday__ |  Taxonomy, OTU analysis [OTUs_bacteria](https://github.com/jjholsa/MMB-117/blob/master/03_JoiningReads_filtering_OTUs.md), [OTUs_fungi](https://github.com/jjholsa/MMB-117/blob/master/04_JoiningReads_filtering_OTUs_fungi.md)(__Jenni__)|
| __Friday__ | Basic analysis in R (__Jenni__) |
| __Week 2__ |  |
| __Monday__ |  Working with own data (__Jenni__) |
| __Tuesday__ | Working with own data (__Jenni__) |
| __Wednesday__ |  Statistical analysis of microbial community data (__Eeva Eronen-Rasimus__) |
| __Thursday__ |  Working with own data/preparation of seminar|
| __Friday__ | Seminar, Q&A, feedback |


### How to quit ongoing interactive job

1. if you are the interactive window, type 'exit'
2. if you are not in the window

```
squeue -u username
scancel JOBID
```

username= your csc username

JOBID = task number shown after squeue is run
