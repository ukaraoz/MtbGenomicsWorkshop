----------------------------------------
QCing and filtering the read alignments
----------------------------------------
Before we can attempt variant calling, we need to make sure that the alignments that are resulted from mapping artefacts are identified. We also want to get an overall picture on the mapping quality.

^^^^^^^^^^^^^^^^^
Duplicate marking
^^^^^^^^^^^^^^^^^
During the lecture, you heard about library and optical duplicates, which if they exist will bias the variant calling. We will mark these duplicates using ``picard``.

First, let's sort the ``.bam`` file, this time using ``picard``. See the note in the previous section. This should give identical results to ``samtools sort``, we are simply doing it this way for pipelining consistency.

::

 java -Xmx16g -Djava.io.tmpdir=/tmp -jar /opt/picard.jar SortSam \
 I=$MYRESULTS/mapping/${sample}.mapped.bam O=$MYRESULTS/mapping/${sample}.sorted.bam SORT_ORDER=coordinate

Now, we can mark the duplicates:
::

 java -Xmx16g -Djava.io.tmpdir=/tmp -jar /opt/picard.jar MarkDuplicates \
 I=$MYRESULTS/mapping/${sample}.sorted.bam O=$MYRESULTS/mapping/${sample}.sorted.nodup.bam \
 REMOVE_DUPLICATES=true ASSUME_SORT_ORDER=coordinate M=$MYRESULTS/mapping/${sample}.sorted.dupmetrics

The metrics file will contain some statistics about the de-duplication. In the resulting ``bam`` file, only one fragment from each duplicate group survives unchanged, other duplicate fragments are given a FLAG ``0x400`` and will NOT be used downstream. Optimally, detection and marking of duplicate fragments should be done per library, i.e., over all read groups corresponding to a given library. In practice, often it is fine to take a shortcut and is sufficient to do it per lane (read group) to keep things naively *parallelizable*.

^^^^^^^^^^^^^^^^^^^^^^^^^^^
Assessing depth of coverage 
^^^^^^^^^^^^^^^^^^^^^^^^^^^
You might often want to take a peak at the depth of coverage using the read alignments. You can do this with ``samtools depth``:
::

 samtools depth -a $MYRESULTS/mapping/${sample}.sorted.nodup.bam > $MYRESULTS/mapping/${sample}.coverage.txt

This will give you the depth of coverage for each position in the genome.

Question: How many lines should be in this file?
Question: Can you write an ``awk`` one-liner to quickly compute the average depth of average?


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Assessing mapping qualities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Next, we will generate summary statistics on mapping qualities. For this, we will use ``qualimap``.
``qualimap`` has a GUI but we don't want to use it here. To avoid launching the GUI, we do ``unset DISPLAY`` first.
::

 unset DISPLAY;
 qualimap bamqc -bam $MYRESULTS/mapping/${sample}.sorted.nodup.bam --java-mem-size=4g -outformat PDF -outdir $MYRESULTS/mapping/${sample}.bam.qualimap

The results will be in a text file named ${sample}.bam.qualimap/genome_results.txt. Later when we scale our analysis up to multiple isolates, we will parse out these files to get summary statistics for all of the isolates.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Indel realignment and filtering low quality alignments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We will use ``pilon`` to call our variants. This will give us a candidate list from a single sample, which we can further filter after we merge variants from multiple samples. ``pilon`` with its ``--variant`` option will perform *indel realignment*. We can also specify *depth* and mapping quality based filters. Run ``pilon``:

::

 java -Xmx16g -jar /opt/pilon-1.22.jar --genome $MYDATA/ref/NC_000962.3.fna --bam $MYRESULTS/mapping/${sample}.sorted.nodup.bam --output $MYRESULTS/mapping/${sample}_pilon --variant  --mindepth 10 --minmq 40 --minqual 20

The output will be a ``.vcf`` file and a ``.fasta`` file. ``.fasta`` file has the sequence from this reference based assembly, which we won't be using downstream.