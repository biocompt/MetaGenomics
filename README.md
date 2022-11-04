# MetaGenomics


### 1. QC of the samples
The downloaded samples must be processed to ensure data quality. A quality control analysis should be performed to filter out those samples that do not exceed the minimum quality parameters. ***FastQC*** analyzes each FASTQ file, while ***MultiQC*** creates a global report of all analyzed files, giving us a very intuitive and easy-to-analyze output.
```
fastqc $file --noextract --t 4 --o $QC/
multiqc $QC/ -o $QC/Report/
```
Once we have analyzed our samples, we have to remove adapters or remove some bases that have a low sequencing quality. There are plenty of tools to process FASTQ files (fastp, cutadapt, etc.). Here, I show two of them.
```
fastp -i $FILE_R1.fastq.gz -t $LENGTH_TO_CUT_IN_R1 -o $FILTERED_SAMPLE/$FILE_R1.fastq.gz -I $FILE_R2.fastq.gz -T $LENGTH_TO_CUT_IN_R2 -O $FILTERED_SAMPLE/$FILE_R2.fastq.gz -c
cutadapt -a $ADAPTER_FORWARD -A $ADAPTER_REVERSE -j $NUMBER_OF_CORES -q $MINIMUM_QUALITY_OF_EACH_BASE -o $FILTERED_SAMPLE/$FILE_R1.fastq.gz -p $FILTERED_SAMPLE/$FILE_R2.fastq.gz $FILE_R1.fastq.gz $FILE_R2.fastq.gz
```
We then reanalyzed these filtered samples to ensure correct sample processing.
```
fastqc $file --noextract --t 4 --o $QC_FILTERED/
multiqc $QC_FILTERED/ -o $QC_FILTERED/Report/
```
### 2. Merge samples and keep the uniques
In this step, the end-of-pair samples are merged, filtering by a minimum length and eliminating duplicate sequences, keeping only the non-repetitive sequences. So, to merge we only have to specify the R1 files and usearch will search for their pair.
```
usearch -fastq_mergepairs $SAMPLE_R1 -maxdiffs 20 -fastqout $MERGED/MERGED_SAMPLES.fastq
```
*If samples are single end, we can join all of them in one file*

### 2 - Create the report
~/usearch -fastx_info Merged/merged.fastq -output Merged/Stats_merge.txt

### 3 - Remove primers of amplification
~/usearch -fastx_truncate Merged/merged.fastq -stripleft 20 -stripright 20 -fastqout Merged/truncated.fastq

### 4 - Filter the samples
~/usearch -fastq_filter Merged/truncated.fastq -fastaout Merged/filtered.fa -relabel filt -fastq_maxee 1 -fastq_minlen 200

### 5 - Stay with the uniques
~/usearch -fastx_uniques Merged/filtered.fa -fastaout Merged/uniques.fasta -sizeout -relabel Uniq -strand both
```
