Copying data and preparing the directories
==========================================

First let's create a variable pointing to the data directory::

  DATA=/home/mtb_upm/data

Next, create a directory named ```wgs_assembly``` and move into that directory::

  mkdir ~/wgs_assembly
  cd ~/wgs_assembly

Inside that directory, create the following directory::

  mkdir -p ~/wgs_assembly/data

Copy the contents of the data directory into your data directory::
  
  cp -r $DATA/* ~/wgs_assembly/data/

Next, create the directories to hold your results::

  mkdir ~/wgs_assembly/results/qc
  mkdir ~/wgs_assembly/results/assemblies

