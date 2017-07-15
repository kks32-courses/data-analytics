## Classification and Regression
Classification and regression are two common forms of supervised learning, where
algorithms attempt to predict a variable from features of objects using labeled train‐
ing data (i.e., examples where we know the answer). The difference between them is
the type of variable predicted: in classification, the variable is discrete (i.e., it takes on
a finite set of values called classes); for example, classes might be spam or nonspam for
emails, or the language in which the text is written. In regression, the variable predic‐
ted is continuous (e.g., the height of a person given her age and weight).

### Linear regression
Linear regression is one of the most common methods for regression, predicting the
output variable as a linear combination of the features. MLlib also supports L^1 and L^2
regularized regression, commonly known as Lasso and ridge regression.
The linear regression algorithms are available through the `mllib.regression.Line
arRegressionWithSGD` , `LassoWithSGD` , and `RidgeRegressionWithSGD` classes. These
follow a common naming pattern throughout MLlib, where problems involving mul‐
tiple algorithms have a “With” part in the class name to specify the algorithm used.

Here, SGD is Stochastic Gradient Descent.

These classes all have several parameters to tune the algorithm:
> `numIterations`: Number of iterations to run (default: 100 ).

> `stepSize`: Step size for gradient descent (default: 1.0 ).

> `intercept`: Whether to add an intercept or bias feature to the data—that is, another feature whose value is always 1 (default: false ).

> `regParam` Regularization parameter for Lasso and ridge (default: 1.0 ).
In Python, you use the class method `LinearRegressionWithSGD.train()`, to which you pass key/value parameters.

```Python
Linear regression in Python
from pyspark.mllib.regression import LabeledPoint
from pyspark.mllib.regression import LinearRegressionWithSGD
points = # (create RDD of LabeledPoint)
model = LinearRegressionWithSGD.train(points, iterations=200, intercept=True)
print ("weights: %s, intercept: %s" % (model.weights, model.intercept))
```

### Logistic regression
Logistic regression is a binary classification method that identifies a linear separating
plane between positive and negative examples. In MLlib, it takes LabeledPoint s with
label 0 or 1 and returns a LogisticRegressionModel that can predict new points.

The LogisticRegressionModel from these algorithms computes a score between 0
and 1 for each point, as returned by the logistic function. It then returns either 0 or 1
based on a threshold that can be set by the user: by default, if the score is at least 0.5, it
will return 1 . You can change this threshold via `setThreshold()`. You can also disable it altogether via `clearThreshold()`, in which case `predict()` will return the raw
scores. For balanced datasets with about the same number of positive and negative
examples, we recommend leaving the threshold at 0.5. For imbalanced datasets, you
can increase the threshold to drive down the number of false positives (i.e., increase
precision but decrease recall), or you can decrease the threshold to drive down the
number of false negatives.

> [Jupyter notebook for regression](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/regression-exercise.ipynb)

> [Solution: Jupyter notebook for regression](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/regression.ipynb)

### Spam classification

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
> [Jupyter notebook for spam](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/spam.ipynb)

### Predicting tips from NY taxi data

> [Jupyter notebook for NY taxi data](ny-taxi-exercise.ipynb) or [view notebook](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/ny-taxi-exercise.ipynb)


> [Solution: Jupyter notebook for NY taxi data](ny-taxi-exercise.ipynb)
or [view solution](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/ny-taxi.ipynb)

### Decision trees and random forests
Decision trees are a flexible model that can be used for both classification and regres‐
sion. They represent a tree of nodes, each of which makes a binary decision based on
a feature of the data (e.g., is a person’s age greater than 20?), and where the leaf nodes
in the tree contain a prediction (e.g., is the person likely to buy a product?). Decision
trees are attractive because the models are easy to inspect and because they support
both categorical and continuous features.

![Decision Tree](decision-tree.png)

In MLlib, you can train trees using the `mllib.tree.DecisionTree` class, through the
static methods trainClassifier() and trainRegressor() . Unlike in some of the
other algorithms, the Java and Scala APIs also use static methods instead of a Deci
sionTree object with setters. The training methods take the following parameters:

> `data`: RDD of LabeledPoint .

> `numClasses`: (classification only) Number of classes to use.

> `impurity`: Node impurity measure; can be gini or entropy for classification, and must be
variance for regression.

> `maxDepth`: Maximum depth of tree (default: 5 ).

> `maxBins`: Number of bins to split data into when building each node (suggested value: 32 ).

> `categoricalFeaturesInfo`: A map specifying which features are categorical, and how many categories they each have. For example, if feature 1 is a binary feature with labels 0 and 1, and feature 2 is a three-valued feature with values 0, 1, and 2, you would pass {1: 2,
2: 3} . Use an empty map if no features are categorical.

The online [MLlib documentation](https://spark.apache.org/docs/latest/mllib-decision-tree.html) contains a detailed explanation of the algorithm
used. The cost of the algorithm scales linearly with the number of training examples,
number of features, and maxBins . For large datasets, you may wish to lower maxBins
to train a model faster, though this will also decrease quality.

The `train()` methods return a `DecisionTreeModel` . You can use it to predict values
for a new feature vector or an RDD of vectors via predict() , or print the tree using
`toDebugString()`. This object is serializable, so you can save it using Java Serializa‐
tion and load it in another program.

RandomForest is available through `RandomForest.trainClassifier` and `trainRegressor`.
Apart from the pertree parameters just listed, RandomForest takes the following parameters:

> `numTrees`: How many trees to build. Increasing numTrees decreases the likelihood of over‐
fitting on training data.

> `featureSubsetStrategy`: Number of features to consider for splits at each node; can be auto (let the library select it), all , sqrt , log2 , or onethird ; larger values are more expensive.

> `seed`: Random-number seed to use.

Random forests return a `WeightedEnsembleModel` that contains several trees (in the
`weakHypotheses` field, weighted by `weakHypothesisWeights` ) and can `predict()` an
RDD or Vector . It also includes a toDebugString to print all the trees.

> [Jupyter notebook for decision tree](trees-exercise.ipynb) or [View notebook]( or [View notebook](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/trees-exercise.ipynb)

> [Solution: Jupyter notebook for decision tree](trees.ipynb) or [View notebook](https://nbviewer.jupyter.org/urls/raw.githubusercontent.com/kks32-courses/data-analytics/master/spark/mllib/trees.ipynb)
