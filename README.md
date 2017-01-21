# Theory focused on the required skills of CCA Spark and Hadoop Developer Certification

## Table of contents
1. [Data Ingest] (#1-data-ingest)
 1. [Import data from a MySQL database into HDFS using Sqoop](#i-import-data-from-a-mysql-database-into-hdfs-using-sqoop)
 2. [Export data to a MySQL database from HDFS using Sqoop](#ii-export-data-to-a-mysql-database-from-hdfs-using-sqoop)

## 1. Data Ingest

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
--username dbuser --password pw \
```

By default, Sqoop will import a table named foo to a directory named foo inside your home directory in HDFS. For example, if your username is someuser, then the import tool will write to /user/someuser/foo/(files). You can adjust the parent directory of the import with the --warehouse-dir argument. For example:

```
sqoop import --connnect <connect-str> --table foo --warehouse-dir /shared
```
This command would write to a set of files in the /shared/foo/ directory.

You can also explicitly choose the target directory, like so:
```
sqoop import --connnect <connect-str> --table foo --target-dir /dest
```

This will import the files into the /dest directory. --target-dir is incompatible with --warehouse-dir.


* The *import* imports a single table

```
sqoop import --table table1 \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser --password pw \
--fields-terminated-by "\t"
```
* Sqoop’s incremental mode

Argument | Description
--- | ---
--check-column (col) | Specifies the column to be examined when determining which rows to import.
--incremental (mode) | Specifies how Sqoop determines which rows are new. Legal values for mode include append and lastmodified.
--last-value (value) | Specifies the maximum value of the check column from the previous import.

You should specify append mode when importing a table where new rows are continually being added with increasing row id values. You specify the column containing the row’s id with --check-column. Sqoop imports rows where the check column has a value greater than the one specified with --last-value.

An alternate table update strategy supported by Sqoop is called lastmodified mode. You should use this when rows of the source table may be updated, and each such update will set the value of a last-modified column to the current timestamp. Rows where the check column holds a timestamp more recent than the timestamp specified with --last-value are imported.

At the end of an incremental import, the value which should be specified as --last-value for a subsequent import is printed to the screen. When running a subsequent import, you should specify --last-value in this way to ensure you import only the new or updated data.

Example:
```
sqoop import --table table1 \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser --password pw \
--incremental lastmodified \
--check-column column1 \
--last-value '2017-01-19 18:09:00'
```

* Selecting the Data to Import

  * Import only specified columns

  ```
  sqoop import --table table1 \
  --connect jdbc:mysql://dbhost/database1 \
  --username dbuser --password pw \
  --columns "column1,column2,column5"
  ```

  * Filtering

  ```
  sqoop import --table table1 \
  --connect jdbc:mysql://dbhost/database1 \
  --username dbuser --password pw \
  --where "column1='value1'"
  ```

  * Free form query imports

  Sqoop can also import the result set of an arbitrary SQL query. Instead of using the --table, --columns and --where arguments, you       can   specify a SQL statement with the --query argument.

  When importing a free-form query, you must specify a destination directory with --target-dir.

  If you want to import the results of a query in parallel, then each map task will need to execute a copy of the query, with results     partitioned by bounding conditions inferred by Sqoop. Your query must include the token $CONDITIONS which each Sqoop process will       replace with a unique condition expression. You must also select a splitting column with --split-by.

  ```
  sqoop import \
  --query 'SELECT a.*, b.* FROM a JOIN b on (a.id == b.id) WHERE $CONDITIONS' \
  --split-by a.id --target-dir /user/foo/joinresults
  ```

### ii. Export data to a MySQL database from HDFS using Sqoop
The export tool exports a set of files from HDFS back to an RDBMS. The target table must already exist in the database. The input files are read and parsed into a set of records according to the user-specified delimiters.

The default operation is to transform these into a set of *INSERT* statements that inject the records into the database. In "update mode," Sqoop will generate *UPDATE* statements that replace existing records in the database.

NOTE: The RDBMS table must already exist prior to export
```
sqoop export \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser --password pw \
--export-dir /databas1/output \
--update-mode allowinsert \
--table table1
```
