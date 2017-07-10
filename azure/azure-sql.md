### Azure HDInsight parameters supported with the %%sql magic
The `%%sql` magic supports different parameters that you can use to control the kind of output that you receive when you run queries. The following table lists the output.

| Parameter | Example | Description |
| --- | --- | --- |
| -o |`-o <VARIABLE NAME>` |Use this parameter to persist the result of the query, in the %%local Python context, as a [Pandas](http://pandas.pydata.org/) dataframe. The name of the dataframe variable is the variable name you specify. |
| -q |`-q` |Use this to turn off visualizations for the cell. If you don't want to auto-visualize the content of a cell and just want to capture it as a dataframe, then use `-q -o <VARIABLE>`. If you want to turn off visualizations without capturing the results (for example, for running a SQL query, like a `CREATE TABLE` statement), use `-q` without specifying a `-o` argument. |
| -m |`-m <METHOD>` |Where **METHOD** is either **take** or **sample** (default is **take**). If the method is **take**, the kernel picks elements from the top of the result data set specified by MAXROWS (described later in this table). If the method is **sample**, the kernel randomly samples elements of the data set according to `-r` parameter, described next in this table. |
| -r |`-r <FRACTION>` |Here **FRACTION** is a floating-point number between 0.0 and 1.0. If the sample method for the SQL query is `sample`, then the kernel randomly samples the specified fraction of the elements of the result set for you. For example, if you run a SQL query with the arguments `-m sample -r 0.01`, then 1% of the result rows are randomly sampled. |
| -n |`-n <MAXROWS>` |**MAXROWS** is an integer value. The kernel limits the number of output rows to **MAXROWS**. If **MAXROWS** is a negative number such as **-1**, then the number of rows in the result set is not limited. |

**Example:**

    %%sql -q -m sample -r 0.1 -n 500 -o query2
    SELECT * FROM hivesampletable

The statement above does the following:

* Selects all records from **hivesampletable**.
* Because we use -q, it turns off auto-visualization.
* Because we use `-m sample -r 0.1 -n 500` it randomly samples 10% of the rows in the hivesampletable and limits the size of the result set to 500 rows.
* Finally, because we used `-o query2` it also saves the output into a dataframe called **query2**.
