# Exercise: Word count

Let's count the frequency of words that appear in Charles Dicken's `A tale of
two cities`. Do not include special characters when counting the word frequency.

* Upload the [`A tale of two cities`](two-cities.txt) file to `/user/livy` in
the storage container attached to the Spark cluster.

![Azure upload to blob storage](upload-blob-storage.png)

* This allows you to read the text file from the root directory directly as
`sc.textFile("two-cities.txt")`.

* Start by creating a new RDD by splitting the words.

* Then create an RDD of pairs `(word, 1)` for each occurrence of a word using
a `map()`

* Finally, use the `reduceByKey()` function, which merges the values for each
key using an associative reduce function.

* The following function removes any special characters:

```Python
# Define a function to remove any special characters
def removePunctuation(text):
    """Removes punctuation, changes to lower case, and strips leading and trailing spaces.
    Note:
        Only spaces, letters, and numbers should be retained.  Other characters should should be
        eliminated (e.g. it's becomes its).  Leading and trailing spaces should be removed after
        punctuation is removed.
    Args:
        text (str): A string.
    Returns:
        str: The cleaned up string.
    """
    return re.sub(r'[^a-z0-9\s]','',text.lower().strip())
```

> [Solution: WordCount Jupyter notebook](word-count.ipynb)
