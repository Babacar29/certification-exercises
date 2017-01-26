ººtr# Theory focused on the required skills of CCA Spark and Hadoop Developer Certification

## Table of contents
1. [Introduction](#1-introduction)
2. [Data Ingest](#2-data-ingest)
 1. [Import data from a MySQL database into HDFS using Sqoop](#i-import-data-from-a-mysql-database-into-hdfs-using-sqoop)
 2. [Export data to a MySQL database from HDFS using Sqoop](#ii-export-data-to-a-mysql-database-from-hdfs-using-sqoop)
 3. [Change the delimiter and file format of data during import using Sqoop](#iii-change-the-delimiter-and-file-format-of-data-during-import-using-sqoop)
 4. [Ingest real-time and near-real time (NRT) streaming data into HDFS using Flume](#iv-ingest-real-time-and-near-real-time-nrt-streaming-data-into-hdfs-using-flume)
 5. [Load data into and out of HDFS using the Hadoop File System (FS) commands](#v-load-data-into-and-out-of-hdfs-using-the-hadoop-file-system-fs-commands)
3. [Transform, Stage, Store](#3-transform-stage-store)
  1. [Load data from HDFS and store results back to HDFS using Spark](#i-load-data-from-hdfs-and-store-results-back-to-hdfs-using-spark)
  2. [Join disparate datasets together using Spark](#ii-join-disparate-datasets-together-using-spark)
  3. [Calculate aggregate statistics (e.g., average or sum) using Spark](#iii-calculate-aggregate-statistics-eg-average-or-sum-using-spark)
  4. [Filter data into a smaller dataset using Spark](#iv-filter-data-into-a-smaller-dataset-using-spark)
  5. [Write a query that produces ranked or sorted data using Spark](#v-write-a-query-that-produces-ranked-or-sorted-data-using-spark)
4. [Data Analysis](#4-data-analysis)
  1. [Read and/or create a table in the Hive metastore in a given schema](#i-read-andor-create-a-table-in-the-hive-metastore-in-a-given-schema)
  2. [Extract an Avro schema from a set of datafiles using avro-tools](#ii-extract-an-avro-schema-from-a-set-of-datafiles-using-avro-tools)
  3. [Create a table in the Hive metastore using the Avro file format and an external schema file](#iii-create-a-table-in-the-hive-metastore-using-the-avro-file-format-and-an-external-schema-file)
  4. [Improve query performance by creating partitioned tables in the Hive metastore](#iv-improve-query-performance-by-creating-partitioned-tables-in-the-hive-metastore)
  5. [Evolve an Avro schema by changing JSON files](#v-evolve-an-avro-schema-by-changing-json-files)

## 1. Introduction

In this page you can find a summary of the theory focused on the required skills of CCA Spark and Hadoop Developer Certification.

For more information visit the following link: http://www.cloudera.com/training/certification/cca-spark.html

:back: [[Back to table of contents]](#table-of-contents)

## 2. Data Ingest

### i. Import data from a MySQL database into HDFS using Sqoop

* The *help* allows you to see a list of all tools

```
sqoop help
```
* The *list-tables* list all tables of a database

```
sqoop list-tables \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser \
--password pw
```
* The *import-all-tables* imports an entire database

```
sqoop import-all-tables \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser \
--password pw \
```

By default, Sqoop will import a table named foo to a directory named foo inside your home directory in HDFS. For example, if your username is someuser, then the import tool will write to /user/someuser/foo/(files). You can adjust the parent directory of the import with the `--warehouse-dir` argument. For example:

```
sqoop import --connnect <connect-str> --table foo --warehouse-dir /shared
```
This command would write to a set of files in the /shared/foo/ directory.

You can also explicitly choose the target directory, like so:
```
sqoop import --connnect <connect-str> --table foo --target-dir /dest
```

This will import the files into the /dest directory. `--target-dir` is incompatible with `--warehouse-dir`.


* The *import* imports a single table

```
sqoop import --table table1 \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser \
--password pw \
--fields-terminated-by "\t"
```
* Sqoop’s incremental mode

Argument | Description
--- | ---
--check-column (col) | Specifies the column to be examined when determining which rows to import.
--incremental (mode) | Specifies how Sqoop determines which rows are new. Legal values for mode include append and lastmodified.
--last-value (value) | Specifies the maximum value of the check column from the previous import.

You should specify append mode when importing a table where new rows are continually being added with increasing row id values. You specify the column containing the row’s id with `--check-column`. Sqoop imports rows where the check column has a value greater than the one specified with `--last-value`.

An alternate table update strategy supported by Sqoop is called lastmodified mode. You should use this when rows of the source table may be updated, and each such update will set the value of a last-modified column to the current timestamp. Rows where the check column holds a timestamp more recent than the timestamp specified with `--last-value` are imported.

At the end of an incremental import, the value which should be specified as `--last-value` for a subsequent import is printed to the screen. When running a subsequent import, you should specify `--last-value` in this way to ensure you import only the new or updated data.

Example:
```
sqoop import --table table1 \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser \
--password pw \
--incremental lastmodified \
--check-column column1 \
--last-value '2017-01-19 18:09:00'
```

* Selecting the Data to Import

  * Import only specified columns

  ```
  sqoop import --table table1 \
  --connect jdbc:mysql://dbhost/database1 \
  --username dbuser \
  --password pw \
  --columns "column1,column2,column5"
  ```

  * Filtering

  ```
  sqoop import --table table1 \
  --connect jdbc:mysql://dbhost/database1 \
  --username dbuser \
  --password pw \
  --where "column1='value1'"
  ```

  * Free form query imports

  Sqoop can also import the result set of an arbitrary SQL query. Instead of using the `--table`, `--columns` and `--where` arguments, you       can   specify a SQL statement with the `--query` argument.

  When importing a free-form query, you must specify a destination directory with `--target-dir`.

  If you want to import the results of a query in parallel, then each map task will need to execute a copy of the query, with results     partitioned by bounding conditions inferred by Sqoop. Your query must include the token $CONDITIONS which each Sqoop process will       replace with a unique condition expression. You must also select a splitting column with `--split-by`.

  ```
  sqoop import \
  --query 'SELECT a.*, b.* FROM a JOIN b on (a.id == b.id) WHERE $CONDITIONS' \
  --split-by a.id --target-dir /user/foo/joinresults
  ```

:back: [[Back to table of contents]](#table-of-contents)

### ii. Export data to a MySQL database from HDFS using Sqoop
The export tool exports a set of files from HDFS back to an RDBMS. The input files are read and parsed into a set of records according to the user-specified delimiters.

The default operation is to transform these into a set of *INSERT* statements that inject the records into the database. In `--update mode`, Sqoop will generate *UPDATE* statements that replace existing records in the database.


```
sqoop export \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser \
--password pw \
--export-dir /databas1/output \
--update-mode allowinsert \
--table table1
```
> :exclamation: The target table must already exist in the database.

:back: [[Back to table of contents]](#table-of-contents)

### iii. Change the delimiter and file format of data during import using Sqoop

* Change the delimiter

  The default delimiters are a comma (,) for fields, a newline (\n) for records, no quote character, and no escape character.

  Delimiters may be specified as:

    - a character (--fields-terminated-by X)
    - an escape character (--fields-terminated-by \t). Examples of escape characters are:
      - \b (backspace)
      - \n (newline)
      - \r (carriage return)
      - \t (tab)
      - \" (double-quote)
      - \' (single-quote)
      - \\\ (backslash)
    - The octal representation of a UTF-8 character's code point. This should be of the form \\0ooo, where ooo is the octal value. For example, `--fields-terminated-by` \001 would yield the ^A character.
    - The hexadecimal representation of a UTF-8 character's code point. This should be of the form \0xhhh, where hhh is the hex value. For example, `--fields-terminated-by` \0x10 would yield the carriage return character.

Output line formatting arguments:

| Argument | Description     
| :------------- | :-------------
|--enclosed-by (char)	| Sets a required field enclosing character
|--escaped-by (char)	| Sets the escape character
|--fields-terminated-by (char)	| Sets the field separator character
|--lines-terminated-by (char)	|Sets the end-of-line character


* File format of data
  * Import as hive table

  ```
  sqoop import \
  --connect jdbc:mysql://localhost/database1 \
  --username dbuser \
  --password pw \
  --fields-terminated-by ',' \
  --table table1 \
  --hive-database hivedatabase1 \
  --hive-table hivetable1 \
  --hive-import
  ```

  * Import as avro data file

  sqoop import saves the schema in a JSON file in the local path where the sqoop sentence is executed.

  ```
  sqoop import \
  --connect jdbc:mysql://localhost/database1 \
  --username dbuser \
  --password pw \
  --table table1 \
  --target-dir /foo/file_avro \
  --as-avrodatafile
  ```

  * Import as parquet file

  ```
  sqoop import \
  --connect jdbc:mysql://localhost/database1 \
  --username dbuser \
  --password pw \
  --table table1 \
  --target-dir /foo/file_parquet \
  --as-parquetfile
  ```

:back: [[Back to table of contents]](#table-of-contents)

### iv. Ingest real-time and near-real time (NRT) streaming data into HDFS using Flume

* Setting up an agent

Flume agent configuration is stored in a local configuration file. This is a text file that follows the Java properties file format. Configurations for one or more agents can be specified in the same configuration file. The configuration file includes properties of each source, sink and channel in an agent and how they are wired together to form data flows.

* Configuring individual components

Each component (source, sink or channel) in the flow has a name, type, and set of properties that are specific to the type and instantiation. For example, an Avro source needs a hostname (or IP address) and a port number to receive data from. A memory channel can have max queue size (“capacity”), and an HDFS sink needs to know the file system URI, path to create files, frequency of file rotation (“hdfs.rollInterval”) etc. All such attributes of a component needs to be set in the properties file of the hosting Flume agent.

* Wiring the pieces together

The agent needs to know what individual components to load and how they are connected in order to constitute the flow. This is done by listing the names of each of the sources, sinks and channels in the agent, and then specifying the connecting channel for each sink and source. For example, an agent flows events from an Avro source called avroWeb to HDFS sink hdfs-cluster1 via a file channel called file-channel. The configuration file will contain names of these components and file-channel as a shared channel for both avroWeb source and hdfs-cluster1 sink.

* A simple example

Here, we give an example configuration file, describing a single-node Flume deployment. This configuration lets a user generate events and subsequently logs them to the console.

```
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

This configuration defines a single agent named a1. a1 has a source that listens for data on port 44444, a channel that buffers event data in memory, and a sink that logs event data to the console. The configuration file names the various components, then describes their types and configuration parameters.

Given this configuration file, we can start Flume as follows:

```
flume-ng agent --conf /etc/flume-ng/conf \
--conf-file example.conf \
--name a1 -Dflume.root.logger=INFO,console
```

* Some other Examples

  * Spooling Directory Source

  This source lets you ingest data by placing files to be ingested into a “spooling” directory on disk. This source will watch the specified directory for new files, and will parse events out of new files as they appear. The event parsing logic is pluggable. After a given file has been fully read into the channel, it is renamed to indicate completion (or optionally deleted).

 Example for an agent named agent-1:

  ```
  a1.channels = ch-1
  a1.sources = src-1

  a1.sources.src-1.type = spooldir
  a1.sources.src-1.channels = ch-1
  a1.sources.src-1.spoolDir = /var/log/apache/flumeSpool
  a1.sources.src-1.fileHeader = true
  ```

  * HDFS sink

  This sink writes events into the Hadoop Distributed File System (HDFS). It currently supports creating text (`DataStream`) and `SequenceFile` (default). It supports compression in both file types. The files can be rolled (close current file and create a new one) periodically based on the elapsed time or size of data or number of events. It also buckets/partitions data by attributes like timestamp or machine where the event originated.

  Example for agent named a1:

  ```
  a1.channels = c1
  a1.sinks = k1
  a1.sinks.k1.type = hdfs
  a1.sinks.k1.channel = c1
  a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
  a1.sinks.k1.hdfs.filePrefix = events-
  a1.sinks.k1.hdfs.round = true
  a1.sinks.k1.hdfs.roundValue = 10
  a1.sinks.k1.hdfs.roundUnit = minute
  a1.sinks.k1.hdfs.rollInterval = 0
  a1.sinks.k1.hdfs.rollSize = 524288
  a1.sinks.k1.hdfs.rollCount = 0
  a1.sinks.k1.hdfs.fileType = DataStream
  ```

  The above configuration will round down the timestamp to the last 10th minute. For example, an event with timestamp 11:54:34 AM, June 12, 2012 will cause the hdfs path to become /flume/events/2012-06-12/1150/00.



:back: [[Back to table of contents]](#table-of-contents)

### v. Load data into and out of HDFS using the Hadoop File System (FS) commands

Show the content of HDFS directory:
```
hdfs dfs -ls
```
Upload a file to HDFS:
```
hdfs dfs -put <localDocumentName> <HDFSDocumentName>
```
Download a file to Local from HDFS:
```
hdfs dfs -get <HDFS directory>/<HDFS filename> <Localfilename>
```
Remove a fields from HDFS:
```
hdfs dfs -rm -R [-skipTrash]
```
> :exclamation: Be careful with `-skipTrash` option because it will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.

:back: [[Back to table of contents]](#table-of-contents)

## 3. Transform, Stage, Store

### i. Load data from HDFS and store results back to HDFS using Spark

In this example, we use a few transformations to build a dataset of (String, Int) pairs called counts and then save it to a file.

```scala
val textFile = sc.textFile("hdfs://...")
val counts = textFile.flatMap(line => line.split(" "))
                 .map(word => (word, 1))
                 .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
```

:back: [[Back to table of contents]](#table-of-contents)

### ii. Join disparate datasets together using Spark

* SPARK RDD

  * Join [Pair]

  Performs an inner join using two key-value RDDs. Please note that the keys must be generally comparable to make this work.

  Listing Variants
  ```scala
  def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]
  def join[W](other: RDD[(K, W)], numPartitions: Int): RDD[(K, (V, W))]
  def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))]
  ```

  Example

  ```scala
  val a = sc.parallelize(List("dog", "salmon", "salmon", "rat", "elephant"), 3)
  val b = a.keyBy(_.length)
  val c = sc.parallelize(List("dog","cat","gnu","salmon","rabbit","turkey","wolf","bear","bee"), 3)
  val d = c.keyBy(_.length)
  b.join(d).collect

  res0: Array[(Int, (String, String))] = Array((6,(salmon,salmon)), (6,(salmon,rabbit)), (6,(salmon,turkey)), (6,(salmon,salmon)), (6,(salmon,rabbit)), (6,(salmon,turkey)), (3,(dog,dog)), (3,(dog,cat)), (3,(dog,gnu)), (3,(dog,bee)), (3,(rat,dog)), (3,(rat,cat)), (3,(rat,gnu)), (3,(rat,bee)))
  ```

  * leftOuterJoin [Pair]

  Performs an left outer join using two key-value RDDs. Please note that the keys must be generally comparable to make this work correctly.

  Listing Variants

  ```scala
  def leftOuterJoin[W](other: RDD[(K, W)]): RDD[(K, (V, Option[W]))]
  def leftOuterJoin[W](other: RDD[(K, W)], numPartitions: Int): RDD[(K, (V, Option[W]))]
  def leftOuterJoin[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, Option[W]))]
  ```

  Example

  ```scala
  val a = sc.parallelize(List("dog", "salmon", "salmon", "rat", "elephant"), 3)
  val b = a.keyBy(_.length)
  val c = sc.parallelize(List("dog","cat","gnu","salmon","rabbit","turkey","wolf","bear","bee"), 3)
  val d = c.keyBy(_.length)
  b.leftOuterJoin(d).collect

  res1: Array[(Int, (String, Option[String]))] = Array((6,(salmon,Some(salmon))), (6,(salmon,Some(rabbit))), (6,(salmon,Some(turkey))), (6,(salmon,Some(salmon))), (6,(salmon,Some(rabbit))), (6,(salmon,Some(turkey))), (3,(dog,Some(dog))), (3,(dog,Some(cat))), (3,(dog,Some(gnu))), (3,(dog,Some(bee))), (3,(rat,Some(dog))), (3,(rat,Some(cat))), (3,(rat,Some(gnu))), (3,(rat,Some(bee))), (8,(elephant,None)))
  ```

  * SPARK DataFrame

  // Inner join implicit
  df1.join(df2, df1("field1") === df2("field1"))

  // Inner join explicit
  df1.join(df2, df1("field1") === df2("field1"), "inner")

  // Left outer join explicit
  df1.join(df2, df1("field1") === df2("field1"), "left_outer")

  // Right outer join explicit
  df1.join(df2, df1("field1") === df2("field1"), "right_outer")

:back: [[Back to table of contents]](#table-of-contents)

### iii. Calculate aggregate statistics (e.g., average or sum) using Spark

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

### iv. Filter data into a smaller dataset using Spark

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

### v. Write a query that produces ranked or sorted data using Spark

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

## 4. Data Analysis

### i. Read and/or create a table in the Hive metastore in a given schema

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

### ii. Extract an Avro schema from a set of datafiles using avro-tools

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

### iii. Create a table in the Hive metastore using the Avro file format and an external schema file

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

### iv. Improve query performance by creating partitioned tables in the Hive metastore

coming soon...

:back: [[Back to table of contents]](#table-of-contents)

### v. Evolve an Avro schema by changing JSON files

coming soon...

:back: [[Back to table of contents]](#table-of-contents)
