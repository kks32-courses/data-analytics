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
| `sample(withReplacement, fraction, [seed])` | Sample an RDD, with or without replacement. | `rdd.sample(false, 0.5)` | Non-deterministic |
