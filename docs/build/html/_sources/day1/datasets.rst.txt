Data Sets
================================

The datasets we will use are sequencing reads from *M. tuberculosis* genome generated using Illumina and PacBio platforms.
For the Illumina platform, there are two sets of reads covering the same genome at 20X and 60X. We will also use a reference assembly for *M. tuberculosis H37Rv* (`NC_000962.3 <https://www.ncbi.nlm.nih.gov/nuccore/NC_000962.3>`_). H37Rv genome is 4411532 bp long.

The datasets are located under `home/mtb_upm` directory, which has the following content.
If you want to download the data set to your machine and run it locally, 
you can find the data `here <>`_.

+----------------------------------------------+-------------------------------------------------------------------------+
| File Content                                 |                                                                         |
+==============================================+=========================================================================+
| data/reads/illumina/MTB_illumina_R1.fastq    | 60X coverage run, Illumina short reads, forward                         |
+----------------------------------------------+-------------------------------------------------------------------------+
| data/reads/illumina/MTB_illumina_R2.fastq    | 60X coverage run, Illumina short reads, forward                         |
+----------------------------------------------+-------------------------------------------------------------------------+
| data/reads/illumina/MTB_illumina20X_R1.fastq | 20X coverage run, Illumina short reads, forward                         |
+----------------------------------------------+-------------------------------------------------------------------------+
| data/reads/illumina/MTB_illumina20X_R2.fastq | 20X coverage run, Illumina short reads, forward                         |
+----------------------------------------------+-------------------------------------------------------------------------+
| data/reads/pacbio/MTB_pacbio_R1.fastq        | PacBio long reads, forward                                              |
+----------------------------------------------+-------------------------------------------------------------------------+
| data/reads/pacbio/MTB_pacbio_R2.fastq        | PacBio long reads, reverse                                              |
+----------------------------------------------+-------------------------------------------------------------------------+
| data/ref/NC_000962.3.fna                     | Mycobacterium tuberculosis H37Rv reference assembly, 4411532 bp, linear |
+----------------------------------------------+-------------------------------------------------------------------------+
