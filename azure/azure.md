# Getting started with Apache Spark in Azure

Learn how to create an [Apache Spark](hdinsight-apache-spark-overview.md) cluster in HDInsight and run interactive Spark SQL queries using [Jupyter](https://jupyter.org) notebook.

![Quickstart diagram describing steps to create an Apache Spark cluster on Azure HDInsight](hdinsight-spark-quickstart-interactive-spark-query-flow.png "Spark quickstart using Apache Spark in HDInsight. Steps illustrated: create a cluster; run Spark interactive query")

## Prerequisites

* **An Azure subscription**. Before you begin this tutorial, you must have an Azure subscription. See [Create your free Azure account today](https://azure.microsoft.com/free).

## What is Apache Spark on Azure HDInsight?
Spark clusters on HDInsight offer a fully managed Spark service. Benefits of creating a Spark cluster on HDInsight are listed here.

| Feature | Description |
| --- | --- |
| Ease of creating Spark clusters |You can create a new Spark cluster on HDInsight in minutes using the Azure Portal, Azure PowerShell, or the HDInsight .NET SDK. |
| Ease of use |Spark cluster in HDInsight include Jupyter and Zeppelin notebooks. You can use these for interactive data processing and visualization.|
| REST APIs |Spark clusters in HDInsight include [Livy](https://github.com/cloudera/hue/tree/master/apps/spark/java#welcome-to-livy-the-rest-spark-server), a REST API-based Spark job server to remotely submit and monitor jobs. |
| Support for Azure Data Lake Store | Spark cluster on HDInsight can be configured to use Azure Data Lake Store as an additional storage, as well as primary storage (only with HDInsight 3.5 clusters). |
| Integration with Azure services |Spark cluster on HDInsight comes with a connector to Azure Event Hubs. Customers can build streaming applications using the Event Hubs, in addition to [Kafka](http://kafka.apache.org/), which is already available as part of Spark. |
| Support for R Server | You can set up a R Server on HDInsight Spark cluster to run distributed R computations with the speeds promised with a Spark cluster. |
| Integration with third-party IDEs | HDInsight provides plugins for IDEs like IntelliJ IDEA and Eclipse that you can use to create and submit applications to an HDInsight Spark cluster.|
| Concurrent Queries |Spark clusters in HDInsight support concurrent queries. This enables multiple queries from one user or multiple queries from various users and applications to share the same cluster resources. |
| Caching on SSDs |You can choose to cache data either in memory or in SSDs attached to the cluster nodes. Caching in memory provides the best query performance but could be expensive; caching in SSDs provides a great option for improving query performance without the need to create a cluster of a size that is required to fit the entire dataset in memory. |
| Integration with BI Tools |Spark clusters on HDInsight provide connectors for  BI tools such as [Power BI](http://www.powerbi.com/) and [Tableau](http://www.tableau.com/products/desktop) for data analytics. |
| Pre-loaded Anaconda libraries |Spark clusters on HDInsight come with Anaconda libraries pre-installed. [Anaconda](http://docs.continuum.io/anaconda/) provides close to 200 libraries for machine learning, data analysis, visualization, etc. |
| Scalability |Although you can specify the number of nodes in your cluster during creation, you may want to grow or shrink the cluster to match workload. All HDInsight clusters allow you to change the number of nodes in the cluster. Also, Spark clusters can be dropped with no loss of data since all the data is stored in Azure Storage or Data Lake Store. |

### Kernels for Jupyter notebook on Spark clusters in Azure HDInsight

HDInsight Spark clusters provide kernels that you can use with the Jupyter notebook on Spark for testing your applications. A kernel is a program that runs and interprets your code. The three kernels are:

- **PySpark** - for applications written in Python2
- **PySpark3** - for applications written in Python3
- **Spark** - for applications written in Scala

### Benefits of using the kernels

Here are a few benefits of using the new kernels with Jupyter notebook on Spark HDInsight clusters.

- **Preset contexts**. With  **PySpark**, **PySpark3**, or the **Spark** kernels, you do not need to set the Spark or Hive contexts explicitly before you start working with your applications. These are available by default. These contexts are:

   * **sc** - for Spark context
   * **sqlContext** - for Hive context

    So, you don't have to run statements like the following to set the contexts:

      	sc = SparkContext('yarn-client')
      	sqlContext = HiveContext(sc)

    Instead, you can directly use the preset contexts in your application.

- **Cell magics**. The PySpark kernel provides some predefined “magics”, which are special commands that you can call with `%%` (for example, `%%MAGIC` <args>). The magic command must be the first word in a code cell and allow for multiple lines of content. The magic word should be the first word in the cell. Adding anything before the magic, even comments, causes an error.     For more information on magics, see [here](http://ipython.readthedocs.org/en/stable/interactive/magics.html).

The following table lists the different magics available through the kernels.

   | Magic | Example | Description |
   | --- | --- | --- |
   | help |`%%help` |Generates a table of all the available magics with example and description |
   | info |`%%info` |Outputs session information for the current Livy endpoint |
   | configure |`%%configure -f`<br>`{"executorMemory": "1000M"`,<br>`"executorCores": 4`} |Configures the parameters for creating a session. The force flag (-f) is mandatory if a session has already been created, which ensures that the session is dropped and recreated. Look at [Livy's POST /sessions Request Body](https://github.com/cloudera/livy#request-body) for a list of valid parameters. Parameters must be passed in as a JSON string and must be on the next line after the magic, as shown in the example column. |
   | sql |`%%sql -o <variable name>`<br> `SHOW TABLES` |Executes a Hive query against the sqlContext. If the `-o` parameter is passed, the result of the query is persisted in the %%local Python context as a [Pandas](http://pandas.pydata.org/) dataframe. |
   | local |`%%local`<br>`a=1` |All the code in subsequent lines is executed locally. Code must be valid Python2 code even irrespective of the kernel you are using. So, even if you selected **PySpark3** or **Spark** kernels while creating the notebook, if you use the `%%local` magic in a cell, that cell must only have valid Python2 code.. |
   | logs |`%%logs` |Outputs the logs for the current Livy session. |
   | delete |`%%delete -f -s <session number>` |Deletes a specific session of the current Livy endpoint. Note that you cannot delete the session that is initiated for the kernel itself. |
   | cleanup |`%%cleanup -f` |Deletes all the sessions for the current Livy endpoint, including this notebook's session. The force flag -f is mandatory. |

> **Info** In addition to the magics added by the PySpark kernel, you can also use the [built-in IPython magics](https://ipython.org/ipython-doc/3/interactive/magics.html#cell-magics), including `%%sh`. You can use the `%%sh` magic to run scripts and block of code on the cluster headnode.

- **Auto visualization**. The **Pyspark** kernel automatically visualizes the output of Hive and SQL queries. You can choose between several different types of visualizations including Table, Pie, Line, Area, Bar.

> This section is from: [Microsoft Azure Docs](https://github.com/MicrosoftDocs/azure-docs)
