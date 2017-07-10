# Key/value RDDs
Key/value RDDs are commonly used to perform aggregations, and often we will do
some initial ETL (extract, transform, and load) to get our data into a key/value
format. Key/value RDDs expose new operations (e.g., counting up reviews for each
product, grouping together data with the same key, and grouping together two
different RDDs).

Spark provides special operations on RDDs containing key/value pairs. These RDDs
are called `pair RDDs`. Pair RDDs are a useful building block in many programs, as
they expose operations that allow you to act on each key in parallel or regroup
data across the network. For example, pair RDDs have a `reduceByKey()` method that
can aggregate data separately for each key, and a `join()` method that can merge
two RDDs together by grouping elements with the same key. It is common to extract
fields from an RDD (representing, for instance, an event time, customer ID, or
other identifier) and use those fields as keys in pair RDD operations.

Running `map()` changes a regular RDD into a pair RDD:

```Python
# Create a pair of first word in a string and the string.
lines = sc.parallelize(["Hello World", "KBTU Cambridge"])
pairs = lines.map(lambda x: (x.split(" ")[0], x))
pairs.collect()
```
> output

> `[('Hello', 'Hello World'), ('KBTU', 'KBTU Cambridge')]`


### Transformations on one pair RDD
> PairRDD: {(1, 2), (3, 4), (3, 6)}

| Function name | Purpose | Example | Result |
| -- | -- | -- | -- |
|  `reduceByKey(func)` | Combine values with the same key. | `rdd.reduceByKey((x, y) => x + y)` | `{(1,2), (3,10)}` |
|  `groupByKey()` | Group values with the same key.| `rdd.groupByKey()` | `{(1, [2]), (3, [4, 6])}` |
|  `mapValues(func)` | Apply a function to each value of a pair RDD without changing the key. | `rdd.mapValues(x => x+1)` | `{(1, 3), (3, 5), (3, 7)}` |
|  `flatMapValues(func)` | Apply a function to each value of a pair RDD without changing the key. | `rdd.flatMapValues(x => (x to 5))` | `{(1, 2), (1, 3), (1, 4), (1, 5), (3, 4), (3, 5)}` |
|  `keys()` | Return an RDD of just the keys. | `rdd.keys()` | `{1, 3, 3}` |
|  `values()` | Return an RDD of just the values. | `rdd.values()` | `{2, 4, 6}` |
|  `sortByKey()` | Return an RDD sorted by the key. | `rdd.sortByKey()` | `{(1, 2), (3, 4), (3, 6)}` |


### Transformation of two-pair RDDs
> rdd = {(1, 2), (3, 4), (3, 6)} other = {(3, 9)}

![Two-pair RDDs transformations](two-pair-rdd-transformation.png)

> [Download Jupyter notebook of pair RDD transformations](key-value.ipynb)
