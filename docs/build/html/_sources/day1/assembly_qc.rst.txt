Assessing the quality of the assemblies
=======================================

We will now start assessing the quality of our assembly. Overall, we want an assembly that is in as few contigs as possible, as complete as possible (with respect to a reference), and as accurate as possible (ideally at single base pair resolution).  So, when evaluating an assembly, there are 3 factors to consider listed below. Usually, in any assembly strategy there are tradeoffs between these factors. 

* **Contiguity**: Long contigs are better than short contigs. Long contigs provide more information about the genomic structure (for instance gene order, synteny)
* **Completeness**: We want the assembly to represent as much of the true genome as possible. The assumed genome size in our case is ~4.4 Mb
* **Accuracy**: We want an assembly that has few large-scale misassemblies and consensus errors (mismatches or indels). The latter is especially critical for identification of nucleotide variants.
  
To evaluate assemblies, we will use ``quast`` (Quality Assessment Tool for Genome Assemblies). In this case, we have a high quality reference assembly we can use for groundtruthing. Typically, there will be a lot of short "leftover" contigs that contain repetitive or low-complexity sequence or low-quality reads that cannot be assembled. To avoid biasing our evaluation, we will only use contigs that are at least 500 bp long (default threshold in ``quast``).
You can ``quast`` as follows on the 3 assemblies:

**DO NOT RUN**. We will use precomputed results.
::

 # define a variable for the reference genome
 REF_GENOME=/home/mtb_upm/mtb_genomics_workshop_data/genome-assembly-annotation/data/NC_000962.3.fna
 quast.py -R $REF_GENOME -o $MYRESULTSDIR/short_long_reads_assembly/assemblies/quast_3assemblies_qc \
 $MYRESULTSDIR/short_long_reads_assembly/assemblies/MTB_illumina20X_assembly.fasta \
 $MYRESULTSDIR/short_long_reads_assembly/assemblies/MTB_illumina60X_assembly.fasta \
 $MYRESULTSDIR/short_long_reads_assembly/assemblies/MTB_pacbio_assembly.fasta

Copy the precomputed results:
::

 cp -r ~/mtb_genomics_workshop_data/genome-assembly-annotation/precalculated/assemblies/quast_3assemblies_qc/ $MYRESULTSDIR/short_long_reads_assembly/assemblies/

In ``quast_3assemblies_qc`` directory, open ``report.html`` file in a browser.

::
 firefox $MYRESULTSDIR/short_long_reads_assembly/assemblies/quast_3assemblies_qc/report.html&

When you opened the results web page, put your green stickies on. We will discuss these results in detail.
