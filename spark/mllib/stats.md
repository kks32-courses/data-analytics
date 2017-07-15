## Statistics
Basic statistics are an important part of data analysis, both in ad hoc exploration and
understanding data for machine learning. MLlib offers several widely used statistic
functions that work directly on RDDs, through methods in the `mllib.stat.Statistics` class. Some commonly used ones include:

### `Statistics.colStats(rdd)`
Computes a statistical summary of an RDD of vectors, which stores the min,
max, mean, and variance for each column in the set of vectors. This can be used
to obtain a wide variety of statistics in one pass.

### `Statistics.corr(rdd, method)`
Computes the correlation matrix between columns in an RDD of vectors, using
either the Pearson or Spearman correlation ( method must be one of pearson and
spearman ).

### `Statistics.corr(rdd1, rdd2, method)`
Computes the correlation between two RDDs of floating-point values, using
either the Pearson or Spearman correlation ( method must be one of pearson and
spearman ).

### `Statistics.chiSqTest(rdd)`
Computes Pearson’s independence test for every feature with the label on an
RDD of LabeledPoint objects. Returns an array of ChiSqTestResult objects that
capture the p-value, test statistic, and degrees of freedom for each feature.
Label and feature values must be categorical (i.e., discrete values).

Apart from these methods, RDDs containing numeric data offer several basic
statistics such as `mean()`, `stdev()`, and `sum()`. In addition, RDDs support
`sample()` and `sampleByKey()` to build simple and stratified samples of data.

> [Jupyter notebook for stats](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/statistics-exercise.ipynb)

> [Solution: Jupyter notebook for stats](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/statistics.ipynb)
