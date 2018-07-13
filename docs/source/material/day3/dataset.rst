========
Datasets
========

The dataset is a subset of 1000 *M. tuberculosis* isolates whole genome sequenced by Casali et al.: `Evolution and transmission of drug-resistant tuberculosis in a Russian population <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3939361/>`_. In the previous tutorial, we generated an initial set of variants from a single *M. tuberculosis* isolate. In this subsequent tutorial, we will be repeating that for additinal isolates. We will then combine all of these variants into a single dataset for further analysis.


The datasets are located under `/home/mtb_upm/mtb_genomics_workshop_data/mapping-variantcalling/data` directory, which has the following content:

+--------------------------+-----------------------------------------------------------------------------------------------+
| File                     | Content                                                                                       |
+==========================+===============================================================================================+
| Mtb_repeats.bed          | precalculated, coordinates of the repetitive regions in the reference *M.tuberculosis* genome |
+--------------------------+-----------------------------------------------------------------------------------------------+