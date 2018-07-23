Pre-assembly Quality Control
================================

Before running ``sga preqc``, we need to *interleave* *forward* and *reverse* reads into a single file. *Interleaving* reads mean rearranging *forward* and *reverse* reads into a single file, keeping the original order and for each pair writing *forward* read followed by the *reverse* read.

To interleave reads, we will use ``seqtk`` tool:
:: 
 seqtk mergepe $MYRESULTSDIR/qc/MTB_illumina20X_qced_R1.fastq $MYRESULTSDIR/qc/MTB_illumina20X_qced_R2.fastq > $MYRESULTSDIR/qc/MTB_illumina20X_qced_R1R2.fastq
 seqtk mergepe $MYRESULTSDIR/qc/MTB_illumina60X_qced_R1.fastq $MYRESULTSDIR/qc/MTB_illumina60X_qced_R2.fastq > $MYRESULTSDIR/qc/MTB_illumina60X_qced_R1R2.fastq


The ``sga`` computations will take several hours on a single core for the 60X library. To save time, we won't be doing these computations but simply copy the results from the precalculated directory. Here is how to do these computations (**DONOT RUN**).

First, index the data using ``sga index``:
::
 sga index -t 4 -a ropebwt $MYRESULTSDIR/qc/MTB_illumina20X_qced_R1R2.fastq
 sga index -t 4 -a ropebwt $MYRESULTSDIR/qc/MTB_illumina60X_qced_R1R2.fastq

Run *sga preqc*:
::
 sga preqc -t 4 $MYRESULTSDIR/qc/MTB_illumina20X_qced_R1R2.fastq > $MYRESULTSDIR/qc/MTB_illumina20X_qced_R1R2.preqc
 sga preqc -t 4 $MYRESULTSDIR/qc/MTB_illumina60X_qced_R1R2.fastq > $MYRESULTSDIR/qc/MTB_illumina60X_qced_R1R2.preqc

Copy the precalculated results:
::
 cp /home/mtb_upm/mtb_genomics_workshop_data/genome-assembly-annotation/precalculated/MTB_illumina20X_qced_R1R2.preqc $MYRESULTSDIR/qc
 cp /home/mtb_upm/mtb_genomics_workshop_data/genome-assembly-annotation/precalculated/MTB_illumina60X_qced_R1R2.preqc $MYRESULTSDIR/qc


Make the report::
 
 sga-preqc-report.py -P genome_size \
 -P branch_classification_error -P branch_classification_repeat -P branch_classification_variant \
 -P de_bruijn_simulation_lengths -P kmer_distribution -P gc_distribution -P fragment_sizes \
 MTB_illumina20X_qced_R1R2.preqc MTB_illumina60X_qced_R1R2.preqc -o sga-preqc-report.pdf

Open the PDF report and interpret the results.

Questions:
* Was the genome size estimated correctly?
* What differences do you observe between the 20x (blue) and the 60X (red) datasets?
* Which library will be easier to assemble?