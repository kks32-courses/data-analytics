# Exercise: Calculating Pi

Spark can be used for compute-intensive tasks. This code estimates π by
"throwing darts" at a circle. We pick random points in the unit square
((0, 0) to (1,1)) and see how many fall in the unit circle.
The fraction should be π / 4, so we use this to get our estimate.

* Write a function to generate random coordinates and return `True` if the point
is within the circle `x^2 + y^2 < 1`.

* Generate `n` samples, and use a filter to `count` the number of points within
the circle.

* Pi is `4.0 * count / n`

> [Solution for Pi Jupyter notebooks](pi.ipynb)
