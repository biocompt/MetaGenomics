# MetaGenomics


### QC of the samples
The downloaded samples must be processed to ensure the quality of the data. A QC analysis must be done to filtering those samples that not pass the minimun quality parameters.

```
fastqc $file --noextract --t 4 --o $QC/
multiqc $QC/ -o $QC/Report/
```
