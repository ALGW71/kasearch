
---

<div align="center">    
 
# KA-Search: Rapid and exhaustive sequence identity search of known antibodies

---
    
by
Tobias H. Olsen $^{1,\dagger}$, Brennan A. Kenyon $^{1,\dagger}$, Iain H. Moal $^{2}$ and Charlotte M. Deane $^{1,3}$
    
$^{1}$ Oxford Protein Informatics Group, Department of Statistics, University of Oxford, Oxford, United Kingdom  
$^{2}$ GSK Medicines Research Centre, GlaxoSmithKline plc, Stevenage, United Kingdom  
$^{3}$ Exscientia plc, Oxford, United Kingdom  
$^{\dagger}$ These authors contributed equally to this work and share first authorship  
    
</div>


<!---<div style="text-align:center"><img src="data/tool_comparison.png" width="800"/></div> 

*Speed and sensitivity comparison between KA-Search and three commonly used protein sequence identity search tools. Speed was measured by the time to search 10 million sequence with a single query and sensitivity by how often the tools returned the closest or the closest within the top-100 match for 100 heavy chains against the same 10 million sequences.*
--->

## Abstract
Antibodies with similar amino acid sequences, especially across their complementary-determining regions, often share properties. Finding that an antibody of interest has a similar sequence to naturally expressed antibodies in healthy or diseased repertoires is a powerful approach for the prediction of antibody properties, such as immunogenicity or antigen specificity. However, as the number of available antibody sequences is now in the billions and continuing to grow, repertoire mining for similar sequences has become increasingly computationally expensive. Existing approaches are limited by either being low-throughput, non-exhaustive, not antibody-specific, or only searching against entire chain sequences. Therefore, there is a need for a specialized tool, optimized for a rapid and exhaustive search of any antibody region against all known antibodies, to better utilize the full breadth of available repertoire sequences.

We introduce Known Antibody Search (KA-Search), a tool that allows for rapid search of billions of antibody sequences by sequence identity across either the whole chain, the CDRs, or a user defined antibody region. We show KA-Search in operation on the ~2.4 billion antibody sequences available in the OAS database. KA-Search can be used to find the most similar sequences from OAS within 30 minutes using 5 CPUs. We give examples of how KA-Search can be used to obtain new insights about an antibody of interest. KA-Search is freely available at https://github.com/oxpig/kasearch.


-----------

# Software implementation

KA-Search is freely available and can be installed with pip.

~~~.sh
    pip install kasearch
~~~

or directly from github.

~~~.sh
    pip install -U git+https://github.com/oxpig/kasearch
~~~


