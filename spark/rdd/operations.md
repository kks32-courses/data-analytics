# Common Transformations and Actions
## Element-wise transformations

### `map()`
The map() transformation takes in a function and applies it to each element in
the RDD with the result of the function being the new value of each element in
the resulting RDD.

```Python
# Using map transformation
nums = sc.parallelize([1, 2, 3, 4])
squared = nums.map(lambda x: x * x).collect()
for num in squared:
    print(num)
```

> output

```shell
1
4
9
16
```

> **Info** It is useful to note that `map()`’s return type does not have to be
the same as its input type. For example, `map()` can operate on an RDD of
strings and return an RDD of doubles.

### `flatMap()`
Sometimes we want to produce multiple output elements for each input element.
The operation to do this is called `flatMap()`. Like `map()`, `flatMap()`
operates on each element, but returns an iterator with the return values.

```Python
# flatMap() in Python, splitting lines into words
lines = sc.parallelize(["hello world", "hi"])
words = lines.flatMap(lambda line: line.split(" "))
words.first() # returns "hello"
# Print flatmap() output
print(words.take(words.count()))
```
`flatMap()` “flattens” the iterators returned to it, so that instead of ending
up with an RDD of lists we have an RDD of the elements in those lists.

```shell
# flatMap() components
['hello', 'world', 'hi']
# map() components
[['hello', 'world'], ['hi']]
```

> Basic RDD transformations on an RDD containing {1, 2, 3, 3}

| Function name | Purpose | Example | Result |
| -- | -- | -- | -- |
|  `map()` | Apply a function to each element in the RDD and return an RDD of the result. | `rdd.map(x => x + 1)` | `{2, 3, 4, 4}` |
|  `flatMap()` | Apply a function to each element in the RDD and return an RDD of the contents of the iterators returned. Often used to extract words. | `rdd.flatMap(x => x.to(3))` | `{1, 2, 3, 2, 3, 3, 3}` |
|  `filter()` | Return an RDD consisting of elements that pass the condition passed to `filter()`. | `rdd.filter(x => x != 1)` | `{2, 3, 3}` |
|  `distinct()` | Remove duplicates. | `rdd.distinct()` | `{1, 2, 3}` |
| `sample(withReplacement, fraction, [seed])` | Sample an RDD, with or without replacement. | `rdd.sample(False, 0.5)` | Non-deterministic |

> Two-RDD transformations on RDDs containing {1, 2, 3} and {3, 4, 5}

| Function name | Purpose | Example | Result |
| -- | -- | -- | -- |
|  `union()` | Produce an RDD containing elements from both RDDs. | `rdd.union(other)` | `{1, 2, 3, 3, 4, 5}` |
|  `intersection()` | Produce an RDD containing only elements found in both RDDs. | `rdd.intersection(other|  `collect()` | Return all elements of RDD. | `rdd.collect()` | `{1, 2, 3, 3}` |
)` | `{3}` |
|  `subtract()` | Remove the contents of one RDD (e.g., remove training data). | `rdd.subtract(other)` | `{1, 2}` |
|  `cartesian()` | Cartesian product with the other RDD. | `rdd.cartesian(other)` | `{(1, 3), (1, 4), ... (3,5)}` |

## Actions
> Basic actions on an RDD containing {1, 2, 3, 3}

| Function name | Purpose | Example | Result |
| -- | -- | -- | -- |
|  `collect()` | Return all elements of RDD. | `rdd.collect()` | `{1, 2, 3, 3}` |
|  `count()` | Return the number of all elements of RDD. | `rdd.count()` | `4` |
|  `countByValue()` | Number of times each element occurs in the RDD. | `rdd.countByValue()` | `{(1, 1), (2, 1), (3, 2)}` |
|  `take(num)` | Return `num` elements from the RDD. | `rdd.take(2)` | `{1, 2}` |
|  `top(num)` | Return the top num elements from the RDD. | `rdd.top(2)` | `{3, 3}` |
|  `takeSample(withReplacement, num, [seed])` | Return num elements at random. | `rdd.takeSample(False, 1)` | Non-deterministic |
|  `reduce(func)` | Combine the elements of the RDD together in parallel (e.g., sum ). | `rdd.reduce((x, y) => x + y)` | `9` |
|  `fold(zero)(func)` | Same as `reduce()` but with the provided zero value. | `rdd.fold(0)((x, y) => x + y )` | `9` |
|  `aggregate(zeroValue)(seqOp, combOp)` | Similar to reduce() but used to return a different type. | `rdd.aggregate((0, 0)) ((x, y) => (x._1 + y, x._2 + 1), (x, y) => (x._1 + y._1, x._2 + y._2))` | `(9, 4)` |
|  `foreach(func)` | Apply the provided function to each element of the RDD. | `rdd.foreach(func)` | Nothing |

Refer to [Spark v2.1.0 docs](https://spark.apache.org/docs/2.1.0/api/python/pyspark.html)
for more RDD operators in pyspark.
