# MetaGenomics


### QC of the samples
The downloaded samples must be processed to ensure data quality. A quality control analysis should be performed to filter out those samples that do not exceed the minimum quality parameters. ***FastQC*** analyzes each *FASTQ* file, while ***MultiQC*** creates a global report of all analyzed files, giving us a very intuitive and easy-to-analyze output.
```
fastqc $file --noextract --t 4 --o $QC/
multiqc $QC/ -o $QC/Report/
```
Once we have analyzed our samples, we have to remove adapters or remove some bases that have a low sequencing quality. There are plenty of tools to process FASTQ files (fastp, cutadapt, etc.). 
