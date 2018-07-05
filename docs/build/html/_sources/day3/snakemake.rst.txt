##########
Day 1: Introduction
##########
Today we will give an introduction to sequencing techniques and data processing with linux.

Preprocessing & quality filtering using a Snakemake workflow
==========

In this part the following topics will be covered

- introduction to Snakemake
- unzipping files
- making FASTQC plots
- trimming reads with low quality and adapter removal
- combining all steps into a single workflow

Introduction to Snakemake
==========

https://speakerdeck.com/johanneskoester/workflow-management-with-snakemake

- Documentation for snakemake can be found here: https://bitbucket.org/johanneskoester/snakemake/wiki/Documentation
- Use this as a reference while following the step-by-step guide below

Launch Snakemake
==========

- Boot up the Ubuntu live USB-stick
- Open a terminal window using the icon on the left on the screen or by typing: ``ctrl + alt + T``
- Start the Snakemake environment::

    source /usr/local/bin/virtualenvwrapper.sh && workon snakemake
    
- Make sure the snakemake executable is available::

    snakemake -h
    

.. note::

   **Question**: Which version of Snakemake is installed?
   
   
Create a rule to unzip FASTQ files
==========
- Create a directory for this part for the workshop::

    mkdir qc_snakemake 
    cd qc_snakemake


In Snakemake, workflows are specified as Snakefiles. Inspired by GNU Make, a Snakefile contains rules, that denote how to create output files from input files. Dependencies between rules are handled implicitly, by matching filenames of input files against output files. Thereby wildcards can be used to write general rules.

- Create the main workflow file using your favorite text editor. For a graphical editor Gedit is recommended and for a command-line editor Nano is a good choice.::

    gedit Snakefile &

- Add the following lines::

    configfile: "config.json"

    rule all:
        input:
            expand("bunzip/{sample}.fastq",sample=config['data'])
            
    rule bunzip:
        input:
            lambda wildcards: expand("{basedir}{data}",basedir=config["basedir"],data=config["data"][wildcards.data])
        output:
            "bunzip/{data}.fastq"
        log:
            "bunzip/{data}.log"
        threads: 1
        shell: "bunzip2 -d -c {input} > {output}"


- Save the file. If you are using Gedit use the File menu or type ``ctrl + s`` or in Nano type ``ctrl + x`` and ``y`` 

In the above ``Snakefile`` first the configuration file is loaded, next a target rule is defined that will trigger the creation of the required output. Finally a rule is made to unzip FASTQ files compressed in bzip2 format. 

Since version 3.1, Snakemake directly supports the configuration of your workflow. A configuration is provided as a JSON file. The JSON file can be used to define a dictionary of configuration parameters and their values. In the workflow, the configuration is accessible via the global variable ``config``.

- Create a config file::

    gedit config.json

- Add the following configuration::

    {
        "basedir": "/home/nmp/Documents/DeHollander/",
        "samples": {
            "Sample1": ["Sample1"],
            "Sample2": ["Sample2"],
            "Sample3": ["Sample3"]
        },
        "data": {
            "Sample1": ["sample1.fastq.bz2"],
            "Sample2": ["sample2.fastq.bz2"],
            "Sample3": ["sample3.fastq.bz2"]
        },
        "adapters": "/home/nmp/Documents/DeHollander/contaminant_list.txt",
        "adapters_fasta": "/home/nmp/Documents/DeHollander/illumina_truseq_adapters.fa"
    }


- Now run snakemake. It will unpack the input files using the ``bunzip2`` command::

    snakemake
    
.. note::

   **Questions**:
    - Can you explain what happened? Which files are used for input? Which output files are created?
    - Open one of the fastq files either using the File Browser or using ``less`` on the command line
    
Creating quality plots using FASTQC
==========

- Add the following text rule to the Snakefile after the bunzip rule::

    rule fastqc_raw:
        input: fastq="bunzip/{sample}.fastq"
        output: "fastqc_raw/{sample}_fastqc/"
        params: dir="fastqc_raw", adapters=config['adapters']
        log: "fastqc_raw.log"
        threads: 1
        shell: "fastqc -q -t {threads} --contaminants {params.adapters} --outdir {params.dir} {input.fastq} > {params.dir}/{log}"

- Add final fastqc outputs to the target rule ``all``. Is should now look like this (note that a comma is added at the end of the bunzip line)::
    
    rule all:
        input:
            expand("bunzip/{sample}.fastq",sample=config['data']),
            expand("fastqc_raw/{sample}_fastqc/",sample=config['data'])

- Make the plots by running Snakemake again::

    snakemake

