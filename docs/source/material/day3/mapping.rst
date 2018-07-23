=====================================
Mapping Reads to the Reference Genome
=====================================
Now that we have high-quality reads, we will start *mapping* reads to our reference genome. That is, using a "reference" *Mycobacterium tuberculosis* genome, we will determine the likely genomic positions from which the short reads have most likely been sampled.

.. note:: The canonical reference genome sequence for *M. tuberculosis* is that of strain `H37Rv <https://www.nature.com/articles/31159>`_, the first *M. tuberculosis* genome to have been sequenced. As such, *H37Rv* has a high quality genomic sequence and a wealth of information accumulated since then. It is important to realize that the reason *H37Rv* genome is being used as a reference is rather practically grounded.


.. note:: *Read mapping*, *read aligment*, and *sequence alignment* are often used interchangeably but in fact they are different concepts. The purpose of sequence alignment is to identify the *homologous* positions between evolutionarily related sequences. With read mapping, our objective is to identify the genomic region(s) from which the read sequence might be coming from. A mapping is considered to be correct if it overlaps the true region. Once we identify these regions, then we want align the read, in other words find the detailed placement of each base in the read.

For mapping reads to the reference genome, we will use ``BWA`` suit of programs. In this workshop, we will only going to be using two programs within the suite: ``bwa index`` and ``bwa mem``.

``BWA`` (`Burrows Wheeler Aligner <https://academic.oup.com/bioinformatics/article/25/14/1754/225615>`_) uses Burrows Wheeler transform to rapidly map reads to the reference genome. Importantly, ``BWA`` allows for a certain number of mismatches as well as for insertions and deletions.

Before starting mapping, first we first need to index the reference genome. ``Indexing`` means arranging the genome into easily searchable chunks. Index the reference genome sequence using `bwa index`:
::

 bwa index $MYDATA/ref/NC_000962.3.fna

``bwa index`` will  output some files with a set of extensions (.amb, .ann, .bwt, .pac, .sa), which the main alignment program (``bwa mem``) knows the format of. The will all end up in the same directory as the reference fasta file. Indexing is done once for the reference sequence. Before starting mapping, you  need to make sure that these files have been generated. You don't need to know the information in each but here is the information each one stores:

* .amb: text, records appearance of N (or other non-ATGC) in the reference fasta
* .ann: text, to record ref sequences, name, length, etc.
* .bwt: binary, the Burrows-Wheeler transformed sequence
* .pac: binary, packaged sequence (four base pairs encode one byte)
* .sa: binary, suffix array index

Now, we will start mapping reads. Explore the mapping and alignment options that ``bwa mem`` has by typing:
::

 bwa mem

Read mapping has many  uses in bioinformatics pipelines. Depending on your goals, you might want to change the parameters of the mapper, which primarily determine the sensitivity and specificity of mapping. In this workshop, we will use the default options, which work well for short reads.
::

 bwa mem $MYDATA/ref/NC_000962.3.fna $MYRESULTS/qc/${sample}_qced_R1.fastq $MYRESULTS/qc/${sample}_qced_R2.fastq > $MYRESULTS/mapping/${sample}.mapped.sam

which aligns ``forward`` and ``reverse`` reads against the reference, outputs them to the ``stdout`` in ``.sam`` format, which we then save to a file.

Mappers result in either SAM or BAM file (http://genome.sph.umich.edu/wiki/SAM). Those are formats that contain the
alignment information, where BAM is the binary version of the plain text SAM format. Althouth we are outputting a ``.sam`` file here for illustrative purposes-so we can peek into it-, you will typically want to do generate ``.bam`` files for your mapping because ``.sam`` files can get quite big and there is really no need to save them.

Inspect the first few lines of the ``.sam`` output file with ``head`` or ``more``. It should look as follows:
::

 # @SQ SN:NC_000962.3  LN:4411532
 # @PG ID:bwa  PN:bwa  VN:0.7.17-r1188 CL:bwa mem ERS050945/data/ref/NC_000962.3.fna ERS050945/results1/qc/

Question: What does each line with sequences represent? Can you make sense of the information in different columns?