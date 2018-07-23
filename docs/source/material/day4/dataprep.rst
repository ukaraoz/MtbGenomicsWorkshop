=====================
Data Prep and Read QC
=====================
Before starting, make sure that the following variables are defined in your shell environment. That means, you will need to type the following when you launch a new shell terminal:

::

 MYBASE=/home/mtb_upm/mapping-variantcalling
 MYDATA=$MYBASE/data
 MYRESULTS=$MYBASE/results

Create a separate directory for the ``.vcf`` files:
::
 mkdir $MYRESULTS/vcf

