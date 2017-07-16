## Feature Extraction
The mllib.feature package contains several classes for common feature
transformations. These include algorithms to construct feature vectors from text
(or from other tokens), and ways to normalize and scale features.

### TF-IDF
Term Frequency–Inverse Document Frequency, or TF-IDF, is a simple way to
generate feature vectors from text documents (e.g., web pages). It computes two
statistics for each term in each document: the term frequency (TF), which is the
number of times the term occurs in that document, and the inverse document
frequency (IDF), which measures how (in)frequently a term occurs across the
whole document corpus. The product of these values, TF × IDF, shows how relevant
a term is to a specific document (i.e., if it is common in that document but
rare in the whole corpus).

MLlib has two algorithms that compute TF-IDF: `HashingTF` and `IDF`, both in the
`mllib.feature package`. HashingTF computes a term frequency vector of a given
size from a document. In order to map terms to vector indices, it uses a
technique known as the hashing trick. Within a language like English, there are
hundreds of thousands of words, so tracking a distinct mapping from each word to
an index in the vector would be expensive. Instead, HashingTF takes the hash
code of each word modulo a desired vector size, `S`, and thus maps each word
to a number between `0` and `S–1`. This always yields an S-dimensional vector,
and in practice is quite robust even if multiple words map to the same hash
code. The MLlib developers recommend setting `S` between 2^18 and 2^20.


HashingTF can run either on one document at a time or on a whole RDD. It
requires each `document` to be represented as an iterable sequence of
objects—for instance, a list in Python.

```Python
# Using HashingTF in Python
import numpy as np
import scipy.sparse as sps
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.feature import HashingTF

sentence = "hello hello world"
words = sentence.split() # Split sentence into a list of terms
tf = HashingTF(10000) # Create vectors of size S = 10,000
tf.transform(words)
Vectors.sparse(10000, {3065: 1.0, 6861: 2.0})
rdd = sc.wholeTextFiles(data).map(lambda name, text: removePunctuation(text).split())
# Transforms an entire RDD
tfVectors = tf.transform(rdd)
```
Once you have built term frequency vectors, you can use IDF to compute the
inverse document frequencies, and multiply them with the term frequencies to
compute the TF-IDF. You first call `fit()` on an IDF object to obtain an
IDFModel representing the inverse document frequencies in the corpus, then call
`transform()` on the model to transform TF vectors into IDF vectors.

```Python
# Using TF-IDF in Python
from pyspark.mllib.feature import HashingTF, IDF
# Read a set of text files as TF vectors
rdd = sc.wholeTextFiles(data).map(lambda name, text: removePunctuation(text).split())
tf = HashingTF()
tfVectors = tf.transform(rdd).cache()
# Compute the IDF, then the TF-IDF vectors
idf = IDF()
idfModel = idf.fit(tfVectors)
tfIdfVectors = idfModel.transform(tfVectors)
```
