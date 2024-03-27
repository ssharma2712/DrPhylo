# DrPhylo analysis for detecting fragile clades in an inferred phylogeny from phylogenomic datasets #

DrPhylo investigates major species relationships in phylogenies inferred from phylogenomic datasets. This approach applies evolutionary sparse learning (ESL) to build a genetic model for a clade of interest for a given set of sequence alignments from genes or genomic loci. DrPhylo outputs:
<br />


```
Gene species concordance (GSC)              : A numeric value that identifies gene-species combinations (sequence) harboring concordant ( GSC > 0) or conflicting (GSC < 0) phylogenetic signals. 

Sequence Classification Probability (SCP)   : Classification probability for a species for being a member of the clade of interest.  

Clade Proabaility (CP)                      : Probability (0-1) for the clade of interest. A low value for CP indicates the plausibility of being fragile.  

Model grid                                  : This is a grid representation of the clade model. The rows of this grid represent species with SCP and columns for genes. 
                                              Green       (GSC > 0 )
                                              Red         (GSC < 0 )
                                              White       (GSC = 0 )
                                              Cross mark  (Missing data)    

```
<br />
A schematic outline of DrPhylo analysis:
			<div style="display: flex; justify-content: center;">
			    <img src="https://github.com/ssharma2712/DrPhylo/assets/11808951/332cbd52-a1b3-4593-a0dd-62f6723376a0" width="600">
			</div>


## Installation ##

To run DrPhylo, you will need Python 3.8 or later installed, as well as the following Python libraries:
```R
numpy
biopython
matplotlib
pandas
```

You can install these libraries using pip:

`pip install biopython numpy pandas matplotlib`

To perform DrPhylo analysis, get the ESL package using the following on the command line:

	git clone -b grid_search https://github.com/kumarlabgit/ESL ESL
	cd ESL
	bash setup.sh


## Input ##
DrPhylo takes a phylogenetic tree in newick format and sequence alignments of genes or genomic loci in fasta format. The clade of interest in the newick tree needs to be marked by a node ID as presented in the example. An user can also provide a text file for the hypothesis 
 
<br />

```
Sequence alignmnets         : A list of sequence alignments into fasta format.  

Phylogenetic tree           : A phylogenetic tree with clade ID for the clade of interest.  

Phylogenetic hypothesis     : A text file containing the species name with corresponding response value (+1/-1). It is required for user-defined hypotheses.  
```
<br />

## Implementation ##

Once the setup is complete and all required Python packages are installed, one can perform DrPhylo analysis using the required inputs and other necessary optional arguments.

<br />

```
python3 ESL_pipeline.py tree_file.nwk alignment_list.txt  --optional arguments
```
<br />

#### Required arguments:

<br />

```

tree_file.nwk         : A phylogenetic tree in newick format with a node ID to construct a hypothesis for the clade of interest.
                        The hypothesis can also be specified with a separate file using the --response parameter.
                        It is highly recommended that the number of species in the clade be equal to or greater than those outside of the clade.
                        It is also recommended to use the smart sampling option (—-smart_sampling) when the number of species inside the clade is greater than the number outside the clade.  

alignment_list.txt    : A text file contains a list of paths for all sequence alignments. For example,
                        angiosperm_alns/7276_C12.fasta
                        angiosperm_alns/5111_C12.fasta
                        angiosperm_alns/5507_C12.fasta

```
<br />	

#### Optional argumnets:

<br />

