Read Quality Control and Trimming
================

To perform data quality control, we will use ``FastQC``, a tool that processes reads and presents visual reports.

First, peak into the first few lines of ``.fastq`` files for the Illumina reads from 60X sequencing run
::

 cd ~/wgs_assembly/data/reads/illumina/
 head -n 10 MTB_illumina60X_R1.fastq
 head -n 10 MTB_illumina60X_R2.fastq


What can you tell about the headers? What is the ``read length`` for this run? Can you quickly count the number of reads (Hint: use ``grep`` and ``wc``)?

Running ``fastqc``
---------------
We will now run ``fastqc``. For most tools used in practicals, you can get help by using ``-h`` or ``-help`` flag. Type the following to get help for ``fastqc``::

  fastqc -h

Run ``fastqc`` for ``forward`` reads from 60X and 20X runs saving the results under your results directory::

  fastqc -o ~/wgs_assembly/results/qc ./MTB_illumina60X_R1.fastq 
  fastqc -o ~/wgs_assembly/results/qc ./MTB_illumina20X_R1.fastq 

Now run it again for ``reverse`` reads::
  
  fastqc -o ~/wgs_assembly/results/qc ./MTB_illumina60X_R2.fastq 
  fastqc -o ~/wgs_assembly/results/qc ./MTB_illumina20X_R2.fastq 

Repeat for forward and reverse reads from the PacBio run::
  
  cd ~/wgs_assembly/data/reads/pacbio
  fastqc -o ~/wgs_assembly/results/qc ./MTB_pacbio_R1.fastq 
  fastqc -o ~/wgs_assembly/results/qc ./MTB_pacbio_R2.fastq 


``fastqc`` performs a suite of tests and then for each indicates whether the test has been passed (green tick) or failed (red cross). Notice tough that ``fastqc`` has no idea about what your library should look like. All of its tests are based on a completely random library with 50% GC content and therefore it will show the test as failed if your sample doesn't match these assumptions. For instance, you might have reads sampled from a high AT or high GC genome, and it will fail per sequence GC content. You need to be aware of how your library been generated and interpret the ``fastqc`` reports taking that into account.

Question: For a whole genome shotgun library, what would you expect "per base sequence content" to look like?
Question: What if you had amplicon sequences from a multiplex PCR assay?

Now, open ``fastqc`` reports in a browser::

  firefox ~/wgs_assembly/results/qc/MTB_illumina60X_R1_fastqc.html
  firefox ~/wgs_assembly/results/qc/MTB_illumina60X_R2_fastqc.html
  firefox ~/wgs_assembly/results/qc/MTB_illumina20X_R1_fastqc.html
  firefox ~/wgs_assembly/results/qc/MTB_illumina20X_R2_fastqc.html
  firefox ~/wgs_assembly/results/qc/MTB_pacbio_R1_fastqc.html
  firefox ~/wgs_assembly/results/qc/MTB_pacbio_R2_fastqc.html

* Per base sequence quality: This is probably one of the most important diagnostics. Inspecting this, you can determine what quality trimming parameters you should use in downstream analysis.
* Per base sequence content: For a randomly sampled genome with a GC content of 50%, we would expect that at any given position within a read, there will be an equal probability of finding a A,C,G, or T base. Our library from *M. tuberculosis* is expected to reflect the GC content, which holds here. There seems to be some minor bias at the start of the read, which may be due to PCR duplicates during amplification or during library preparation.
* Sequence duplication levels: In a whole genome shotgun library, very few reads are expected to occur more than once. In a shotgun library of a targeted region, a low level of duplication can simply be the result of high level of sampling (i.e. high coverage). In addition, duplication might also indicate some kind of enrichment bias (i.e. PCR overamplification)

https://sequencing.qcfail.com/ is a good resource where people post about different ways a sequencing run might have failed.


Running ``sickle`` to quality trim reads:
-----------------------------------------
Now that we have an idea about the quality profile of our reads, we can do quality based trimming of reads. We will use a ``Q-score`` threshold of 30 (-q 30), remove all reads that are shorter than 100 bp after trimming (-l 100), and also trim 10 bp from 5' end just to reduce bias. After trimming, singletons reads (reads whose pair dropped off) are save into a separate file:

::

  sickle pe -t sanger -q 30 -l 100 -n 10 \
  -f ~/wgs_assembly/data/reads/MTB_illumina60X_R1.fastq \
  -r ~/wgs_assembly/data/reads/MTB_illumina60X_R2.fastq \
  -o ~/wgs_assembly/results/qc/MTB_illumina60X_qced_R1.fastq \
  -p ~/wgs_assembly/results/qc/MTB_illumina60X_qced_R2.fastq \
  -s ~/wgs_assembly/results/qc/MTB_illumina60X_qced_single.fastq 1>~/wgs_assembly/results/qc/MTB_illumina60X_sickle.log
  
  sickle pe -t sanger -q 30 -l 100 -n 10 \
  -f ~/wgs_assembly/data/reads/MTB_illumina20X_R1.fastq \
  -r ~/wgs_assembly/data/reads/MTB_illumina20X_R2.fastq \
  -o ~/wgs_assembly/results/qc/MTB_illumina20X_qced_R1.fastq \
  -p ~/wgs_assembly/results/qc/MTB_illumina20X_qced_R2.fastq \
  -s ~/wgs_assembly/results/qc/MTB_illumina20X_qced_single.fastq 1>~/wgs_assembly/results/qc/MTB_illumina20X_sickle.log


