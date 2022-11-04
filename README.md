# MetaGenomics


### QC of the samples
The downloaded samples must be processed to ensure data quality. A quality control analysis should be performed to filter out those samples that do not exceed the minimum quality parameters. ***FastQC*** analyzes each FASTQ file, while ***MultiQC*** creates a global report of all analyzed files, giving us a very intuitive and easy-to-analyze output.
```
fastqc $file --noextract --t 4 --o $QC/
multiqc $QC/ -o $QC/Report/
```
Once we have analyzed our samples, we have to remove adapters or remove some bases that have a low sequencing quality. There are plenty of tools to process FASTQ files (fastp, cutadapt, etc.). Here, I show two of them.
```
fastp -i $FILE_R1.fastq.gz -t $LENGTH_TO_CUT_IN_R1 -o $FILTERED_SAMPLE/$FILE_R1.fastq.gz -I $FILE_R2.fastq.gz -T $LENGTH_TO_CUT_IN_R2 -O $FILTERED_SAMPLE/$FILE_R2.fastq.gz
cutadapt -a $ADAPTER_FORWARD -A $ADAPTER_REVERSE -j $NUMBER_OF_CORES -q $MINIMUM_QUALITY_OF_EACH_BASE -o $FILTERED_SAMPLE/$FILE_R1.fastq.gz -p $FILTERED_SAMPLE/$FILE_R2.fastq.gz $FILE_R1.fastq.gz $FILE_R2.fastq.gz
```
