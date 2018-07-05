Pre-assembly Quality Control
================================

Before running ``sga preqc``, we need to *interleave* *forward* and *reverse* reads into a single file. *Interleaving* reads mean rearranging *forward* and *reverse* reads into a single file, keeping the original order and for each pair writing *forward* read followed by the *reverse* read.

First, move into ``results/qc`` directory::
 
 cd ~/wgs_assembly/results/qc/

To interleave reads, we will use ``seqtk`` tool:
:: 
 seqtk mergepe MTB_illumina20X_qced_R1.fastq MTB_illumina20X_qced_R2.fastq > MTB_illumina20X_qced_R1R2.fastq
 seqtk mergepe MTB_illumina60X_qced_R1.fastq MTB_illumina60X_qced_R2.fastq > MTB_illumina60X_qced_R1R2.fastq

Index the data using ``sga index``:
::
 sga index -t 4 -a ropebwt MTB_illumina20X_qced_R1R2.fastq
 sga index -t 4 -a ropebwt MTB_illumina60X_qced_R1R2.fastq

Run *sga preqc*:
::
 sga preqc -t 4 MTB_illumina20X_qced_R1R2.fastq > MTB_illumina20X_qced_R1R2.preqc
 sga preqc -t 4 MTB_illumina60X_qced_R1R2.fastq > MTB_illumina60X_qced_R1R2.preqc

Make the report::
 
 sga-preqc-report.py -P genome_size \
 -P branch_classification_error -P branch_classification_repeat -P branch_classification_variant \
 -P de_bruijn_simulation_lengths -P kmer_distribution -P gc_distribution -P fragment_sizes \
 MTB_illumina20X_qced_R1R2.preqc MTB_illumina60X_qced_R1R2.preqc -o report.pdf

Open the PDF report and interpret the results.

Questions:
* Was the genome size estimated correctly?
* What differences do you observe between the 20x (blue) and the 60X (red) datasets?
* Which library will be easier to assemble?