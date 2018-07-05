Microbial Genome Annotation
===========================
Now that we have assembled the reads into contigs, we will annotate the genome. That means we will predict genes and assign functional annotation to them. *M. tuberculosis* is a very clonal organism and its genome is already well annotated so typically when we assemble a new *M. tuberculosis* isolate, there is no need to do ``de novo`` annotation. Although we are using *M. tuberculosis* as a working example here, the methodology is applicable to all microbial genomes.

A set of programs are available for genome annotation tasks. Here we will use ``Prokka``, a pipeline comprising several open source bioinformatic tools and databases. ``Prokka`` automates the process of locating open reading frames (ORFs) and RNA regions on contigs, translating ORFs to protein sequences, searching for protein homologs and producing standard output files. For gene finding and translation, ``Prokka`` uses a program named ``prodigal``. Homology searches are then performed using ``BLAST`` and ``HMMER`` based on the translated protein sequences as queries against a set of public databases (CDD, PFAM, TIGRFAM) as well as custom databases that are included with ``Prokka``.

Create a directory to hold the annotation results:
::

 mkdir -p ~/wgs_assembly/results/annotation/prokka
 cd ~/wgs_assembly/results/annotation/

Run ``Prokka`` (will take ~20 minutes for our assembly):
::
 prokka contigs.fa --outdir $SAMPLE --norrna --notrna --metagenome --cpus 8

``Prokka`` produces several types of output:

* a ``.gff`` file, which is a standardised, tab-delimited, format for genome annotations
* a Genbank (``.gbk``) file, which is a more detailed description of nucleotide sequences and the genes encoded in these.

When your dataset has been annotated you can view the annotations directly in the ``gff`` file. Peak into the ``gff`` file by doing:
::

 more -S PROKKA_11242015.gff


Question: How many coding regions were found by Prodigal? (Hint: use ``grep``)
Question: How many of the coding regions were given an enzyme identifier?
Question: How many were given a COG identifier?

Visualizing the annotations with Artemis
----------------------------------------
Artemis is a graphical Java program to browse annotated genomes.
Launch Artemis and load ``gff`` file:
::
 art&
 


