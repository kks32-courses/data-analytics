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

We can now evaluate the number of components in the RDD. This should show `100`.

```Python
# Evaluate number of components in `numbers`
numbers.count()
```
Try to get the first element in the RDD using `.first()` function call.

> **Warning** This approach of creating a RDD is very useful when you are learning Spark, since you can quickly create your own RDDs in the shell and perform operations on them. Keep in mind, however, that outside of prototyping and testing, this is not widely used since it requires that you have your entire dataset in memory on one machine.