**NB:** You need to manually install a version of [ANARCI](https://github.com/oxpig/ANARCI) in the same environment. ANARCI can also be installed using bioconda, however, this version is maintained by a third party.

~~~.sh
    conda install -c bioconda anarci
~~~

----------

# Download pre-aligned data to search against

This list contains the download links for the paper version of the pre-aligned OAS and any future releases, ready for KA-Search. 

**NB**: The following datasets are large, you should therefore ensure you have enough space before trying to download them.

- [OAS-aligned](http://opig.stats.ox.ac.uk/webapps/ngsdb/kasearch_aligned_oas/paper_aligned_oas_sep2022.tar) (Paper version), a pre-aligned version of OAS, from September 2022, with 2.4 billion sequences taking up **~63GB**. 
- [OAS-aligned-small](https://zenodo.org/record/7384311/files/oasdb_small.tar) (Paper version), a pre-aligned version of OAS, from September 2022, with 144 million sequences taking up **~4.4GB**. 
- [OAS-aligned-tiny](https://zenodo.org/record/7384311/files/oas-aligned-tiny.tar) (Paper version), a pre-aligned version of OAS, from September 2022, with 16 million sequences taking up **~400MB**. 


After downloading, extract the pre-aligned dataset with "tar -xf downloaded_file.tar". Give the extacted dataset path when initiating KA-Search to search against it. See how to do this by following the KA-Search notebook guide below.


---------

# KA-Search guide

KA-Search is designed to be downloaded and run locally. As a demo, we have set up a reduced version of KA-Search on a [Colab notebook](https://colab.research.google.com/github/TobiasHeOl/kasearch/blob/main/notebooks/KAsearch_colab.ipynb) that can be run remotely. KA-Search, as setup on the Colab, uses the OAS-aligned-tiny version of OAS to reduce the time and memory required to download the database. The Colab demo is composed of two parts:

- **Quick and easy use of KA-Search**: Here we allow the user to try out KA-Search with minimal configuration, simply paste your antibody variable domain sequence in and try it out!!

- **KA-Search with more configuration**: Here we expose the KA-Search API and go through a more in depth tutorial of how it can be set up for your particular usecase. We explain how to preprocess the query sequence, the possible search configurations, how to extract the metadata after finding the most identical sequences and how to preprocess your own database so it can be used with KA-Search.  

If the user want to follow this tutorial locally, we also provide [a Jupyter notebook](https://github.com/oxpig/kasearch/blob/main/notebooks/examples.ipynb) showcasing KA-Search. The content of the Jupyter notebook is the same as what is in the "KA-Search with more configuration" section of the Colab. By running it locally you can also search against the whole of OAS-aligned. 

---------

# Description of the returned results

The returned output from KA-Search consists of the all columns and metadata in the pre-aligned datasets searched as well as a column named "Identity", that contains the calculated sequence identity. The returned output is always sorted by highest identity.

For the OAS-aligned datasets these columns are;

- Each column from AIRR's rearrangement schema ([see here for exact description](https://docs.airr-community.org/en/stable/datarep/rearrangements.html)).
- Additional sequence specific information derived by OAS processing, i.e. nucleotides for the constant region if present, ANARCI numbering and ANARCI status. For more information see the [OAS paper](https://doi.org/10.1002/pro.4205).
- Metadata from the OAS data unit the sequence was derived from, i.e. author, species, experimental run and unique sequences in run. For more information see the [OAS help page](http://opig.stats.ox.ac.uk/webapps/oas/documentation).
- Lastly, the column "Identity", which contains the calculated sequence identity between the query and target sequence.

**NB:** Some returned columns contain NaNs. This is because those columns could not be populated when the data was originally processed, and is not a side-effect of KA-Search. 

---------

# Description of the main arguments and examples of different types of search

The main arguments for your search are;

- **database_path**: Path to the database to search. If not specified, the OAS-aligned-tiny (~400MB of 16m human heavy chain sequences) dataset will be downloaded and searched against. 
- **allowed_chain**: Which chain to search, either only heavy (Heavy), only light (Light) or any chain (Any)
- **allowed_species** Which species to search against (this depends on what species are in the used pre-aligned data). For OAS-aligned this includes, Human, Mouse, Camel and Humanized. 
- **regions**: Which specific region to search against. A list of regions to search, either the provided ones ('whole', 'cdrs' or 'cdr3'), or user-defined ones. An example of a user-defined one is \['111 ', '111A', '112A', '112 '\].
- **length_matched**: A list of false and true for whether to only compare sequences where the length of the region to search match. Example: [False, True, True]

**NB**: The length of regions list and length_matched list needs to be the same.


### 1. Example searching for whole region human sequences

In this example, we search for similar human heavy chains across the whole variable region, while also trying to find sequences which might differ in length.

~~~python

raw_queries = [
    'VKLLEQSGAEVKKPGASVKVSCKASGYSFTSYGLHWVRQAPGQRLEWMGWISAGTGNTKYSQKFRGRVTFTRDTSATTAYMGLSSLRPEDTAVYYCARDPYGGGKSEFDYWGQGTLVTVSS',
]

results = EasySearch(raw_queries, 
               allowed_chain='Heavy',           
               allowed_species='Human',   
               regions=['whole'],          
               length_matched=[False], 
              )
~~~

### 2. Example searching for the CDRH3 of any species, but only with exact CDRH3 length match.

In this example, we search for exact length CDRH3s from any species. If one is interested in finding sequences with CDR3 lengths that differ in length, the length_match argument should be set to False.

~~~python

raw_queries = [
    'VKLLEQSGAEVKKPGASVKVSCKASGYSFTSYGLHWVRQAPGQRLEWMGWISAGTGNTKYSQKFRGRVTFTRDTSATTAYMGLSSLRPEDTAVYYCARDPYGGGKSEFDYWGQGTLVTVSS',
]

results = EasySearch(raw_queries, 
               allowed_chain='Heavy',           
               allowed_species='Any',   
               regions=['cdr3'],          
               length_matched=[True], 
              )
~~~

---------


### Citation

```
@article{Olsen2022,
  title={KA-Search: Rapid and exhaustive sequence identity search of known antibodies},
  author={Tobias H. Olsen, Brennan A. Kenyon, Iain H. Moal and Charlotte M. Deane},
  journal={bioRxiv},
  doi={10.1101/2022.11.01.513855},
  year={2022}
}
```  
