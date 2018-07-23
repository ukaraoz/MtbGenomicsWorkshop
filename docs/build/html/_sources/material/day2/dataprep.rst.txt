Copying data and preparing the directories
==========================================

First let's create a directory named ```wgs_assembly``` for this practical. And then, under that directory create subdirectories to hold our data and results:
::
 mkdir ~/wgs_assembly
 mkdir -p ~/wgs_assembly/data
 mkdir -p ~/wgs_assembly/results
 mkdir -p ~/wgs_assembly/results/qc
 mkdir -p ~/wgs_assembly/results/short_long_reads_assembly

Now, copy the data into our data directory:
::
 cp -r /home/mtb_upm/mtb_genomics_workshop_data/genome-assembly-annotation/data/* ~/wgs_assembly/data

Define variables for convenience:
::
 MYDATADIR=~/wgs_assembly/data
 MYRESULTSDIR=~/wgs_assembly/results

