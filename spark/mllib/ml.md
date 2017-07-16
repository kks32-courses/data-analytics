# Machine learning

MLlib is Spark’s library of machine learning functions. Designed to run in
parallel on clusters, MLlib contains a variety of learning algorithms and is
accessible from all of Spark’s programming languages.

![Machine learning](ml.png)

> **warning** As of Spark 2.0, the RDD-based APIs in the spark.mllib package have entered maintenance mode. The primary Machine Learning API for Spark is now the DataFrame-based API in the spark.ml package.

## Data types

MLlib supports local vectors and matrices stored on a single machine, as well as
distributed matrices backed by one or more RDDs. Local vectors and local matrices
are simple data models that serve as public interfaces.

### Local vectors

A local vector has integer-typed and 0-based indices and double-typed values, stored on a single machine. MLlib supports two types of local vectors: dense and sparse. A dense vector is backed by a double array representing its entry values, while a sparse vector is backed by two parallel arrays: indices and values. For example, a vector `(1.0, 0.0, 3.0)` can be represented in dense format as `[1.0, 0.0, 3.0]` or in sparse format as `(3, [0, 2], [1.0, 3.0])`, where `3` is the size of the vector.

MLlib recognizes the following types as dense vectors:

* NumPy’s array
* Python’s list, e.g., [1, 2, 3]

and the following as sparse vectors:

* MLlib’s SparseVector.
* SciPy’s csc_matrix with a single column

> **Info** recommend using NumPy arrays over lists for efficiency, and using
the factory methods implemented in Vectors to create sparse vectors.

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
sv2 = sps.csc_matrix((np.array([1.0, 3.0]), np.array([0, 2]), np.array([0, 2])), \
                      shape = (3, 1))
```

### Labeled point
A labeled point is a local vector, either dense or sparse, associated with a
label/response. In MLlib, labeled points are used in supervised learning
algorithms. We use a double to store a label, so we can use labeled points in
both regression and classification. For binary classification, a label should be
either 0 (negative) or 1 (positive). For multiclass classification, labels
should be class indices starting from zero: `0, 1, 2, ...`

```Python
from pyspark.mllib.linalg import SparseVector
from pyspark.mllib.regression import LabeledPoint

# Create a labeled point with a positive label and a dense feature vector.
pos = LabeledPoint(1.0, [1.0, 0.0, 3.0])

# Create a labeled point with a negative label and a sparse feature vector.
neg = LabeledPoint(0.0, SparseVector(3, [0, 2], [1.0, 3.0]))
```

#### Sparse data

It is very common in practice to have sparse training data. MLlib supports
reading training examples stored in `LIBSVM` format, which is the default format
used by `LIBSVM` and `LIBLINEAR`. It is a text format in which each line
represents a labeled sparse feature vector using the following format:

```
label index1:value1 index2:value2 ...
```
where the indices are one-based and in ascending order. After loading, the feature indices are converted to zero-based.
