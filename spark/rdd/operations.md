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
```
