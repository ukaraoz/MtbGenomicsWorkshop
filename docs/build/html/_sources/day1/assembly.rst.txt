Assembly
========
We can now proceed with the assembly of short and long reads. We will assemble each library (2 short and 1 long reads library) reseparately and compare the assemblies.

Create a directory to hold the assembly results:
::

 mkdir -p ~/wgs_assembly/results/short_long_reads_assembly
 cd ~/wgs_assembly/results/short_long_reads_assembly


Assembly using Illumina short reads
-----------------------------------
We will assemble the *M. tuberculosis* genome using the *qced* short reads, separately for 20X and 60X libraries. We will be using ``SPAdes`` assembler, which is a ``de Bruijn graph`` based short read assembler.

The parameters that go into a short read assembler can be tricky to optimize and tuning them is quite time consuming. The most important parameter is the ``k-mer`` size to be used to built the ``de Bruijn graph``. ``SPAdes`` makes the parameterization quite easy and will select parameter values automatically.

Run ``SPAdes`` as follows using 4 threads (``-t 4``):
::

 spades.py -o MTB_illumina20X_spades -t 4 -12 ~/wgs_assembly/results/qc/MTB_illumina20X_qced_R1R2.fastq
 spades.py -o MTB_illumina60X_spades -t 4 -12 ~/wgs_assembly/results/qc/MTB_illumina60X_qced_R1R2.fastq

The assembly for the 20X library will be computed quickly but for the 60X library the computation will take about 1 hour. Copy the precalculated assembly directory:
::

 cp -r $DAY1_PRECALCULATED/MTB_illumina60X_spades ./MTB_illumina60X_spades

Assembly using PacBio long reads
--------------------------------
Now, we will generate an assembly of the *M. tuberculosis* genome using PacBio long reads. Longer reads are better at resolving repeats and therefor will typically give more contiguous assemblies compared to short reads. However, long reads have a significantly higher error rate so consensus base qualities are expected to be much lower.

To assemble long reads, we will use ``canu``, an ``overlap-layour-consensus`` assembler. Generate the assembly with the following command (this will take about ~30 min.).
::
 canu gnuplotTested=true -p MTB_pacbio_canu -d MTB_pacbio_canu_auto genomeSize=4.4m -pacbio-raw MTB_pacbio_circular-consensus-sequence-reads.fastq

 Copy the precomputed assembly:
 ::

 cp -r $DAY1_PRECALCULATED/MTB_pacbio_canu_auto ./MTB_pacbio_canu_auto

Now you should have 3 separate *M. tuberculosis* assemblies. Copy the ``fasta`` files for the assembled contigs into a separate directory:
::
 mkdir ~/wgs_assembly/results/short_long_reads_assembly/assemblies
 cp ~/wgs_assembly/results/short_long_reads_assembly/MTB_pacbio_canu_auto/MTB_pacbio_canu.contigs.fasta ~/wgs_assembly/results/short_long_reads_assembly/assemblies/MTB_pacbio_assembly.fasta
 cp ~/wgs_assembly/results/short_long_reads_assembly/MTB_illumina20X_spades/contigs.fasta ~/wgs_assembly/results/short_long_reads_assembly/assemblies/MTB_illumina20X_assembly.fasta
 cp ~/wgs_assembly/results/short_long_reads_assembly/MTB_illumina60X_spades/contigs.fasta ~/wgs_assembly/results/short_long_reads_assembly/assemblies/MTB_illumina60X_assembly.fasta

