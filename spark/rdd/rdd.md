# Resilient Distributed Dataset
The resilient distributed dataset (RDD) is an immutable, fault-tolerant
distributed collection of objects that can be operated on in parallel. Each
dataset in RDD is divided into logical partitions, which may be computed on
different nodes of the cluster. RDDs can be created in two ways: by loading an
external dataset, or by distributing a collection of objects (e.g., a list or
set) in the driver program.

## Creating a RDD
The simplest way to create RDDs is to take an existing collection in your
program and pass it to SparkContext’s parallelize() method.

> **Info** Open the Jupyter iPython notebook created in the Azure Spark section.
You should see `SparkSession available as 'spark'.`, when you run your first
command.

Now that Spark kernel is running and is connected via Jupyter notebooks. Let's
go ahead and create a list `num` with values ranging from 0 to 100. We can now
create a RDD in memory using `SparkContext`

```Python
# Create a range of 0 - 100
num = range(100)
# Create a RDD in memory using Spark Context
numbers = sc.parallelize(num)
```

> **Warning** This approach of creating a RDD is very useful when you are
learning Spark, since you can quickly create your own RDDs in the shell and
perform operations on them. Keep in mind, however, that outside of prototyping
and testing, this is not widely used since it requires that you have your entire
dataset in memory on one machine.

## Operating on RDD
Once created, RDDs offer two types of operations: transformations and actions.
Transformations are operations on RDDs that return a new RDD, such as `map()`
and `filter()`. Actions are operations that return a result to the driver
program or write it to storage, and kick off a computation, such as `count()`
and `first()`. Spark treats transformations and actions very differently, so
understanding which type of operation you are performing will be important.
If you are ever confused whether a given function is a transformation or an
action, you can look at its return type: transformations return RDDs,
whereas actions return some other data type.

### Actions
Actions are the operations that return a final value to the driver program or
write data to an external storage system. Actions force the evaluation of
the transformations required for the RDD they were called on, since they need to
actually produce output.

We can now evaluate the number of components in the RDD. This should show `100`.

```Python
# Evaluate number of components in `numbers`
numbers.count()
```
Try to get the first element in the RDD using `.first()` function call.

### Transformations

Transformations construct a new RDD from a previous one. Transformations and
actions are different because of the way Spark computes RDDs. For example, one
common transformation is filtering data that matches a predicate.

We can use our numbers RDD to create a new RDD holding just even numbers in the
list.

```Python
# Filter and create a new RDD of even numbers between 0 and 100.
# This is a lazy evaluation and is not computed until required.
even = numbers.filter(lambda number: number%2 ==0)
```

See the type of variable `even`:

```Python
# See the type of `even`
print(even)
```

> output

```shell
PythonRDD[2] at RDD at PythonRDD.scala:48
```

#### Lazy evaluation
Although you can define new RDDs any time, Spark computes them only in a lazy
fashion—that is, the first time they are used in an action. This approach might
seem unusual at first, but makes a lot of sense when you are working with Big
Data.

