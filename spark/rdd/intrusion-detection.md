# Exercise: Intrusion detection

[KDD99 cup](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html)
 is the data set used for The Third International Knowledge Discovery
and Data Mining Tools Competition, which was held in conjunction with KDD-99
The Fifth International Conference on Knowledge Discovery and Data Mining.
The competition task was to build a network intrusion detector, a predictive
model capable of distinguishing between `bad` connections, called intrusions or
attacks, and `good` normal connections. This database contains a standard set
of data to be audited, which includes a wide variety of intrusions simulated in
a military network environment.

In this exercise we will use the reduced dataset (10 percent) provided for the
KDD Cup 1999, containing nearly half million network interactions. The file is
provided as a Gzip file that we will upload to `/user/livy` in the `kbtu`
blob storage.

## Challenge

* Load the [raw data](http://kdd.ics.uci.edu/databases/kddcup99/kddcup.data_10_percent.gz)
 and count the number of records.
* Print the first 5 lines of the raw data.
* Filter and count only the `normal` interactions and measure how long the
computation takes.
* Sample the data to measure the percentage of normal interaction and compare it
with the whole data. Measure the duration of computation in both cases. (Hint:
use `sample()` function to sample a portion of the data).
* Count the number of attack interactions. (Hint: `subtract` normal interactions
  from the entire dataset to get `attack` interactions).
* Extract protocols (second column in the CSV) and services (third column).
Create all possible pairs of protocols and services. (Hint: Use `cartesian`).
* Measure the total and mean duration of `normal` and `attack` interactions.
The state is defined in column 41, and the duration is in column 0. Use
`aggregate` to do the same.

> Hint for measuring time

```Python
from time import time
t0 = time()
# computation
tt = time() - t0
print ("Count completed in {} seconds".format(round(tt,3)))
```

<!--
> [Solution: Jupyter notebook for intrusion detection](kdd99.ipynb)
-->
