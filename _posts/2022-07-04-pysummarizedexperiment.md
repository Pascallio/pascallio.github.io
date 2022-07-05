---
layout: post
title: pySummarizedExperiment
author: "Pascal Maas"
categories: Python, omics
image: pySummarizedExperiment.png
---

In the various -omics fields measurements are often done on assay of
genes x samples or compounds x aliquots (here, I will generalize by
using features x samples). However, data is often stored in a ‘long’
dataframe format where each combination of feature - sample -
measurement gets a single row. This is not an issue on a small scale,
but as I will demonstrate, this becomes problematic on large-scale
experiments. Before I dive into pySummarizedExperiment, Python version
of R’s SummarizedExperiment package, I will discuss the long dataframe
format first. 

## Long dataframe

Having all your data in a single table has a couple of advantages. For example, it is easy to store
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

The pySummarizedExperiment object circumvents these issues by
restructuring data with constraints. These prevent the occurrence of
many-to-many relations between columns. Simultaneously, this
separates metadata from actual data, making it possible to apply
algorithms without sub setting. Let's take an example similar to the long dataframe and convert it into a pySummarizedExperiment. First, we need to create a dataframe:


```python
import pandas as pd
from random import randint
from random import seed


seed(42)
df = pd.DataFrame({
  "Feature": [1, 2, 3] * 3,
  "ID": ["1A", "2A", "3B"] * 3,
  "Sample": ["A"] * 3 + ["B"] * 3 + ["C"] * 3,
  "Date": ["2022-07-03"] * 3 + ["2022-07-04"] * 3 + ["2022-07-05"] * 3,
  "Value": [randint(0, 1000) for _ in range(9)]
})
display(df)
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Feature</th>
      <th>ID</th>
      <th>Sample</th>
      <th>Date</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1A</td>
      <td>A</td>
      <td>2022-07-03</td>
      <td>654</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2A</td>
      <td>A</td>
      <td>2022-07-03</td>
      <td>114</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>3B</td>
      <td>A</td>
      <td>2022-07-03</td>
      <td>25</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>1A</td>
      <td>B</td>
      <td>2022-07-04</td>
      <td>759</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2A</td>
      <td>B</td>
      <td>2022-07-04</td>
      <td>281</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3</td>
      <td>3B</td>
      <td>B</td>
      <td>2022-07-04</td>
      <td>250</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>1A</td>
      <td>C</td>
      <td>2022-07-05</td>
      <td>228</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2</td>
      <td>2A</td>
      <td>C</td>
      <td>2022-07-05</td>
      <td>142</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3</td>
      <td>3B</td>
      <td>C</td>
      <td>2022-07-05</td>
      <td>754</td>
    </tr>
  </tbody>
</table>
</div>


Next, we can create the object by supplying the dataframe and the names of the columns representing the rowIndex (Feature) and colIndex (Sample). By checking the data-relations between the columns, the `pySummarizedExperiment` class is able to detect in which slot columns belong: rowData, colData or Assay data. The column `ID` is rowData as its data relation is one-to-one with the `Feature` column. The same principle applies for the `Date` and `Sample` columns. The column `Value` gets correctly identified as Assay, since it's data relation is one-to-many for both `Feature` and `Sample` columns. These relations between columns form the only constraints for the `pySummarizedExperiment` object.


```python
from pySummarizedExperiment import pySummarizedExperiment
exp = pySummarizedExperiment(longDf = df, rowIndex = "Feature", colIndex = "Sample")
print(exp)
```

    class: pySummarizedExperiment
    dim: 3 3
    metadata(1): Datetime
    rownames(3): 1, 2, 3
    rowData names(1): ID
    colnames(3): A, B, C
    colData names(1): Date
    assays(1): Value
    

In fact, due to these constraints, the pySummarizedExperiment minimizes
the amount of memory any omics dataset will use. This is proven by
printing the fold change in memory of a long dataframe and the
same data in a pySummarizedExperiment. We can see that this number
approaches the number of metadata columns (colData + rowData columns):


```python
import sys
print(sys.getsizeof(df) / sys.getsizeof(exp))
```

    1.8764478764478765
    

To demonstrate the importance of this, let’s take a practical example from metabolomics. In a typical
untargeted metabolomics dataset we have about 100 samples and 10.000
compounds. However, for each sample, we save quite some metadata, e.g the time of measurement, 
sample type, replicate number and batch. For each compound we
might save its median-, minimum-, and maximum mass, median-, minimum- and maximum retention time, the number of peaks found, relative standard deviation,
and Background effect(s). This could be extended with even more metrics. However, with these columns we can save up to 13x
in comparison with a long dataframe. This means a 1.3GB dataset can be
reduced to just 100MB, which easily allows for machine learning algorithms and other applications.

Another advantage is the matrix format of the assay. This allows for each data interpretation without stacking or reshaping your data. For example, it is possible to perform data inspecting in a `pySummarizedExperiment` object by calling the `.boxplot()` method. This produces a boxplot for each column sample. We can also create a boxplot for each feature by setting the parameter `transpose = True`: 


```python
exp.boxplot()
```
    
![png](/assets/img/blog_10_1.png){ width=60% }

```python
exp.boxplot(transpose = True)
```

![png](/assets/img/blog_11_1.png){ width=60% }

Since a PCA is one of the most common transformations applied on -omics data, it is natively supported as well. First we autoscale the data and store it in the assay `scaled` by calling the method `.autoScale()`. We then use this assay, together with 2 components to calculate the transformation. The method `.pca()` returns a DataFrame that can be plotted by calling `.plot.scatter()` with the appropriate axes:


```python
exp["scaled"] = exp.autoScale()
exp.pca(assay="scaled", components = 2).plot.scatter(x = 0, y = 1)
```
    
![png](/assets/img/blog_13_1.png){ width=60% }
    
The core message of this post is to separate your metadata from your
data whenever possible. To still have all (meta)data available in a
single object, pySummarizedExperiment was developed. You can find it on my GitHub page [here](pascallio.github.com/pySummarizedExperiment) or in your terminal install via pip:

```{bash}
pip install pySummarizedExperiment
```
