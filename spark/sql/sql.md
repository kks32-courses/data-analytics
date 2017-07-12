# Spark SQL

Spark SQL is a component added in Spark 1.0 that is quickly becoming Spark’s
preferred way to work with structured and semistructured data. By structured
data, we mean data that has a schema—that is, a consistent set of fields
acrossdata records. Spark SQL supports multiple structured data sources as
input, and because it understands their schema, it can efficiently read only
the fields you require from these data sources.

One use of Spark SQL is to execute SQL queries. Spark SQL can also be used to
read data from an existing Hive installation. For more on how to configure
this feature, please refer to the Hive Tables section. When running SQL from
within another programming language the results will be returned as a
Dataset/DataFrame. You can also interact with the SQL interface using the
command-line or over JDBC/ODBC.

# Apache Hive
One common structured data source on Hadoop is Apache Hive. Hive can store
tables in a variety of formats, from plain text to column-oriented formats,
inside HDFS or other storage systems. Spark SQL can load any table supported
by Hive.


# Datasets and DataFrames
A Dataset is a distributed collection of data. Dataset is a new interface
added in Spark 1.6 that provides the benefits of RDDs (strong typing,
ability to use powerful lambda functions) with the benefits of Spark SQL’s
optimized execution engine. A Dataset can be constructed from JVM objects
and then manipulated using functional transformations (map, flatMap,
filter, etc.).

> **warning** Python does not have the support for the Dataset API.
But due to Python’s dynamic nature, many of the benefits of the Dataset
API are already available (i.e. you can access the field of a row by name
naturally `row.columnName`).

A DataFrame is a Dataset organized into named columns. It is conceptually
equivalent to a table in a relational database or a data frame in R/Python,
but with richer optimizations under the hood. DataFrames can be constructed
from a wide array of sources such as: structured data files, tables in Hive,
external databases, or existing RDDs.

# DataFrames and Spark SQL

A DataFrame is equivalent to a relational table in Spark SQL, and can be
created using various functions in SQLContext. Load a
[JSON file of people](people.json) as a dataframe:

```Python
df = spark.read.json("people.json")
```

Use `.show()` to display the content of the  data frame.

```Python
df.show()
```
| age| name|
| -- | -- |
|null|Michael|
|  30|   Andy|
|  19| Justin|

## Dataset Operations (aka DataFrame Operations)

### `.printSchema()`
Print the schema in a tree format

```Python
df.printSchema()
```

### `.select()`

Show only `name` column

```Python
df.select("name").show()
```

### `.select()` and `df['col-header']`

Select everybody, but increment the age by 1

```Python
df.select(df['name'], df['age'] + 1).show()
```

### `.filter()` and `df['col-header']`

Select people older than 21

```Python
df.filter(df['age'] > 21).show()
```

### `.cout()`

Count people by age

```Python
df.groupBy("age").count().show()
```

### `createOrReplaceTempView(name)`

Creates or replaces a local temporary view with this DataFrame. The lifetime of
this temporary table is tied to the SparkSession that was used to create this
DataFrame.

### Inferring the Schema using Reflection

Spark SQL can convert an RDD of Row objects to a DataFrame, inferring the
datatypes. Rows are constructed by passing a list of key/value pairs as `kwargs`
to the Row class. The keys of this list define the column names of the table,
and the types are inferred by sampling the whole dataset, similar to the
inference that is performed on JSON files.

Load [people.txt](people.txt) to blob store.

```Python
from pyspark.sql import Row

# Load a text file and convert each line to a Row.
lines = sc.textFile("people.txt")
parts = lines.map(lambda l: l.split(","))
people = parts.map(lambda p: Row(name=p[0], age=int(p[1])))

# Infer the schema, and register the DataFrame as a table.
schemaPeople = spark.createDataFrame(people)
schemaPeople.createOrReplaceTempView("people")

# SQL can be run over DataFrames that have been registered as a table.
teenagers = spark.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

# The results of SQL queries are Dataframe objects.
# rdd returns the content as an :class:`pyspark.RDD` of :class:`Row`.
teenNames = teenagers.rdd.map(lambda p: "Name: " + p.name).collect()
for name in teenNames:
    print(name)
```

