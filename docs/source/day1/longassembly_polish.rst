Improving the accuracy of the long read assembly
==================================================
In the lecture, we have seen that PacBio long reads have a significantly higher error rate compared to Illumina short reads. As a result, the long read assembly will have many errors in the consensus sequence (primarily indels). We can increase the accuracy of the consensus sequences using the Illumina reads, which will provide more coverage and higher quality bases. This process of improving the accuracy of the long reads using high fidelity short reads is sometimes referred to as *polishing*.

To polish PacBio assembly, we will use ``pilon`` which will generate a new consensus sequence for the assembly. The details of the following commands will be clear in the next lecture. For now, run the following commands:
::

 mkdir ~/wgs_assembly/results/short_long_reads_assembly/correct_pacbio_assembly
 PACBIO_ASSEMBLY=~/wgs_assembly/results/short_long_reads_assembly/assemblies/MTB_pacbio_assembly.fasta
 cd ~/wgs_assembly/results/short_long_reads_assembly
 bwa index $PACBIO_ASSEMBLY
 bwa mem -t 4 -p $PACBIO_ASSEMBLY ~/wgs_assembly/results/qc/MTB_illumina60X_qced_R1R2.fastq | samtools sort -T tmp -o ~/wgs_assembly/results/short_long_reads_assembly/correct_pacbio_assembly/MTB_illumina60X_qced.mapped_to_pacbio_contigs.sorted.bam -
 samtools index ~/wgs_assembly/results/short_long_reads_assembly/correct_pacbio_assembly/MTB_illumina60X_qced.mapped_to_pacbio_contigs.sorted.bam
 java -Xmx8G -jar /opt/pilon-1.22.jar --genome $PACBIO_ASSEMBLY --frags ~/wgs_assembly/results/short_long_reads_assembly/correct_pacbio_assembly/MTB_illumina60X_qced.mapped_to_pacbio_contigs.sorted.bam --output MTB_pacbio_assembly_shortreadcorrected

Rerun quast with 4 assemblies:
::

 quast.py -R ../NC_000962.3.fna -o quast_4assemblies_qc ./MTB_illumina20X_assembly.fasta ./MTB_illumina60X_assembly.fasta ./MTB_pacbio_assembly.fasta ./MTB_pacbio_assembly_shortreadcorrected.fasta