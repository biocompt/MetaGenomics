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
usearch -fastq_mergepairs $SAMPLE_R1 -maxdiffs $MAX_DIFF -fastqout $MERGED/MERGED_SAMPLES.fastq
```
*If samples are single end, we can join all of them in one file*. Besides, *it is recommended to create a report of the merge, as well as of any step, to follow all the steps and ensure the quality of the analysis (--fastx_info to create the report)*
Once the samples have been merged into a single file, the amplification primers are removed. These primers are usually 20 base pairs upstream and downstream.
```
usearch -fastx_truncate $MERGED/MERGED_SAMPLES.fastq -stripleft 20 -stripright 20 -fastqout $MERGED/$TRUNCATED.fastq
```
After that, we filter the samples by a minimum length. This value depends on the mean length of our samples.
```
usearch -fastq_filter $MERGED/$TRUNCATED.fastq -relabel filt -fastq_minlen $ACCEPTABLE_MIN_LENGTH -fastaout $MERGED/$FILTERED.fa
```
Then, we just keep unique sequences.
```
usearch -fastx_uniques $MERGED/$FILTERED.fa -sizeout -relabel Uniq -strand both -fastaout $MERGED/$UNIQUES.fasta
```
We de-noise these unique sequences, removing chimeras and correcting reads with sequencing errors.
```
usearch -unoise3 $MERGED/$UNIQUES.fasta -zotus $ZOTUS/ZOTUS.fa
```
And, lastly, we sort the zotus by length and applied a second filtering by minimum sequence length.
```
usearch -sortbylength $ZOTUS/ZOTUS.fa -minseqlength 175 -fastaout $ZOTUS/SORTED_ZOTUS.fa
```
Now we are ready to run the metagenomic analysis.
### 3. Run the metagenomic analysis
In this step we proceed to identified the species in our sample, or at least, the gender/family of the micro-organism that we could not identified. First, we clusterized the zotus to minimize memory use.
```
usearch -cluster_smallmem $ZOTUS/SORTED_ZOTUS.fa -id 0.99 -centroids $ZOTUS/ZOTUS_99.fa
```
Then, we create the OTU table to which map our reads to each of the clustered zotus.
```
usearch -otutab $MERGED/$TRUNCATED.fastq -otus $ZOTUS/ZOTUS_99.fa -id 0.97 -otutabout $RESULTS/$OTU_TAB.tsv
```
Now, we are ready to identify the microorganism by blasting our sample with a specific database. This database must be downloaded or created with the microorganisms typical of the environment we are studying.
```
usearch -usearch_local $ZOTUS/ZOTUS_99.fa  -db $SPECIFIC_DATABASE -id 0.95 -notmatched $ZOTUS/$ZOTUS_NOT_MATCHED.fa  -strand both -top_hit_only -query_cov 0.90
```
Those samples that could not be identified by specific database, are blasted with a more general database.
```
usearch -sintax $ZOTUS/$ZOTUS_NOT_MATCHED.fa -db $SYNTAX_DATABASE -strand both -tabbedout $RESULTS/$NOT_MATCHED_PREDICTIONS.tsv -sintax_cutoff 0.97
```
### 3. BLAST with ONLINE DATABASES
It is advisable to analyze those samples that have been identified to family/gene level in online databases such as NCBI, uploading our sequence to the web and doing BLAST with specific 16S RNA databases (rRNA/ITS databases).
