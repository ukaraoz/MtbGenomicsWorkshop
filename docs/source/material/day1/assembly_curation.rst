Improving the accuracy of the PacBio long read assembly
=======================================================

As PacBio and Nanopore reads have a higher error rate than Illumina reads it is expected that the long read assemblies has errors in the consensus sequence (mostly insertions and deletions). We can increase the accuracy of our consensus sequence by using more coverage (up to around 100x), or by “polishing” the assembly (inferring a better consensus sequence) with high-accuracy Illumina reads. We’ll use Pilon to calculate a new consensus sequence for our PacBio assembly. First we have to map the Illumina reads to our assembly using bwa.

bwa index assemblies/ecoli.pacbio.25x.canu-contigs.fasta
bwa mem -t 4 -p assemblies/ecoli.pacbio.25x.canu-contigs.fasta ecoli.illumina.50x.fastq | samtools sort -T tmp -o ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam -
samtools index ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam
Now we can run pilon:

java -Xmx4G -jar /usr/local/pilon/pilon_2.11-1.21-one-jar.jar --genome assemblies/ecoli.pacbio.25x.canu-contigs.fasta --frags ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam --output assemblies/ecoli.pacbio.25x.canu-contigs-pilon

