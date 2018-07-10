********
Day 1
********

Microbial Genome Assembly and Annotation

In this lab we will perform de novo bacterial genome assembly. We will also annotate the assembled genome.
Although we will specifically be assembling a *Mycobacteria tuberculosis* genome, the methodology presented here applies to any bacterial genome.You will be guided through the genome assembly starting with data quality control, through to building contigs and analysis of the results. At the end of the lab you will know:

1. How to perform basic quality checks on the read data
2. How to run a short read assembler on Illumina short reads
3. How to run a long read assembler on Pacific Biosciences long reads
4. How to improve the accuracy of a long read assembly using short reads
5. How to assess the quality of an assembly
6. How to annotate a microbial genome

.. toctree::
   :maxdepth: 1

   datasets
   dataprep
   read_qc
   preassembly_qc
   assembly
   assembly_qc
   longassembly_polish
   genome_annotation
