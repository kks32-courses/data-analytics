# Machine learning

MLlib is Spark’s library of machine learning functions. Designed to run in
parallel on clusters, MLlib contains a variety of learning algorithms and is
accessible from all of Spark’s programming languages.

![Machine learning](ml.png)

> **warning** As of Spark 2.0, the RDD-based APIs in the spark.mllib package have entered maintenance mode. The primary Machine Learning API for Spark is now the DataFrame-based API in the spark.ml package.

## Data types

MLlib supports local vectors and matrices stored on a single machine, as well as distributed matrices backed by one or more RDDs. Local vectors and local matrices are simple data models that serve as public interfaces.

### Local vectors

A local vector has integer-typed and 0-based indices and double-typed values, stored on a single machine. MLlib supports two types of local vectors: dense and sparse. A dense vector is backed by a double array representing its entry values, while a sparse vector is backed by two parallel arrays: indices and values. For example, a vector (1.0, 0.0, 3.0) can be represented in dense format as [1.0, 0.0, 3.0] or in sparse format as (3, [0, 2], [1.0, 3.0]), where 3 is the size of the vector.

MLlib recognizes the following types as dense vectors:

* NumPy’s array
* Python’s list, e.g., [1, 2, 3]

and the following as sparse vectors:

* MLlib’s SparseVector.
* SciPy’s csc_matrix with a single column

> **Info** recommend using NumPy arrays over lists for efficiency, and using the factory methods implemented in Vectors to create sparse vectors.

```Python
import numpy as np
import scipy.sparse as sps
from pyspark.mllib.linalg import Vectors

# Use a NumPy array as a dense vector.
dv1 = np.array([1.0, 0.0, 3.0])
# Use a Python list as a dense vector.
dv2 = [1.0, 0.0, 3.0]
# Create a SparseVector.
sv1 = Vectors.sparse(3, [0, 2], [1.0, 3.0])
# Use a single-column SciPy csc_matrix as a sparse vector.
sv2 = sps.csc_matrix((np.array([1.0, 3.0]), np.array([0, 2]), np.array([0, 2])), shape = (3, 1))
```

## Statistics
Basic statistics are an important part of data analysis, both in ad hoc exploration and
understanding data for machine learning. MLlib offers several widely used statistic
functions that work directly on RDDs, through methods in the mllib.stat.Statis
tics class. Some commonly used ones include:

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

> [Jupyter notebook for stats](statistics.ipynb)

## Classification and Regression
Classification and regression are two common forms of supervised learning, where
algorithms attempt to predict a variable from features of objects using labeled train‐
ing data (i.e., examples where we know the answer). The difference between them is
the type of variable predicted: in classification, the variable is discrete (i.e., it takes on
a finite set of values called classes); for example, classes might be spam or nonspam for
emails, or the language in which the text is written. In regression, the variable predic‐
ted is continuous (e.g., the height of a person given her age and weight).

## Spam classification

As a quick example of MLlib, we show a very simple program for building a spam
classifier. This program uses two MLlib algorithms: `HashingTF` , which builds
term frequency feature vectors from text data, and
`Logistic RegressionWithLBFGS`, which implements the logistic regression
procedure using Limited-memory Broyden–Fletcher–Goldfarb–Shanno (BFGS)
algorithm. We assume that we start with two files, [spam.txt](spam.txt) and [normal.txt](normal.txt), each of which contains examples of spam and non-spam
emails, one per line. We then turn the text in each file into a feature vector
with TF, and train a logistic regression model to separate the two types of
messages.

```Python
# Load ML libraries
from pyspark.mllib.regression import LabeledPoint
from pyspark.mllib.feature import HashingTF
from pyspark.mllib.classification import LogisticRegressionWithLBFGS

# Create RDDs of Spam and Normal text
spam = sc.textFile("spam.txt")
normal = sc.textFile("normal.txt")

# Create a HashingTF instance to map email text to vectors of 10,000 features.
tf = HashingTF(numFeatures = 10000)

# Each email is split into words, and each word is mapped to one feature.
spamFeatures = spam.map(lambda email: tf.transform(email.split(" ")))
normalFeatures = normal.map(lambda email: tf.transform(email.split(" ")))

# Create LabeledPoint datasets for positive (spam) and negative (normal) examples.
positiveExamples = spamFeatures.map(lambda features: LabeledPoint(1, features))
negativeExamples = normalFeatures.map(lambda features: LabeledPoint(0, features))
trainingData = positiveExamples.union(negativeExamples)
trainingData.cache() # Cache since Logistic Regression is an iterative algorithm.

# Run Logistic Regression using the LBFGS algorithm.
model = LogisticRegressionWithLBFGS.train(trainingData)

# Test on a positive example (spam) and a negative one (normal). We first apply
# the same HashingTF feature transformation to get vectors, then apply the model.
posTest = tf.transform("I recently inherited wealth, send me money ...".split(" "))
negTest = tf.transform("Unfortunately, I won't be at the office ...".split(" "))
print ("Prediction for positive test example: %g" % model.predict(posTest))
print ("Prediction for negative test example: %g" % model.predict(negTest))
```
> [Jupyter notebook for spam](spam.ipynb)

> [Jupyter notebook for regression](regression.ipynb)

> [Jupyter notebook for NY taxi data](ny-taxi.ipynb)

> [Jupyter notebook for decision tree](trees.ipynb)
