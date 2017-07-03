# Apache Spark

[Apache Spark](https://spark.apache.org/) is an open-source parallel processing framework that supports in-memory processing to boost the performance of big-data analytic applications. It was originally developed at UC Berkeley in 2009. 

Spark is designed for data science and its abstraction makes data science easier. Spark’s ability to cache the dataset in memory greatly speeds up iterative data processing, making Spark an ideal processing engine for implementing algorithms that can interpret and learn from the data.

Apache Spark consists of Spark Core and a set of libraries. The core is the distributed execution engine and the Java, Scala, and Python APIs offer a platform for distributed Extract Transform Load (ETL) application development. Apache Spark has built-in modules for streaming, SQL, machine learning and graph processing. 

![Spark components](spark-components.png)
> Apache Spark components

## Spark Core
Spark Core is the underlying general execution engine that contains the basic functionality of Spark, including components for task scheduling, memory management, fault recovery, interacting with storage systems, and more. It provides In-Memory computing and referencing datasets in external storage systems. Spark Core also defines the API for the resilient distributed data-sets (`RDD`s), which represent a collection of items distributed across many compute nodes which can be manipulated in parallel. 


## Spark SQL
Spark SQL provides a new data abstraction called `SchemaRDD` for working with structured and semi-structured data. It allows querying data via `SQL` as well as the `Apache Hive` variant of SQL—called the Hive Query Language (`HQL`)—and it supports many sources of data, including Hive tables, Parquet,
and JSON.

## Spark Streaming
This component leverages Spark Core's fast scheduling capability to enable processing of live streams of data. It ingests data in mini-batches and performs RDD transformations on those mini-batches of data.

## Spark MLlib
Spark also includes MLlib, a library that provides a growing set of machine algorithms for common data science techniques: Classification, Regression, Collaborative Filtering, Clustering and Dimensionality Reduction.

Spark’s ML Pipeline API is a high level abstraction to model an entire data science workflow.   The ML pipeline package in Spark models a typical machine learning workflow and provides abstractions like Transformer, Estimator, Pipeline & Parameters.  This is an abstraction layer that makes data scientists more productive.

## GraphX
GraphX is a library for manipulating graphs (e.g., a social network graphs, city-scale road networks) 
and performing graph-parallel computations. GraphX extends the Spark RDD API to create a directed graph with arbitrary properties attached to each vertex and edge. GraphX also provides various oper‐
ators for manipulating graphs (e.g., `subgraph` and `mapVertices`) and a library of common graph algorithms (e.g., PageRank and triangle counting).
