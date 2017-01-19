# Exercises created by me for preparing the certification of CCA Spark and Hadoop Developer Certification

## Data Ingest

### Import data from a MySQL database into HDFS using Sqoop

* The *help* allows you to see a list of all tools

```bash
sqoop help
````
* The *list-tables* list all tables of a database
```bash
sqoop list-tables \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser \
--password pw
```
* The *import-all-tables* imports an entire database

```bash
sqoop import-all-tables \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser --password pw \
```

By default, Sqoop will import a table named foo to a directory named foo inside your home directory in HDFS. For example, if your username is someuser, then the import tool will write to /user/someuser/foo/(files). You can adjust the parent directory of the import with the --warehouse-dir argument. For example:

```bash
sqoop import --connnect <connect-str> --table foo --warehouse-dir /shared
```
This command would write to a set of files in the /shared/foo/ directory.

You can also explicitly choose the target directory, like so:
```bash
sqoop import --connnect <connect-str> --table foo --target-dir /dest
```

This will import the files into the /dest directory. --target-dir is incompatible with --warehouse-dir.


* The *import* imports a single table
```bash
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
```bash
sqoop import --table table1 \
--connect jdbc:mysql://dbhost/database1 \
--username dbuser --password pw \
--incremental lastmodified \
--check-column column1 \
--last-value '2017-01-19 18:09:00'
```