We shall now look at an example of lazy evaluation. Typically, an RDD is
created by loading a file or data from an external database using
`SparkContext.textFile()`. In this example, we will load the
[2003 NY taxi data](http://www.andresmh.com/nyctaxitrips/) located in the
Azure Blob storage.

```Python
trips = sc.textFile("wasb://data@cdspsparksamples.blob.core.windows.net/NYCTaxi/KDD2016/trip_data_12.csv")
trips.first()
```

> **Info** Access Azure Blob storage using `wasb`

If Spark were to load and store all the lines in the file as soon as we wrote
`trips = sc.textFile(...)`, it would waste a lot of storage space. Instead,
once Spark sees the whole chain of transformations, it can compute just the
data needed for its result. In fact, for the `first()` action, Spark scans the
file only until it finds the first matching line; it doesn’t even read the
whole file.

Lazy evaluation means that when we call a transformation on an RDD (for
instance, calling `map()`), the operation is not immediately performed.
Instead, Spark internally records metadata to indicate that this operation
has been requested. Rather than thinking of an RDD as containing specific
data, it is best to think of each RDD as consisting of instructions on how to
compute the data that we build up through transformations.

> **Info** Although transformations are lazy, you can force Spark to execute
them at any time by running an action, such as count() . This is an
easy way to test out just part of your program.

#### Persisting RDDs
Spark’s RDDs are by default recomputed each time you run an action on
them. If you would like to reuse an RDD in multiple actions, you can ask Spark to
persist it using `.persist()`. After computing it the first time, Spark will
store the RDD contents in memory (partitioned across the machines in
your cluster), and reuse them in future actions. Persisting RDDs on disk
instead of memory is also possible. The behaviour of not persisting by default
may again seem unusual, but it makes a lot of sense for big datasets: if you
will not reuse the RDD, there’s no reason to waste storage space when Spark
could instead stream through the data once and just compute the result.

In practice, you will often use persist() to load a subset of your data into memory
and query it repeatedly. For example, if we knew that we wanted to compute multiple
results about a particular medallion `0BD7C8F5BA12B88E0B67BED28BEA73D8` in the
2003 NY taxi data:

```Python
medallion = trips.filter(lambda line: "0BD7C8F5BA12B88E0B67BED28BEA73D8" in line)
medallion.persist
medallion.first()
medallion.count()
```
If you attempt to cache too much data to fit in memory, Spark will automatically
evict old partitions using a Least Recently Used (LRU) cache policy. The RDDs
will be recomputed when required, and will not break a job due to too much data
in cache. The method `unpersist()` allows to manually remove them from the cache.

#### Unions
The `filter()` operation does not mutate the existing RDD . Instead, it returns
a pointer to an entirely new RDD. The `trips` RDD can still be reused later in the
program—for instance, to search for another medallion. Then, we’ll use another
transformation, `union()`, to print out the number of lines that contained
either medallion #1 or #2.

```Python
# Create another RDD for a different medallion and count the number of trips
medallion2 = trips.filter(lambda line: "D7D598CD99978BD012A87A76A7C891B7" in line)
medallion2.count()
```

Transformations `union()` is a bit different than `filter()`, as it operates on
two RDDs instead of one. Transformations can actually operate on any number of
input RDDs.

```Python
# Union operation combining data of medallion 1 and 2
medallions = medallion.union(medallion2)
medallions.count()
```
As we derive new RDDs from each other using transformations, Spark keeps
track of the set of dependencies between different RDDs, called the lineage graph. It
uses this information to compute each RDD on demand and to recover lost data if
part of a persistent RDD is lost.

### Passing Functions to Spark
Most of Spark’s transformations, and some of its actions, depend on passing in
functions that are used by Spark to compute data. For shorter functions `lambda`
can be used.

```Python
word = rdd.filter(lambda s: "error" in s)
def containsError(s):
  return "error" in s
word = rdd.filter(containsError)
```

> **Warning** Watch out for inadvertently serializing the object containing the function.

When you pass a function that is the member of an object, or contains references
to fields in an object (e.g., `self.field`), Spark sends the entire object to
worker nodes, which can be much larger than the bit of required information.

```Python
class SearchFunctions(object):
  def __init__(self, query):
    self.query = query
  def isMatch(self, s):
    return self.query in s
  def getMatchesFunctionReference(self, rdd):
    # Problem: references all of "self" in "self.isMatch"
    return rdd.filter(self.isMatch)
```

Instead, just extract the fields you need from your object into a local variable and pass
that in:

```Python
class WordFunctions(object):
  ...
  def getMatchesNoReference(self, rdd):
    # Safe: extract only the field we need into a local variable
    query = self.query
    return rdd.filter(lambda x: query in x)
```
## Summary
To summarize, every Spark program and shell session will work as follows:
1. Create some input RDDs from external data.
2. Transform them to define new RDDs using transformations like `filter()`.
3. Ask Spark to `persist()` any intermediate RDDs that will need to be reused.
4. Launch actions such as `count()` and `first()` to kick off a parallel
computation, which is then optimized and executed by Spark.

> [Download Jupyter notebook of RDDs](rdd.ipynb)
