---
layout: page
title: Single-cell analysis with Multiple Kernel Learning (scMKL)
description: Published in Nature Communication Biology
img: assets/img/scMKL_logo_small.png
importance: 1
category: work
related_publications: true
---

<h1 align="center">
<img src="https://github.com/ohsu-cedar-comp-hub/scMKL/blob/main/scMKL_logo.png?raw=true" width="500"/>
</h1><br>


![PyPI](https://img.shields.io/pypi/v/scmkl?label=pypi%20package)
[![PyPI Downloads](https://static.pepy.tech/personalized-badge/scmkl?period=total&units=INTERNATIONAL_SYSTEM&left_color=GREY&right_color=GREEN&left_text=downloads)](https://pepy.tech/projects/scmkl)
[![Anaconda-Server Badge](https://anaconda.org/ivango17/scmkl/badges/version.svg?style=flat&cache-control=no-cache)](https://anaconda.org/ivango17/scmkl)
[![Anaconda-Server Badge](https://anaconda.org/ivango17/scmkl/badges/downloads.svg?style=flat&cache-control=no-cache)](https://anaconda.org/ivango17/scmkl)
[![Anaconda-Server Badge](https://anaconda.org/ivango17/scmkl/badges/latest_release_date.svg?style=flat&cache-control=no-cache)](https://anaconda.org/ivango17/scmkl)


Single-cell Multiple Kernel Learning (scMKL) is a published supervised 
classification framework that is interpretable and capable of integrating 
across several several omics types. Leveraging Celer and AnnData objects, 
scMKL is scalable and has been packaged and documented for easy implementation 
by researchers everywhere!

ScMKL can:
- Train on a reference dataset and annotate cells based on phenotype (cell 
type, disease state, treatment, ect...)
- Annotate cells in an unsupervised manner where it trains on cluster IDs and 
feature grouping weights determine cluster phenotypes 
- Download and filter gene sets of interest with built-in functions
- Train on unimodal or multimodal data (currently supports RNA-seq, ATAC-seq, 
ADT-seq, and methylation data in any combination)
- Produce interpretable weight and performance metrics plots


## Installation

Installing scMKL is made easy with `conda` or `pip`.

To install with `conda`:
```bash
# Activate env with python >=3.11.1,<=3.13
conda activate scmkl_env

# Install the package
conda install -c conda-forge -c bioconda ivango17::scmkl
```

To install with `pip`:
```bash
# Activate env with python >=3.11.1,<=3.13
conda activate scmkl_env

# Install the package
pip install scmkl
```

## Set up

The table below shows what type of data scMKL expects based on the modality in 
question. In all situations, matrices should be cells x features.

| Modality | Input matrix description |
| -------- | ----------------- |
| Transcriptomics (RNA) | Counts matrix |
| Chromatin  Accessibility (ATAC) | Binarized counts matrix (any counts more than 0 become 1) |
| Epitope (ADT) | Counts matrix |
| Methylomics (MET) | Aggregated methylation over windows (e.g. mean methylation per genomics window) |

Additionally, scMKL requires a feature grouping dictionary where each key, 
value pair is represented as `'group1': ['feature1', 'feature2', 'feature7']`. 
Features can be genes for RNA, regions for ATAC and MET, or proteins for ADT. 
Any number of groups can be used and features can be overlapping between 
groups (e.g. `'feature1'` is in five groups).

Here, I want to show how to run scMKL with a scRNA-seq dataset. To do so, we: 
1. Read in gene names, cell labels, gene sets, and the data matrix
2. Create the scMKL-formatted AnnData object
3. Train and test scMKL
4. Plot the results

```python
import scmkl
import numpy as np

# Reading in required data
gene_names = np.load('PBMC_gene_names.npy')
cell_labels = np.load('PBMC_celltypes.npy')
gene_sets = np.load('Azimuth_2021.pkl', allow_pickle=True)
x = np.load('PBMC_RNA_counts.npy')

# Create scMKL-formatted AnnData object (automatically splits data)
adata = scmkl.create_adata(
    x, 
    feature_names=gene_names, 
    group_dict=gene_sets,
    allow_multiclass=True
    )

# Run scMKL
results = scmkl.one_v_rest(adata, names=['RNA'])

# Creating confusion matrix, weights heatmap, and performance across lambda
summary = scmkl.get_summary(multi)
scmkl.plot_metric(multi_summary)
scmkl.plot_conf_mat(results, title='', normalize=True)
scmkl.weights_heatmap(results, scale_weights=True)
```

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/plot_metric_multi.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/plot_conf_mat_multi.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/weights_heatmap_multi.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

