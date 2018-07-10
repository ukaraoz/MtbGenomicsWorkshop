Mapping Reads
==================================================
Mappers result in a SAM or BAM file
(http://genome.sph.umich.edu/wiki/SAM). Those are formats that contain the
alignment information, where BAM is the binary version of the plain text SAM
format. Describe::

 # Create directories to hold results
 BASE=/data2-raid/ukaraoz/Work/UPMWorkshop/ERS050945
 DATA=$BASE/data
 RESULTS=$BASE/results1
 mkdir -p $RESULTS/fastqc; mkdir -p $RESULTS/log; mkdir -p $RESULTS/qc; mkdir -p $RESULTS/mapping

 # Define the sample
 sample=ERS050945

First, generate a ``fastqc`` report:
::

 fastqc $DATA/reads/${sample}_R1.fastq
 fastqc $DATA/reads/${sample}_R2.fastq

Inspect the read quality profile and pick quality trimming parameters.
Quality trim reads using ``sickle``:
::

 sickle pe -t sanger -q 30 -l 50 -n 5 \
 -f $DATA/reads/${sample}_R1.fastq -r $DATA/reads/${sample}_R2.fastq \
 -o $RESULTS/qc/${sample}_qced_R1.fastq -p $RESULTS/qc/${sample}_qced_R2.fastq \
 -s $RESULTS/qc/${sample}_single.fastq 1> $RESULTS/log/${sample}.sickle.log

Question: How many paired reads remain after quality trimming?

Mapping reads to the reference genome sequence
-------------------------------------------
Next, using a "reference" *Mycobacterium tuberculosis* genome, we will determine the likely genomic positions from which the short reads might have been sampled.


.. note:: The canonical reference genome sequence for *M. tuberculosis* is that of strain `H37Rv <https://www.nature.com/articles/31159>`_, the first *M. tuberculosis* genome to have been sequenced. As such, *H37Rv* has a high quality genomic sequence and a wealth of information accumulated since then. It is important to realize that the reason *H37Rv* genome is being used as a reference is rather practically grounded.


.. note:: *Read mapping*, *read aligment*, and *sequence alignment* are often used interchangeably but in fact they are different concepts. The purpose of sequence alignment is to identify the *homologous* positions between evolutionarily related sequences. With read mapping, our objective is to identify the genomic region(s) from which the read sequence might be coming from. A mapping is considered to be correct if it overlaps the true region. Once we identify these regions, then we want align the read, in other words find the detailed placement of each base in the read.

For mapping reads to the reference genome, we will use ``BWA`` suit of programs. In this workshop, we will only going to be using two programs within the suite: ``bwa index`` and ``bwa mem``.

``BWA`` (`Burrows Wheeler Aligner <https://academic.oup.com/bioinformatics/article/25/14/1754/225615>`_) uses Burrows Wheeler transform to rapidly map reads to the reference genome. Importantly, ``BWA`` allows for a certain number of mismatches as well as for insertions and deletions.
Before starting mapping, first we first need to index the reference genome. ``Indexing`` means arranging the genome into easily searchable chunks.
::
 bwa index $DATA/ref/NC_000962.3.fna

``bwa index`` will  output some files with set extensions, which the main alignment program (``bwa mem``) knows the format of.
Now, we will start aligning. Explore the mapping and alignment options that ``bwa mem`` has by typing:
::

 bwa mem


In this workshop, we will use the default options, which work well for short reads:
::

 bwa mem $DATA/ref/NC_000962.3.fna $RESULTS/qc/${sample}_qced_R1.fastq $RESULTS/qc/${sample}_qced_R2.fastq > $RESULTS/mapping/${sample}.mapped.sam

which aligns ``forward`` and ``reverse`` reads against the reference, outputs them to the ``stdout`` in ``.sam`` format, which we then save to a file.
Inspect the first few lines of the ``.sam`` output file with ``more``. It should look as follows:
::

 # @SQ SN:NC_000962.3  LN:4411532
 # @PG ID:bwa  PN:bwa  VN:0.7.17-r1188 CL:bwa mem ERS050945/data/ref/NC_000962.3.fna ERS050945/results1/qc/

What does each line with sequences represent? Can you make sense of the information in different columns?

While ``.sam`` 


Generating mapping statistics from the SAM/BAM file
-------------------------------------------
You can determine mapping statistics directly from the bam file. Use for
instance::
    
    # Mapped reads only
    samtools view -c -F 4 map/contigs_pair-smds.bam
     
    # Unmapped reads only
    samtools view -c -f 4 map/contigs_pair-smds.bam


Pilon
------
Headers
DP	Valid read depth; some reads may have been filtered">
TD	Total read depth including bad pairs">
PC	Physical coverage of valid inserts across locus">
BQ	Mean base quality at locus">
MQ	Mean read mapping quality at locus">
QD	Variant confidence/quality by depth">
BC	Count of As, Cs, Gs, Ts at locus">
QP	Percentage of As, Cs, Gs, Ts weighted by Q & MQ at locus">
IC	Number of reads with insertion here">
DC	Number of reads with deletion here">
XC	Number of reads clipped here">
AC	Allele count in genotypes, for each ALT allele, in the same order as listed">
AF	Fraction of evidence in support of alternate allele(s)">
SVTYPE	Type of structural variant">
SVLEN	Difference in length between REF and ALT alleles">
END	End position of the variant described in this record">
IMPRECISE	Imprecise change from local reassembly (ALT contains Ns)