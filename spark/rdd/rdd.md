# Resilient Distributed Dataset
The resilient distributed dataset (RDD) is an immutable, fault-tolerant distributed collection of objects that can be operated on in parallel. Each dataset in RDD is divided into logical partitions, which may be computed on different nodes of the cluster. RDDs can be created in two ways: by loading an external dataset, or by distributing a collection of objects (e.g., a list or set) in the driver program.

## Creating a RDD
The simplest way to create RDDs is to take an existing collection in your program
and pass it to SparkContext’s parallelize() method.

> **Info** Open the Jupyter iPython notebook created in the Azure Spark section.
You should see `SparkSession available as 'spark'.`, when you run your first command.

Now that Spark kernel is running and is connected via Jupyter notebooks. Let's go
ahead and create a list `num` with values ranging from 0 to 100. We can now create a RDD in memory using `SparkContext`

```Python
# Create a range of 0 - 100
num = range(100)
# Create a RDD in memory using Spark Context
numbers = sc.parallelize(num)
```

> **Warning** This approach of creating a RDD is very useful when you are learning Spark, since you can quickly create your own RDDs in the shell and perform operations on them. Keep in mind, however, that outside of prototyping and testing, this is not widely used since it requires that you have your entire dataset in memory on one machine.

## Operating on RDD
Once created, RDDs offer two types of operations: transformations and actions. Transformations are operations on RDDs that return a new RDD, such as `map()` and `filter()`. Actions are operations that return a result to the driver program or write it to storage, and kick off a computation, such as `count()` and `first()`. Spark treats transformations and actions very differently, so understanding which type of operation you are performing will be important. If you are ever confused whether a given function is a transformation or an action, you can look at its return type: transformations return RDDs, whereas actions return some other data type.

### Actions
Actions are the operations that return a final value to the driver program or write data to an external storage system. Actions force the evaluation of
the transformations required for the RDD they were called on, since they need to
actually produce output.

We can now evaluate the number of components in the RDD. This should show `100`.

```Python
# Evaluate number of components in `numbers`
numbers.count()
```
Try to get the first element in the RDD using `.first()` function call.

### Transformations

Transformations construct a new RDD from a previous one. Transformations and actions are different because of the way Spark computes RDDs. For example, one com‐
mon transformation is filtering data that matches a predicate.

We can use our numbers RDD to create a new RDD holding just even numbers in the list.

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
fashion—that is, the first time they are used in an action. This approach might seem unusual at first, but makes a lot of sense when you are working with Big Data.

Lazy evaluation means that when we call a transformation on an RDD (for instance, calling `map()`), the operation is not immediately performed. Instead, Spark internally records metadata to indicate that this operation has been requested. Rather than thinking of an RDD as containing specific data, it is best to think of each RDD as consisting of instructions on how to compute the data that we build up through transformations.

> **Info** Although transformations are lazy, you can force Spark to execute
them at any time by running an action, such as count() . This is an
easy way to test out just part of your program.
 
## Summary
To summarize, every Spark program and shell session will work as follows:
1. Create some input RDDs from external data.
2. Transform them to define new RDDs using transformations like filter() .
3. Ask Spark to persist() any intermediate RDDs that will need to be reused.
4. Launch actions such as count() and first() to kick off a parallel computation,
which is then optimized and executed by Spark.