- Have a look at the results using a browser::

    firefox fastqc_raw/Sample*_fastqc/fastqc_report.html
    
.. note::

   **Questions**: 
    - What are the difference between the samples? 
    - Is there any difference in quality? 
    - Can you find adapter contamination?

Trim low quality reads and remove adapters
==========

- Create a rule for the fastq-mcf program::

    rule trim:
        input: fastq="bunzip/{sample}.fastq"
        output: fastq="fastq-mcf/{sample}_trimmed.fastq"
        params: dir="fastq-mcf", adapters=config["adapters_fasta"]
        log: "{sample}_fastq-mcf.log"
        threads: 1
        # -C: number of subsamples used for determining adapter parameters
        # http://scotthandley.wordpress.com/2013/09/25/adapter-removal-and-adaptive-quality-trimming-with-parity-using-fastq-mcf/
        shell: "fastq-mcf {params.adapters} {input.fastq} -o {output.fastq} -C 99999 -P 33 -w 4 -q 25 -u -x 0.01 -l 20 -t 0 -S >> {params.dir}/{log}"

- Add it to the Snakefile after the fastqc-raw rule
- Run snakemake now with a parameter showing the command that is going to be performed::

    snakemake -p

.. note::

   **Exercise**: 
    - Have a look at the fastq-mcf parameters. What is the minimum quality.
    - What do the other parameters mean?
    
Create quality plots of the trimmed data
==========

.. note::

   **Exercise**:
    - Create a rule for creating plots with FASTQC on the trimmed data. 
    - Use the fastqc_rule as as a template and adjust the input and output parameters.
    - Add the fastqc_trim output to the ``rule all:`` by adding another expand() line with the output of the fastqc_trim rule as input.
    - Run snakemake
    - Open the output files in Firefox and compare them to the raw data. What is the effect of trimming the reads?


Create quality plots using R
=========

Is it possible to run R code directly from a Snakefile. To make similar quality plots as FASTQC in R we are going to make use of the 'qrqc' package: http://www.bioconductor.org/packages/release/bioc/html/qrqc.html

- First we need to install some required software on the USB. Copy paste the commands below to perform this installation::

    wget https://bitbucket.org/johanneskoester/snakemake/raw/ef669ba49226368fc71eb1975a89168dcb87d59a/snakemake/utils.py
    mv utils.py ~/.virtualenvs/snakemake/lib/python3.3/site-packages/snakemake/
    sudo apt-get -y install python3-dev
    pip install rpy2
    
    
- Create a rule that reads in the untrimmed and trimmed data and saves a quality plot to disk::

    from snakemake.utils import R

    rule qc_plot:
        input: untrimmed="bunzip/{sample}.fastq", trimmed="fastq-mcf/{sample}_trimmed.fastq"
        output: "qrqc/{sample}_qualplot.png"
        run:
            R("""
            library("qrqc")
        
            s1 <- readSeqFile("{input.untrimmed}")
            s2 <- readSeqFile("{input.trimmed}")
            q <- qualPlot(list(untrimmed = s1, trimmed = s2))
            ggsave("{output}", q)
            """)
    
- Add this rule again to the Snakefile.
- Adjust the ``rule all:`` to incude the output of the qc_plot rule.

.. note::

   **Exercise**: Extend above rule to the other functions of the qrqc package. Have a look at the documentation of the qrqc package to see what is possible. For example create a base frequency plot with the basePlot command.
   
