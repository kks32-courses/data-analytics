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
|  `map()` | Apply a function to each element in the RDD and return an RDD of the result. | `rdd.map(lambda x: x + 1)` | `{2, 3, 4, 4}` |
|  `flatMap()` | Apply a function to each element in the RDD and return an RDD of the contents of the iterators returned. Often used to extract words. | `rdd.flatMap(lambda x: range(x, 4))` | `{1, 2, 3, 2, 3, 3, 3}` |
|  `filter()` | Return an RDD consisting of elements that pass the condition passed to `filter()`. | `rdd.filter(lambda x: x != 1)` | `{2, 3, 3}` |
|  `distinct()` | Remove duplicates. | `rdd.distinct()` | `{1, 2, 3}` |
| `sample(withReplacement, fraction, [seed])` | Sample an RDD, with or without replacement. | `rdd.sample(False, 0.5)` | Non-deterministic |

> Two-RDD transformations on RDDs containing {1, 2, 3} and {3, 4, 5}

| Function name | Purpose | Example | Result |
| -- | -- | -- | -- |
|  `union()` | Produce an RDD containing elements from both RDDs. | `rdd.union(other)` | `{1, 2, 3, 3, 4, 5}` |
|  `intersection()` | Produce an RDD containing only elements found in both RDDs. | `rdd.intersection(other)` | `{3}` |
|  `collect()` | Return all elements of RDD. | `rdd.collect()` | `{1, 2, 3, 3}` |
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
|  `reduce(func)` | Combine the elements of the RDD together in parallel (e.g., sum ). | `rdd.reduce(add)` | `9` |
|  `fold(zero)(func)` | Same as `reduce()` but with the provided zero value. | `rdd.fold(0)(add)` | `9` |
|  `aggregate((0, 0), seqOp, combOp)` | Similar to reduce() but used to return a different type. | `rdd.aggregate((0, 0)) ((x, y) => (x._1 + y, x._2 + 1), (x, y) => (x._1 + y._1, x._2 + y._2))` | `(9, 4)` |
|  `foreach(func)` | Apply the provided function to each element of the RDD. | `def f(x): print(x) rdd.foreach(f)` | Nothing |

Refer to [Spark v2.1.0 docs](https://spark.apache.org/docs/2.1.0/api/python/pyspark.html)
for more RDD operators in pyspark.

> [Download Jupyter notebook of RDD operations](rdd-operations.ipynb)

### aggregate

The aggregate action does not require the return type to be the same type as
the RDD. Like with fold, we supply an initial zero value of the type we want
to return. Then we provide two functions. The first one is used to combine the
elements from our RDD with the accumulator. The second function is needed to
merge two accumulators.

```Python
.aggregate(initial_value, combine_value_with_accumulator, combine_accumulators)
```
Consider an RDD of durations and we would like to compute the total duration,
and the number of elements in the RDD (`.count()`).

```Python
# Create an RDD of durations and is paritioned into 2.
duration = sc.parallelize([0, 0.1, 0.2, 0.4, 0.], 2)
```

In partition #1 we have `[0, 0.1, 0.2]` and partition #2 has `[0.4, 0]`. To
compute the total duration and number of elements in duration, we can use the
aggregate function. Alternatively, to get the total duration we could do a
`.reduce(add)` action and the number of elements could be obtained using
`.count()`. However, this requires iterating through the RDD twice. To obtain
the `(sum, count)` equivalent of `(.reduce(add), .count())`, we use the
`.aggregate()` method.

The `aggregate` method would look something like this:
```Python
# .aggregate( inital_value, combine_element_accumulator, combine_accumulators)
sum_count = duration.aggregate(
    (0, 0) #initial values (sum, count),
    (lambda acc, value: (acc[0] + value, acc[1] + 1)), # combine value with acc))
    (lambda acc1, acc2: (acc1[0] + acc2[0], acc1[1] + acc2[1])) # combine accumulators
)
```

In partition 1, as the aggregate function iterates through each element in the
task, and encounters the first element `0`, the initial value of the accumulator
is `(0, 0)` and runs the `combine value with acc` (first function) in the
`aggregate()`:

```Python
(lambda acc, value: (acc[0] + value, acc[1] + 1)), # combine value with acc))
# The acc is the initial value (0, 0) and value is 0 (first element)
# (lambda (0, 0), 0: (0 + 0, 0 + 1)) => (0, 1)
# for the second element 0.1
# (lambda (0, 1), 0.1: (0 + 0.1, 1 + 1)) => (0.1, 2)
# for the third element 0.2
# (lambda (0.1, 2), 0.1: (0.1 + 0.2, 2 + 1)) => (0.3, 3)
```

Similarly in partition #2, which has `(0.4, 0)`, yields an accumulated values of
`(sum, count)` as `(0.4, 2)`. This is followed by combine accumulators function.

```Python
(lambda acc1, acc2: (acc1[0] + acc2[0], acc1[1] + acc2[1])) # combine accumulators
# this is equivalent to combining acc1 from partition 1 and acc2 from part 2.
# (lambda (0.3, 3), (0.4, 2): (0.3 + 0.4, 3 + 2)) # combine accumulators => (0.7, 5)      
```

Thus the combine accumulators yield a value of `(sum, count)` as `(0.7, 5)`.
The use `.aggregate(...)` means the RDD is iterated through only once, which
improves the efficiency.
