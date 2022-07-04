---
layout: page
title: pySummarizedExperiment
permalink: /pysummarizedexperiment
---

In the various -omics fields measurements are often done on assay of
genes x samples or compounds x aliquots (here, I will generalize by
using features x samples). However, data is often stored in a ‘long’
dataframe format where each combination of feature - sample -
measurement gets a single row. This is not an issue on a small scale,
but as I will demonstrate, this becomes problematic on large-scale
experiments. Before I dive into pySummarizedExperiment, Python version
of R’s SummarizedExperiment package, I will discuss the long dataframe
format first. An example is shown below.

<table>
<thead>
<tr class="header">
<th style="text-align: right;">Compound</th>
<th style="text-align: left;">Sample</th>
<th style="text-align: right;">Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">1</td>
<td style="text-align: left;">A</td>
<td style="text-align: right;">624.9124</td>
</tr>
<tr class="even">
<td style="text-align: right;">2</td>
<td style="text-align: left;">A</td>
<td style="text-align: right;">319.0420</td>
</tr>
<tr class="odd">
<td style="text-align: right;">3</td>
<td style="text-align: left;">A</td>
<td style="text-align: right;">454.4090</td>
</tr>
<tr class="even">
<td style="text-align: right;">1</td>
<td style="text-align: left;">B</td>
<td style="text-align: right;">393.0147</td>
</tr>
<tr class="odd">
<td style="text-align: right;">2</td>
<td style="text-align: left;">B</td>
<td style="text-align: right;">115.2793</td>
</tr>
<tr class="even">
<td style="text-align: right;">3</td>
<td style="text-align: left;">B</td>
<td style="text-align: right;">842.8201</td>
</tr>
<tr class="odd">
<td style="text-align: right;">1</td>
<td style="text-align: left;">C</td>
<td style="text-align: right;">566.1801</td>
</tr>
<tr class="even">
<td style="text-align: right;">2</td>
<td style="text-align: left;">C</td>
<td style="text-align: right;">847.3736</td>
</tr>
<tr class="odd">
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
<td style="text-align: right;">201.2670</td>
</tr>
</tbody>
</table>

This format has a couple of advantages. For example, it is easy to store
complex data in a single file, while simultaneous being human-readable.
However, it is not efficient as it will cause data duplication. With
increasing data sizes due to improved measurements and the rising field
of single-cell omics, the data duplication effect is quickly becoming
more important.

In a long dataframe, the amount of rows needed can be calculated by
multiplying the number of features x number of samples. For most -omics
studies, there are about 2.000 - 20.000 features with an increasing
amount of samples (often represented by patients). With 100 patients,
the number of rows needed is between 200.000 and 2 million. However,
with single cell omics on the rise, the number of samples can exceed
over 100.000. This would result in 200 million - 2 billion rows of data,
an enormous amount.

This is not the full story. For each feature, we often have metadata we
use to subset our data or that is used in algorithms. For example, for
each feature we might want to store identifiers for databases,
properties like the sequence (in case of transcripts) or molecular mass
(in case of proteomics / metabolomics). For samples we might want to
attach attributes like date of measurement, sample type, phenotype,
and/or other identifiers. In a long dataframe, each of these attributes
will have its own column, with features x samples datapoints. As I will
demonstrate, not only are these metadata the cause of major memory
usage, the number of metadata columns is actually the fold change limit
when comparing data storage between a long dataframe and a
pySummarizedExperiment.

There is another reason why the long format is not desired, in
particular for data analysis. Most statistical algorithms depend in some
way on a matrix format of rows x columns. The best example is a
Principle Component Analysis (PCA), which requires you to format the
data in matrices. It is not uncommon to encounter code that transforms
dataframes into matrices just to perform a PCA. Lastly, while a long
dataframe stores all data in a single representation, it consequently
combines data and metadata. This requires you to subset your dataframe
to obtain only data before applying algorithms.

Clearly there are several disadvantages of combining data into a single
dataframe, that become increasingly apparent with bigger studies.

## The pySummarizedExperiment object

<img src="assets/img/pySummarizedExperiment.png" width="400px" />

The pySummarizedExperiment object circumvents these issues by
restructuring data with constraints. These prevent the occurrence of
many-to-many relationships between columns. Simultaneously, this
separates metadata from actual data, making it possible to apply
algorithms without sub setting.

In fact, due to these constraints, the pySummarizedExperiment minimizes
the amount of memory any omics dataset will use. This is proven by
plotting the potential fold change in memory of a long dataframe and the
same data in a pySummarizedExperiment. We can see that this number
approaches the number of metadata columns in the samples + columns in
features. Let’s take a practical example from metabolomics. In a typical
untargeted metabolomics dataset we have about 100 samples and 10.000
compounds. However, for each sample, we save quite some metadata, namely
Datetime, type, calline, replicate, and batch. For each compound we
might save it’s MZmed, MZmin, MZmax, RTmed, Rtmin, RTmax, nPeaks, RSD,
and Blank effect. This could be extended with isotope, adduct and/or
fragment status, etc. However, with these columns we can save up to 14x
in comparison with a long dataframe. This means a 1.4GB dataset can be
reduced to just 100MB, which easily allows for machine learning
algorithms and other applications.

The core message of this post is to separate your metadata from your
data whenever possible. To still have all (meta)data available in a
single object, pySummarizedExperiment was developed.
