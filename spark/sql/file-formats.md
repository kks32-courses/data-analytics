# File Operations

## File formats
Spark makes it very simple to load and save data in a large number of file
formats. Formats range from unstructured, like text, to semistructured, like
JSON, to structured, like SequenceFiles. The input formats that Spark wraps all
transparently handle compressed formats based on the file extension.

![File formats](file-formats.png)

Text files are very simple to load from and save to with Spark. When we load 
a single text file as an RDD, each input line becomes an element in the RDD. We can also
load multiple whole text files at the same time into a pair RDD, with the key being the
name and the value being the contents of each file. Loading a single text file is as simple as calling the `textFile()` function on our SparkContext with the path to the file:

```Python
input = sc.textFile("file:///user/livy/two-cities.txt")
```