Merge paired-end Illumina data
=========
Sequence that are produced by the Illumina HiSeq or MiSeq machines come in pairs of fastq files with each pair of reads. Files containing R1 in the filename are in the forward direction and with R2 in the reverse. The two read pairs can be merged into a single, longer sequence. Downstream analysis can benefit of the longer read length. Several tools excist to merge paired-end read, and in this section we will make use of FLASH (http://ccb.jhu.edu/software/FLASH/).


- First we need to setup a new directory to store the results::

    mkdir /home/nmp/mergepairs && cd /home/nmp/mergepairs

- Create a new config.json file with this content::

    {
        "basedir": "/home/nmp/Documents/DeHollander/",
        "samples": {
            "Sample4": ["Sample4"],
            "Sample5": ["Sample5"]
        },
        "data": {
            "Sample4": {"forward": "sample4_R1.fastq", "reverse": "sample4_R2.fastq"},
            "Sample5": {"forward": "sample5_1.fq", "reverse": "sample5_2.fq"}
        },
        "adapters_fasta": "/home/nmp/Documents/DeHollander/illumina_truseq_adapters.fa"
    }

- The first step again is to trim the data. Since in the previous section we have worked with single end data, we have to adjust the fastq-mcf rule::

    rule fastqmcf_paired:
        input:
            forward = lambda wildcards: config["basedir"] + config["data"][wildcards.data]['forward'],
            reverse = lambda wildcards: config["basedir"] + config["data"][wildcards.data]['reverse']
        output:
            forward="fastq-mcf/{data}_R1_trimmed.fastq",
            reverse="fastq-mcf/{data}_R2_trimmed.fastq"
        params: dir="fastq-mcf", adapters=config["adapters_fasta"]
        log: "{data}_fastq-mcf.log"
        threads: 1
        # -C: number of subsamples used for determining adapter parameters
        # http://scotthandley.wordpress.com/2013/09/25/adapter-removal-and-adaptive-quality-trimming-with-parity-using-fastq-mcf/
        shell: "fastq-mcf {params.adapters} {input} -o {output.forward} -o {output.reverse} -C 99999 -P 33 -w 4 -q 25 -u -x 0.01 -l 20 -t 0 -S >> {params.dir}/{log}"

- We need to get a list of samples from the config file. Add this line to the Snakefile after the config.json file is loaded::

    DATA_TO_SAMPLE = {
        data: sample for sample, datasets in config["samples"].items()
        for data in datasets}

- Now lets run Snakemake to create a trimmed file of the forward read from Sample4::

    snakemake fastq-mcf/Sample4_R1_trimmed.fastq
    
One of the strengths of Snakemake is that you can run it on multiple input files with a range of parameters automatically. We are going to run the FLASH program on Sample4 and Sample5 with a range of mismatch parameters. Therefore we need to create a MISMATCHES variable and a 'final' rule that will trigger the creation of all needed files. 

- Add these lines directly after the ``DATA_TO_SAMPLE`` variable::

    MISMATCHES = "0 0.1 0.5".split(' ')

    rule all:
        input:
            expand("flash/{data}_trimmed_{mismatch}.extendedFrags.fastq".split(),data=config["samples"],mismatch=MISMATCHES)

- Now add a rule to run FLASH on a range of mismatch values::

    rule flash_trimmed:
        input:
            forward="fastq-mcf/{data}_R1_trimmed.fastq",
            reverse="fastq-mcf/{data}_R2_trimmed.fastq"
        output:
            "flash/{data}_trimmed_{mismatch}.extendedFrags.fastq"
        params:
            sample=lambda wildcards: DATA_TO_SAMPLE[wildcards.data],
            prefix="{data}_trimmed_{mismatch}", dir="flash",minlength="5",maxlength="400",mismatch="{mismatch}"
        log:
            "{data}_trimmed_{mismatch}_mergepairs.log"
        shell: 
            "flash -m {params.minlength} -M {params.maxlength} {input} -x {params.mismatch} -o {params.prefix} -d {params.dir} > {params.dir}/{log}"


- To visualize the steps Snakemake is going to perform on the different datasets, we can create a dependency graph with the following command::

    $ snakemake --dag | dot | display

- Finally, let's  run Snakemake to create all of the files seen in the previous picture::

    snakemake
    
.. note::

   **Questions**:
    - Have a look at the fastq-mcf log file ``fastq-mcf/Sample4_fastq-mcf.log``. Do you see any adapter contamination? Do the same for Sample5.
    - How many reads are merged? Compare the FLASH log file for Sample4 and Sample5: ``flash/Sample4_trimmed_0_mergepairs.log``
    - What is the effect of the different mismatch values? 

The main purpose of merging read pairs is to get longer reads. Using the qrqc package in ``R`` we can compare the length of the sequences before and after merging.

- Create the following ``qrqc_seqlen`` rule at the end of Snakefile::

    from snakemake.utils import R

    rule qrqc_seqlen:
        input: untrimmed="fastq-mcf/{data}_R1_trimmed.fastq", trimmed="flash/{data}_trimmed_0.extendedFrags.fastq"
        output: "qrqc/{data}_lengthplot.png"
        run:
            R("""
            library("qrqc")
        
            s1 <- readSeqFile("{input.untrimmed}")
            s2 <- readSeqFile("{input.trimmed}")
            l <- seqlenPlot(list(untrimmed = s1, trimmed = s2))
            ggsave("{output}", l)
            """)

- Create the plot for Sample4 and have a look at it::
    
    snakemake qrqc/Sample4_lengthplot.png
    display qrqc/Sample4_lengthplot.png


.. note::

   **Optional exercise**:
    - Make a rule to merge data with FLASH using the untrimmed files
    - There are alternative programs to perform the trimming and merging. Create rules for Alientrimmer to trim the data and Usearch to merge data ``usearch -fastq_mergepairs``. Compare this with the output of Fastq-mcf and FLASH.