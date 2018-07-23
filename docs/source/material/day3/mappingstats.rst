------------------------------------------------------------------
Inspecting, summarizing, and manipulating the read alignments
------------------------------------------------------------------
We will now inspect and further process the mapping output. For this, we will use ``samtools`` suite, which as its name suggests let us *massage* the mapping information.

In this section, we will also be playing with the read alignments to have a working understanding of how to get information from and manipulate them.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Converting ``sam`` to ``bam`` and back
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First, let's convert our ``.sam`` mapping output to its binary counterpart ``.bam``. The binary format is much easier for computers to process but very difficult for humans to read.

To convert ``.sam`` to ``.bam``, we use the ``samtools view`` command. We must specify that our input is in ``.sam`` format (by default it expects ``.bam``) using the *-S* option. We will also explicitely specify that we want the output to be ``.bam`` (by default it also produces ``bam``) with the -b option. ``samtools`` follows the UNIX convention of sending its output to the UNIX STDOUT, so we need to use a redirect operator (“>”) to create a BAM file from the output.

::

 samtools view -S -b $MYRESULTS/mapping/${sample}.mapped.sam > $MYRESULTS/mapping/${sample}.mapped.bam

If you need to convert back ``.bam`` to ``.sam`` and look at it on the fly without saving to disk, you would do:
::

 samtools view $MYRESULTS/mapping/${sample}.mapped.bam | head -n 10


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Getting mapping stats directly from ``.bam``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We can also determine mapping statistics directly from the ``bam`` file. Use for instance the following (which uses FLAG options, *-F* and *-f*, more on that later below):
::

 samtools view -c -F 4 $MYRESULTS/mapping/${sample}.mapped.bam
 samtools view -c -f 4 $MYRESULTS/mapping/${sample}.mapped.bam

What does these numbers outputted to STDOUT represent -invoke ``samtools``'s help to answer this-. Do things make sense and add up?

^^^^^^^^^^^^^^^^^^^^^^^^^
Sorting the read alignments
^^^^^^^^^^^^^^^^^^^^^^^^^
When you map and align the reads to the reference, the resulting read alignments  are in random order with respect to their position in the reference genome. In other words, the ``.bam`` file is in the order that the sequences occurred in the input ``.fastq``. Prove to yourself  that this is the case:
::

 samtools view $MYRESULTS/mapping/${sample}.mapped.bam | more


Doing anything useful downstream such as calling variants or visualizing the alignments requires that the ``.bam`` is further manipulated. It must be sorted such that the alignments occur in “genome order”. That is, ordered positionally based upon their alignment coordinates on each chromosome (we obviously have a single one here). Sort the ``.bam`` file:
::

 samtools sort $MYRESULTS/mapping/${sample}.mapped.bam -o $MYRESULTS/mapping/${sample}.sorted.bam

After sorting, check the order:
::

 samtools view $MYRESULTS/mapping/${sample}.sorted.bam | more

Question (easy!): What do you notice?

.. note:: Sorting can also be done using `picard <https://broadinstitute.github.io/picard/>`_. We will be using ``picard`` for removing duplicate alignments after sorting. When we build our variant calling pipeline, we will do the sorting with ``picard`` just to keep things consistent with downstream processing. The results should be identical.


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Indexing read alignments (i.e. indexing ``.bam``)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Indexing a genome sorted ``.bam`` file allows one to quickly extract alignments overlapping particular genomic regions. Indexing is also required by genome viewers (for instance IGV) so that the viewers can quickly display alignments in each genomic region to which you navigate.
This is essential for large genomes.

Index your ``.bam`` file:
::

 samtools index $MYRESULTS/mapping/${sample}.sorted.bam

This will create an additional index file -what's that file's extension?-

For instance, now that we have indexed the ``.bam`` file, we have the flexibility to extract the alignments from a defined genomic region.
We can, for example, extract alignments from the 150th kilobase of the genome.
::

 samtools view $MYRESULTS/mapping/${sample}.sorted.bam NC_000962.3:150000-151000

Question: How many alignments are within *NC_000962.3:150000-151000*?


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Detailed inspection of some alignments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Again, let's inspect the first 10 alignments in our ``.bam`` file in detail:
::

 samtools view $MYRESULTS/mapping/${sample}.sorted.bam | head -n 10

Let's also inspect just the header. The *header* in a ``bam`` file records important information regarding the reference genome to which the reads were aligned, as well as other information about how the BAM has been processed. We can ask the view command to report solely the header by using the -H option. As the downstream programs further process the alignments, they will typically add information about what they did to the header. For now, our header contain bare minimum information.
::

 samtools view -H $MYRESULTS/mapping/${sample}.sorted.bam

The FLAG field in the ``.bam`` format encodes several key pieces of information regarding how an alignment aligned to the reference genome. We can use this information to isolate specific types of alignments that we want to use in our analysis. Here is a table for the flags right from the manual:

+-------+--------------------------------------------------------+
| Bit   | Description                                            |
+=======+========================================================+
| 0x1   | template having multiple segments in sequencing        |
+-------+--------------------------------------------------------+
| 0x2   | each segment properly aligned according to the aligner |
+-------+--------------------------------------------------------+
| 0x4   | segment unmapped                                       |
+-------+--------------------------------------------------------+
| 0x8   | next segment in the template unmapped                  |
+-------+--------------------------------------------------------+
| 0x10  | SEQ being reverse complemented                         |
+-------+--------------------------------------------------------+
| 0x20  | SEQ of the next segment in the template being reversed |
+-------+--------------------------------------------------------+
| 0x40  | the first segment in the template                      |
+-------+--------------------------------------------------------+
| 0x80  | the last segment in the template                       |
+-------+--------------------------------------------------------+
| 0x100 | secondary alignment                                    |
+-------+--------------------------------------------------------+
| 0x200 | not passing quality controls                           |
+-------+--------------------------------------------------------+
| 0x400 | PCR or optical duplicate                               |
+-------+--------------------------------------------------------+
| 0x800 | supplementary alignment                                |
+-------+--------------------------------------------------------+


For example, we often want to call variants solely from paired-end sequences that aligned “properly” to the reference genome. Can you tell why?

To ask the view command to report solely “proper pairs” we use the *-f* option and ask for alignments where the second bit is true (proper pair is true).
::

 samtools view -f 0x2 $MYRESULTS/mapping/${sample}.sorted.bam

* Question: How many properly paired alignments are there? How does that number compare to the number you got above using ``-F``?
* Question: How many improperly paired alignments are there?
* Question: Using what you played with, how would you calculate the fragment size distribution?