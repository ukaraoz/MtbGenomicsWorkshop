=====================
Data Prep and Read QC
=====================
Let's create the directory structure to hold our results and define some variables for convenience:
::

 mkdir ~/mapping-variantcalling
 MYBASE=/home/mtb_upm/mapping-variantcalling
 MYDATA=$MYBASE/data
 MYRESULTS=$MYBASE/results
 mkdir -p $MYDATA/reads; mkdir -p $MYDATA/ref
 mkdir -p $MYRESULTS/fastqc; mkdir -p $MYRESULTS/log; mkdir -p $MYRESULTS/qc; mkdir -p $MYRESULTS/mapping
 
We will start with a single isolate, *ERS050945* and then scale-up the analysis for all of the 99 isolates.
We will define first a variable for the path to the folder with the source data and the precalculated results. We will then copy the data into our folder structure we created above:
::

 SRCBASE=/home/mtb_upm/mtb_genomics_workshop_data/mapping-variantcalling
 sample=ERS050945
 cp $SRCBASE/data/reads/${sample}_*.fastq $MYDATA/reads/
 cp $SRCBASE/data/ref/* $MYDATA/ref

Generate a ``fastqc`` report:
::

 fastqc $MYDATA/reads/${sample}_R1.fastq
 fastqc $MYDATA/reads/${sample}_R2.fastq

Inspect the read quality profile and pick quality trimming parameters.
Quality trim reads using ``sickle``:
::

 sickle pe -t sanger -q 30 -l 50 -n 5 \
 -f $MYDATA/reads/${sample}_R1.fastq -r $MYDATA/reads/${sample}_R2.fastq \
 -o $MYRESULTS/qc/${sample}_qced_R1.fastq -p $MYRESULTS/qc/${sample}_qced_R2.fastq \
 -s $MYRESULTS/qc/${sample}_single.fastq 1> $MYRESULTS/log/${sample}.sickle.log

Question: How many paired reads remain after quality trimming? (hint: inspect the `sickle` command above and try to guess where that information might be)

 
 
 