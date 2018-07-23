
# Microbial Genome Assembly using short and long reads

by [Ulas Karaoz](https://eesa.lbl.gov/profiles/ulas-karaoz/)

## Introduction

In this lab we will perform de novo genome assembly of a bacterial genome. You will be guided through the genome assembly starting with data quality control, through to building contigs and analysis of the results. At the end of the lab you will know:

1. How to perform basic quality checks on the input data
2. How to run a short read assembler on Illumina data
3. How to run a long read assembler on Pacific Biosciences or Oxford Nanopore data
4. How to improve the accuracy of a long read assembly using short reads
5. How to assess the quality of an assembly

## References
[sga preqc](https://arxiv.org/abs/1307.8026)

## Data Sets

MTB
For the purpose of this tutorial we are focusing only on Illumina sequencing which uses ’sequence by synthesis’ technology in a highly parallel fashion. Although Illumina high throughput sequencing provides highly accurate sequence data, several sequence artifacts, including base calling errors and small insertions/deletions, poor quality reads and primer/adapter contamination are quite common in the high throughput sequencing data. The primary errors are substitution errors. The error rates can vary from 0.5-2.0% with errors mainly rising in frequency at the 3’ ends of reads.

we will use the [sga preqc](https://academic.oup.com/bioinformatics/article/30/9/1228/237596/Exploring-genome-characteristics-and-sequence) program to explore our short read data set before starting the assembly. We have generated a report for two E. coli Illumina data sets - one at 15x coverage and one at 50x coverage. This will allow us to look at the effect of sequencing coverage on the results of our assembly.

## Data Preparation

First, lets create and move to a directory that we'll use to work on our assemblies:

```
mkdir -p ~/workspace/Module3-assembly
cd ~/workspace/Module3-assembly
```

For convenience, we'll make symbolic links to the data sets that we'll work with. Run this command from the terminal, which will find all of the sequencing data sets we provided (using the `ls` command) and symlink those files into your current working directory.


```
ls ~/CourseData/CG_data/Module3/ecoli* | xargs -i ln -s {}
```

If you run `ls` you should now be able to see three files of sequencing data.

## Data Quality Control

During the lecture, you learned about **FASTQ** format and **base quality scores** (also called **Q scores**). To remind, **FASTQ** is a text based format to store nucleotide sequences and their associated base quality scores. **base quality score** is the DNA sequencer's estimate of the quality of each base call.

It is always a good idea to get an idea about the sequence data quality for raw reads as they are outputted from the machine. One way to do this is through exploratory visualization of the quality scores and other metrics to get an idea about the quality of reads and the potential sources of bias that might be present in the dataset (i.e. left over sequencing adapters, PCR duplicates). Sometimes, there are also human errors (for instance, potential contamination or mismatched samples) you would want to be aware of right away. For instance, if you did generate whole genome shotgun reads, you want to make sure that the sequence complexity of the reads do reflect that.

To perform data quality control, we will use [fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/), a tool that processes reads and presents visual reports.

### Retrieving read data

Create a directory named ```wgs_assembly``` and move into that directory:

```
mkdir ~/wgs_assembly
cd ~/wgs_assembly
```
Inside that directory, create the following directories:
```
mkdir -p data/reads
mkdir -p results/qc
```
Copy read fastq files into your data directory:
```
cp ~/data/reads/illumina/MTB_illumina_*.fastq ~/wgs_assembly/data/reads/
```
```MTB_illumina_R1.fastq``` and ```MTB_illumina_R2.fastq``` contain **forward** and **reverse** reads respectively. Peak into the first few lines of ```.fastq``` files.
```
head -n 10 data/reads/MTB_illumina_R1.fastq
head -n 10 data/reads/MTB_illumina_R2.fastq
```
What can you tell about the headers? What is the **read length** for this run? Can you quickly count the number of reads (Hint: use ```grep``` and ```wc```)?

### Running ```fastqc```
We will now run ```fastqc```. For most tools used in practicals, you can get help by using ```-h``` or ```-help``` flag. Type:
```
fastqc -h
```
Run ```fastqc``` for **forward** reads saving the results under your results directory:
```
fastqc -o ~/wgs_assembly/results/qc ./MTB_illumina_R1.fastq 
```
Now run it again for **reverse** reads:
```
fastqc -o ~/wgs_assembly/results/qc ./MTB_illumina_R2.fastq 
```
Open ```fastqc``` report in the browser:
```
firefox ~/wgs_assembly/results/qc/MTB_illumina_R1_fastqc.html
firefox ~/wgs_assembly/results/qc/MTB_illumina_R1_fastqc.html
```
## Quality Trimming
```
sickle pe -t sanger -q 30 -l 100 -n 10 \
-f ~/wgs_assembly/data/reads/MTB_illumina_R1.fastq \
-r ~/wgs_assembly/data/reads/MTB_illumina_R2.fastq \
-o ~/wgs_assembly/results/qc/MTB_illumina_qced_R1.fastq \
-p ~/wgs_assembly/results/qc/MTB_illumina_qced_R2.fastq \
-s ~/wgs_assembly/results/qc/MTB_illumina_single.fastq 1>~/wgs_assembly/results/qc/MTB_illumina_sickle.log

more ~/wgs_assembly/results/qc/MTB_illumina_sickle.log
```
## Pre-assembly Quality Control
Before running ```sga preqc```, we need to **interleave** **forward** and **reverse** reads into a single file. First, move into ```qc``` directory:
```
cd ~/wgs_assembly/results/qc/
```
To interleave reads, we will use ```seqtk``` tool:
```
seqtk mergepe MTB_illumina_qced_R1.fastq MTB_illumina_qced_R2.fastq > MTB_illumina_qced_R1R2.fastq
```
Index the data using **sga index**:
```
sga index -t 4 -a ropebwt MTB_illumina_qced_R1R2.fastq
```
Run **sga preqc**:
```
sga preqc -t 4 MTB_illumina_qced_R1R2.fastq > MTB_illumina_qced_R1R2.preqc
```
Make the report:
```
sga-preqc-report.py MTB_illumina_qced_R1R2.preqc 
```

Open the PDF report and try to interpret the results. Was the genome size estimated correctly? What differences do you notice between the 15x (blue) and the 50x (red) datasets? Which dataset do you expect to be easier to assemble (hint: preqc will perform a simulated genome assembly to estimate the contig sizes you might get).


## *M. tuberculosis* Genome Assembly using Illumina short reads

Now we'll assemble the *M. tuberculosis* 50x Illumina data using the [spades](http://bioinf.spbau.ru/spades) assembler. Parameterizing a short read assembly can be tricky and tuning the parameters (for example the size of the k-mer used) is often quite time consuming. Thankfully, spades will automatically select values for its parameters, making it particularly easy to use. You can start spades with this command (it will take a few minutes to run):

```
spades.py -o ecoli-illumina-50-spades/ -t 4 --12 ecoli.illumina.50x.fastq
```

After the assembly completes, let's move the results to a new directory that we'll use to keep track of all of our assemblies:

```
mkdir -p assemblies
cp ecoli-illumina-50-spades/contigs.fasta assemblies/ecoli.illumina.50x.spades-contigs.fasta
```

We can now start assessing the quality of our assembly. We typically measure the quality of an assembly using three factors:

- Contiguity: Long contigs are better than short contigs as long contigs give more information about the structure of the genome (for example, the order of genes)
- Completeness: Most of the genome should be assembled into contigs with few regions missing from the assembly (remember for this exercise we are only assembling one megabase of the genome, not the entire genome)
- Accuracy: The assembly should have few large-scale *misassemblies* and *consensus errors* (mismatches or insertions/deletions)

We'll use `abyss-fac.pl` to calculate how contiguous our spades assembly is. Typically there will be a lot of short "leftover" contigs consisting of repetitive or low-complexity sequence, or reads with a very high error rate that could not be assembled. We don't want to include these in our statistics so we'll only use contigs that are at least 500bp in length (protip: piping tabular data into `column -t` will format the output so the columns nicely line up):

```
abyss-fac.pl -t 500 assemblies/ecoli.illumina.50x.spades-contigs.fasta | column -t
```

The N50 statistic is the most commonly used measure of assembly contiguity. An N50 of x means that 50% of the assembly is represented in contigs x bp or longer. What is the N50 of the spades assembly? How many contigs were produced?

## *M. tuberculosis* Genome Assembly using PacBio long reads

Now, we'll use long sequencing reads to assemble the E. coli genome. Long sequencing reads are better at resolving repeats and typically give much more contiguous assemblies. Long reads have a much higher error rate than short reads though, so we need to use a different assembly strategy. In this tutorial, we'll use [canu](https://github.com/marbl/canu) to assemble the 25X PacBio dataset. The canu assembly of the pacbio data should take about 15 minutes on your computer (maybe take a break while it is running).  Run this command to generate the assembly:

```
/opt/canu-1.7.1/Linux-amd64/bin/canu gnuplotTested=true -p SRR3667790_pass -d SRR3667790_pass genomeSize=4m -pacbio-raw SRR3667790_pass.fastq
```

When it completes, copy the Pacbio assembly to our results directory:

```
# Copy the pacbio assembly from canu's directory
cp ecoli-pacbio-auto/ecoli-pacbio-canu.contigs.fasta assemblies/ecoli.pacbio.25x.canu-contigs.fasta
```


## Assessing the quality of your assemblies using a reference

The accuracy of the genome assembly is determined by how many misassemblies (large-scale rearrangements) and consensus errors (mismatches, insertions or deletions) the assembler makes. Calculating the accuracy of an assembly typically requires the use of a reference genome. We will use the [QUAST](http://quast.bioinf.spbau.ru/) software package to assess the accuracy of the assemblies.

Run QUAST on your three E. coli assemblies by running this command:

```
quast.py -R ~/CourseData/CG_data/Module3/references/ecoli_k12.fasta assemblies/*.fasta
```

Using the web browser for your instance, open the QUAST PDF report and try to determine which of the assemblies was a) the most complete b) the most contiguous and c) the most accurate.

Make	dot	plots	to	compare	assembly	structure	using	Gepard.

GHOME=/opt/Gepard
java -cp /opt/gepard-1.40.0/dist/Gepard-1.40.jar org.gepard.client.cmdline.CommandLine \
-seq1 /data2-raid/ukaraoz/Work/UPMWorkshop/data/pacbio/fastq/SRR3667790_pass/SRR3667790_pass.contigs.fasta \
-seq2 /data2-raid/ukaraoz/Work/UPMWorkshop/data/ref/NC_000962.3.fna \
-matrix /opt/gepard-1.40.0/src/matrices/edna.mat -lower 80 -maxwidth 1500 -maxheight 1500 -outfile gepard_out.png  

## Improving the Accuracy of the Long Read Assemblies

As PacBio and Nanopore reads have a higher error rate than Illumina reads it is expected that the long read assemblies has errors in the consensus sequence (mostly insertions and deletions). We can increase the accuracy of our consensus sequence by using more coverage (up to around 100x), or by "polishing" the assembly (inferring a better consensus sequence) with high-accuracy Illumina reads. We'll use [Pilon](https://github.com/broadinstitute/pilon) to calculate a new consensus sequence for our PacBio assembly. First we have to map the Illumina reads to our assembly using bwa.

```
bwa index assemblies/ecoli.pacbio.25x.canu-contigs.fasta
bwa mem -t 4 -p assemblies/ecoli.pacbio.25x.canu-contigs.fasta ecoli.illumina.50x.fastq | samtools sort -T tmp -o ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam -
samtools index ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam
```

Now we can run pilon:

```
java -Xmx4G -jar /usr/local/pilon/pilon_2.11-1.21-one-jar.jar --genome assemblies/ecoli.pacbio.25x.canu-contigs.fasta --frags ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam --output assemblies/ecoli.pacbio.25x.canu-contigs-pilon
```

Let's also polish the nanopore assembly:

```
bwa index assemblies/ecoli.nanopore.50x.canu-contigs.fasta
bwa mem -t 4 -p assemblies/ecoli.nanopore.50x.canu-contigs.fasta ecoli.illumina.50x.fastq | samtools sort -T tmp -o ecoli.illumina.50x.mapped_to_nanopore_contigs.sorted.bam -
samtools index ecoli.illumina.50x.mapped_to_nanopore_contigs.sorted.bam
java -Xmx4G -jar /usr/local/pilon/pilon_2.11-1.21-one-jar.jar --genome assemblies/ecoli.nanopore.50x.canu-contigs.fasta --frags ecoli.illumina.50x.mapped_to_nanopore_contigs.sorted.bam --output assemblies/ecoli.nanopore.50x.canu-contigs-pilon
```

After running pilon on the PacBio and nanopore assemblies, re-run the Quast step to generate a new report including all five assemblies. Did pilon-polishing help the accuracy of the long read assemblies?