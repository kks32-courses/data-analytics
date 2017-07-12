# Local vectors
A local vector is often used as a base type for RDDs in Spark MLlib. A local vector has integer-typed and 0-based indices and double-typed values, stored on a single machine. MLlib supports two types of local vectors: dense and sparse. A dense vector is backed by a double array representing its entry values, while a sparse vector is backed by two parallel arrays: indices and values.

For dense vectors, MLlib uses either Python lists or the NumPy array type. The later is recommended, so you can simply pass NumPy arrays around.

For sparse vectors, users can construct a SparseVector object from MLlib or pass SciPy scipy.sparse column vectors if SciPy is available in their environment. The easiest way to create sparse vectors is to use the factory methods implemented in Vectors.