```

--response             : Requires a text file containing a user-defined hypothesis. It has two columns, which are tab-separated. The first column contains species names, and the second column contains the response value for the 
                         species (+1/-1). A member species in the clade receives +1 and -1 otherwise. This hypothesis is unconstrained by the tree structure. It is highly recommended that the number of species within the clade of 
                         interest (+1) is equal to the number of species outside the clade. The hypothesis can also be specified using a separate text file provided using the --response parameter.  

--lambda1 (or -z)      : The site sparsity parameter that ranges from 0 to 1. When not specified, the default is 0.1. It is required for building a single-clade model.

--lambda2 (or -y)      : The gene sparsity parameter ranges from 0 to 1, and the default is 0.1 when not specified. It is required for building a single-clade model.

--grid_z               : This option builds multiple clade models for a sequence of site sparsity parameters defined by the user. A user can specify the minimum (0-1), maximum (0-1), and step size for the sequence of site sparsity 
                         parameters. This option must be used with --grid_y and does not require to specify --lambda1.  

--grid_y               : This option builds multiple clade models for a sequence of gene sparsity parameters defined by the user. A user can specify the minimum (0-1), maximum (0-1), and step size for the sequence of gene sparsity 
                         parameters. This option must be used with --grid_z and does not require to specify --lambda2. 

--grid_gene_threshold  : It is defined for early stopping for the grid search over the sparsity parameter space. It takes a value greater than zero (0) and builds models with less than or equal to this number of genes.

--smart_sampling       : This option is recommended when the user-defined hypothesis is not provided using "--resposne" option. Smart sampling ensures a class balance between the number of species inside and outside the clade of 
                         interest by phylogenetic-aware sampling of species outside of the clade.
--output               : The name of the output directory where all results from DrPhylo analysis will be stored. This directory is created automatically by the program. 
```
<br />	

## DrPhylo implementation using example dataset ##

After finishing the setup, change the dir

DrPhylo analysis for building clade models for two labeled clades, producing a result set for each of two hypotheses generated from the input tree:
<br />
<br />

```
python3 ESL_pipeline.py sample_files/ESL_test.nwk sample_files/angiosperm_100_sample_alns.txt --output sample_tree_output

```
<br />

DrPhylo analysis using a user-defined hypothesis, producing a single clade model for the given hypothesis:
<br />

```
python3 ESL_pipeline.py sample_files/ESL_test.nwk sample_files/angiosperm_100_sample_alns.txt --output sample_tree_output --response sample_files/test_pred.txt

```
<br />

DrPhylo analysis for building an ensemble clade model for a user-defined hypothesis using the grid search option. The site and gene sparsity score range from 0.05 to 0.1 and a step size of 0.05:
<br />

```
python3 ESL_pipeline.py sample_files/ESL_test.nwk sample_files/angiosperm_100_sample_alns.txt --grid_z 0.05,0.1,0.05 --grid_y 0.05,0.1,0.05 --output sample_grid2x2_output --response sample_files/test_pred.txt

```
<br />


#### Basic Results Files

`{internal_node_label}_{output_dir}_hypothesis_out_feature_weights.txt`: Tab-separated, two-column file of all non-zero feature weights generated by the sparse group LASSO.
	
`{internal_node_label}_{output_dir}_gene_predictions.txt`: Table of predictions produced by model, including the total weight contributed by each feature group to each sample's total score.

`{internal_node_label}_{output_dir}_gene_predictions.png`: Visual representation of *_gene_predictions.txt file.

`{internal_node_label}_{output_dir}_PSS.txt`: Position significance score for each position, calculated as the sum of absolute weights for each one-hot feature referring to that position. Only positions with non-zero significance are displayed. Also includes MIC values.

`{internal_node_label}_{output_dir}_GSS.txt`: Gene/group significance score for each non-zero gene/feature group, calculated as the sum of absolute weights for each feature in that gene.


#### Grid Search Results Files

In addition to containing versions of each of the basic output files with "_{lambda1}_{lambda2}" appended before the file extension (e.g. {internal_node_label}_{output_dir}_gene_predictions_{lambda1}_{lambda2}.txt) for each model generated, the output directory of a grid search process will contain the following results files aggregated across all non-filtered models produced by the grid search:

`{internal_node_label}_GCS_median.txt`: Median of each gene's contribution scores for each species (i.e. the values under the individual gene columns in the *_gene_predictions.txt file) across all non-filtered models.

`{internal_node_label}_GCS_median.png`: Visual representation of *_GCS_median.txt file.

`{internal_node_label}_PSS_median.txt`: Median of position significance scores across all non-filtered models.

`{internal_node_label}_GSS_median.txt`: Median of gene/group significance scores across all non-filtered models.


## Citation ##
If you use this software in your research, please cite our paper:

Sharma, S. & Kumar, S. (2024). Discovering fragile clades and causal sequences in phylogenomics by evolutionary sparse learning. (In review)
