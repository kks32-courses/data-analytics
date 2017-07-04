# Resilient Distributed Dataset
The resilient distributed dataset (RDD) is an immutable, fault-tolerant distributed collection of objects that can be operated on in parallel. Each dataset in RDD is divided into logical partitions, which may be computed on different nodes of the cluster. RDDs can be created in two ways: by loading an external dataset, or by distributing a collection of objects (e.g., a list or set) in the driver program.

## Creating a RDD
The simplest way to create RDDs is to take an existing collection in your program
and pass it to SparkContext’s parallelize() method.

> **Info** Open the Jupyter iPython notebook created in the Azure Spark section.
