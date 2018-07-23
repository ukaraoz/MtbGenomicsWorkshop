Data Sets
================================

The datasets we will use are sequencing reads from *M. tuberculosis* genome generated using Illumina and PacBio platforms.
For the Illumina platform, there are two sets of reads covering the same library at 20X and 60X coverage. We will also use a reference assembly for *M. tuberculosis H37Rv* (`NC_000962.3 <https://www.ncbi.nlm.nih.gov/nuccore/NC_000962.3>`_). H37Rv genome is 4411532 bp long.

The datasets are located under `/home/mtb_upm/mtb_genomics_workshop_data/genome-assembly-annotation/data` directory, which has the following content:

+----------------------------------------------------+-------------------------------------------------------------------------+
| File                                               | Content                                                                 |
+====================================================+=========================================================================+
| MTB_illumina60X_R1.fastq                           | 60X coverage run, Illumina short reads, forward                         |
+----------------------------------------------------+-------------------------------------------------------------------------+
| MTB_illumina60X_R2.fastq                           | 60X coverage run, Illumina short reads, forward                         |
+----------------------------------------------------+-------------------------------------------------------------------------+
| MTB_illumina20X_R1.fastq                           | 20X coverage run, Illumina short reads, forward                         |
+----------------------------------------------------+-------------------------------------------------------------------------+
| MTB_illumina20X_R2.fastq                           | 20X coverage run, Illumina short reads, forward                         |
+----------------------------------------------------+-------------------------------------------------------------------------+
| MTB_pacbio_circular-consensus-sequence-reads.fastq | PacBio long reads, circular consensus sequence                          |
+----------------------------------------------------+-------------------------------------------------------------------------+
| NC_000962.3.fna                                    | Mycobacterium tuberculosis H37Rv reference assembly, 4411532 bp, linear |
+----------------------------------------------------+-------------------------------------------------------------------------+
