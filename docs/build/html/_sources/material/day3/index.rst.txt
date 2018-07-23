================================================
Day 3: Read mapping and variant calling
================================================

In this lab we will map short reads against the reference *Mycobacteria tuberculosis* genome (H37Rv) and call variants based on the mapped reads. This pipeline is sometimes also called "reference based assembly" since you can use the called variants to generate a new genomic sequence.

At the end of the lab you should:

1. know how to index the reference genome and map short reads
2. be familiar with different file formats (``.bam``, ``.sam``, ``.vcf``)
3. how to query and manipulate ``.bam`` files
4. how to assess mapping quality
5. how to deduplicate the alignments and how to do indel realignment
6. how to filter by mapping quality
7. how to call variants


.. toctree::
   :maxdepth: 1

   dataset.rst
   dataprep_readqc.rst
   mapping
   igv
   mappingstats
   alignment_qc_filter