> output: # Name: Justin

### Programmatically Specifying the Schema
When a dictionary of `kwargs` cannot be defined ahead of time (for example, the
structure of records is encoded in a string, or a text dataset will be parsed
and fields will be projected differently for different users), a DataFrame can
be created programmatically with three steps.

* Create an RDD of tuples or lists from the original RDD;
* Create the schema represented by a StructType matching the structure of
tuples or lists in the RDD created in the step 1.
* Apply the schema to the RDD via createDataFrame method provided by
SparkSession.

```Python3
# Import data types
from pyspark.sql.types import *

# Load a text file and convert each line to a Row.
lines = sc.textFile("people.txt")
parts = lines.map(lambda l: l.split(","))

# Each line is converted to a tuple.
people = parts.map(lambda p: (p[0], p[1].strip()))

# The schema is encoded in a string.
schemaString = "name age"

fields = [StructField(field_name, StringType(), True) for field_name in schemaString.split()]
schema = StructType(fields)

# Apply the schema to the RDD.
schemaPeople = spark.createDataFrame(people, schema)

# Creates a temporary view using the DataFrame
schemaPeople.createOrReplaceTempView("people")

# SQL can be run over DataFrames that have been registered as a table.
results = spark.sql("SELECT name FROM people")

results.show()
```

> output

| name |
| -- |
|Michael|
|   Andy|
| Justin|

[Full API of Spark SQL Dataframe](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame)

> [Spark SQL Jupyter notebook](sql.ipynb)

# Using Spark SQL in Applications
The most powerful way to use Spark SQL is inside a Spark application. This
gives us the power to easily load data and query it with SQL while
simultaneously combining it with `regular` program code in Python, Java, or
Scala.

To use Spark SQL this way, we construct a `HiveContext` (or `SQLContext`) based
on our SparkContext. This context provides additional functions for querying
and interacting with Spark SQL data. Using the `HiveContext`, we can build
`SchemaRDDs`, which represent our structure data, and operate on them with `SQL`
or with normal RDD operations like `map()`.

Load [sample twitter data](tweet.json)

To get started with Spark SQL we need to add a few imports to our program

```Python
# Import Spark SQL
from pyspark.sql import HiveContext, Row
# Or if you can't include the hive requirements
from pyspark.sql import SQLContext, Row
```

Once we’ve added our imports, we need to create a `HiveContext`, or a
`SQLContext` if we cannot bring in the Hive dependencies. Both of these classes
take a `SparkContext` to run on.

```Python
hiveCtx = HiveContext(sc)
```

To make a query against a table, we call the `sql()` method on the `HiveContext`
or `SQLContext`. The first thing we need to do is tell Spark SQL about some data
to query. In this case we will load some Twitter data from JSON, and give it a
name by registering it as a “temporary table” so we can query it with SQL.

```Python
input = hiveCtx.read.json("tweet.json")
input.registerTempTable("tweets")
topTweets = hiveCtx.sql("SELECT text, retweetCount FROM tweets ORDER BY retweetCount LIMIT 10")
print(topTweets.collect())
```

> Accessing the text column in the topTweets SchemaRDD in Python

```Python
topTweetText = topTweets.rdd.map(lambda row : row.text)
print(topTweetText.collect())
```

## User-Defined Functions
User-defined functions, or UDFs, allow you to register custom functions in Python
Spark SQL offers a built-in method to easily register UDFs by passing in a
function in your programming language.

```Python
# Make a UDF to tell us how long some text is
hiveCtx.registerFunction("strLenPython", lambda x: len(x), IntegerType())
lengthSchemaRDD = hiveCtx.sql("SELECT strLenPython(text) FROM tweets LIMIT 10")
print(lengthSchemaRDD.collect())
```

> [Spark Hive Jupyter notebook for tweets](hive-tweet.ipynb)


## Challenge
> [Jupyter notebook for SQL intrusion detection](sql-df.ipynb)
