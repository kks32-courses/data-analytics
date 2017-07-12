# Machine learning

MLlib is Spark’s library of machine learning functions. Designed to run in
parallel on clusters, MLlib contains a variety of learning algorithms and is
accessible from all of Spark’s programming languages.

![Machine learning](ml.png)

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
