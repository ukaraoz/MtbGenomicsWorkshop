Improving the accuracy of the long read assembly
==================================================
In the lecture, we have seen that PacBio long reads have a significantly higher error rate compared to Illumina short reads. As a result, the long read assembly will have many errors in the consensus sequence (primarily indels). We can increase the accuracy of the consensus sequences using the Illumina reads, which will provide more coverage and higher quality bases. This process of improving the accuracy of the long reads using high fidelity short reads is sometimes referred to as *polishing*.

To polish PacBio assembly, we will use ``pilon`` which will generate a new consensus sequence for the assembly. The details of the following commands will be clear in the next lecture. For now, run the following commands:
::

 mkdir $MYRESULTSDIR/short_long_reads_assembly/polished_pacbio_assembly
 PACBIO_ASSEMBLY=$MYRESULTSDIR/short_long_reads_assembly/assemblies/MTB_pacbio_assembly.fasta
 cd $MYRESULTSDIR/short_long_reads_assembly/
 bwa index $PACBIO_ASSEMBLY
 bwa mem -t 4 -p $PACBIO_ASSEMBLY $MYRESULTSDIR/qc/MTB_illumina60X_qced_R1R2.fastq | samtools sort -T tmp -o $MYRESULTSDIR/short_long_reads_assembly/polished_pacbio_assembly/MTB_illumina60X_qced.mapped_to_pacbio_contigs.sorted.bam -
 samtools index $MYRESULTSDIR/short_long_reads_assembly/polished_pacbio_assembly/MTB_illumina60X_qced.mapped_to_pacbio_contigs.sorted.bam
 java -Xmx8G -jar /opt/pilon-1.22.jar --genome $PACBIO_ASSEMBLY --frags $MYRESULTSDIR/short_long_reads_assembly/polished_pacbio_assembly/MTB_illumina60X_qced.mapped_to_pacbio_contigs.sorted.bam --output $MYRESULTSDIR/short_long_reads_assembly/polished_pacbio_assembly/MTB_pacbio_assembly_shortreadcorrected

Do assembly qc with 4 assemblies. **DONOT RUN**, we will use precomputed results.
::

 quast.py -R ../NC_000962.3.fna -o quast_4assemblies_qc ./MTB_illumina20X_assembly.fasta ./MTB_illumina60X_assembly.fasta ./MTB_pacbio_assembly.fasta ./MTB_pacbio_assembly_shortreadcorrected.fasta

Copy precomputed results:
::
 cp -r ~/mtb_genomics_workshop_data/genome-assembly-annotation/precalculated/assemblies/quast_4assemblies_qc/ $MYRESULTSDIR/short_long_reads_assembly/assemblies/

In ``quast_4assemblies_qc`` directory, open ``report.html`` file in a browser. After you open, make sure you turn on the `extended report`.

::
 firefox $MYRESULTSDIR/short_long_reads_assembly/assemblies/quast_4assemblies_qc/report.html&

When you opened the results web page, put your green stickies on. We will discuss these results in detail.
