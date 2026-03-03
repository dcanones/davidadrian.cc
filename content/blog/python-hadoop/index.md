---
title: "Hadoop with Python"
date: 2019-05-01T12:00:00.000Z
lastmod: 2020-03-24T16:25:58.000Z
draft: false
description: "Tutorial on how to interact with Hadoop using Python libraries like mrjob and Snakebite."
tags:
  - "Tutorials"
aliases:
  - /python-hadoop/
featureimage: "feature.jpg"
---

Following this guide you will learn things like:

-   How to load file from **Hadoop** Distributed Filesystem directly info memory
-   Moving files from local to **HDFS**
-   Setup a **Spark** local installation using **conda**
-   Loading data from **HDFS** to a Spark or **pandas** DataFrame
-   Leverage libraries like: pyarrow, impyla, python-hdfs, ibis, etc.

First, let's import some libraries we will be using everywhere in this tutorial, specially **pandas**:

```python
from pathlib import Path
import pandas as pd
import numpy as np
```

## pyspark: Apache Spark

First of all, install findspark, and also pyspark in case you are  working in a local computer. If you are following this tutorial in a  Hadoop cluster, can skip pyspark install. For simplicity I will use  conda virtual environment manager (pro tip: create a virtual environment  before starting and do not break your system Python install!).

```bash
conda install -c conda-forge findspark -y           
conda install -c conda-forge pyspark -y
```

### Spark setup with findspark

Local and cluster mode, uncomment the line depending on your particular situation:

```python
import findspark 
# Local Spark
# findspark.init('/home/cloudera/miniconda3/envs/jupyter/lib/python3.7/site-packages/pyspark/') 
# Cloudera cluster Spark
findspark.init(spark_home='/opt/cloudera/parcels/SPARK2-2.3.0.cloudera4-1.cdh5.13.3.p0.611179/lib/spark2/')
```

### Getting PySpark shell

To get a PySpark shell:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('example_app').master('local[*]').getOrCreate()
```

Let’s get existing databases. I assume you are familiar with Spark DataFrame API and its methods:

```python
spark.sql("show databases").show()
```

```text
+------------+
|databaseName|
+------------+
|  __ibis_tmp|
|   analytics|
|         db1|
|     default|
|     fhadoop|
|        juan|
+------------+
```

### From pandas to Spark

First integration is about how to move data from pandas library,  which is Python standard library to perform in-memory data manipulation,  to Spark. First, let’s load a pandas DataFrame. This one is about Air Quality in Madrid (just to satisfy your curiosity, but not important  with regards to moving data from one place to another one). You can download it [here](https://www.kaggle.com/decide-soluciones/air-quality-madrid). Make sure you install pytables to read `hdf5` data.

```python
air_quality_df = pd.read_hdf('data/air_quality/air-quality-madrid/madrid.h5', key='28079008')
air_quality_df.head()
```

```text
                  date    BEN  CH4    CO  ...
2001-07-01 01:00:00  30.65  NaN  6.91  ...
2001-07-01 02:00:00  29.59  NaN  2.59  ...
2001-07-01 03:00:00   4.69  NaN  0.76  ...
2001-07-01 04:00:00   4.46  NaN  0.74  ...
2001-07-01 05:00:00   2.18  NaN  0.57  ...
```

Let’s make some changes to this DataFrame, like resetting datetime index to not lose information when loading into Spark. Datetime will also be transformed to string as Spark has some issues working with dates (related to system locale, timezones, and so on).

```python
air_quality_df.reset_index(inplace=True)
air_quality_df['date'] = air_quality_df['date'].dt.strftime('%Y-%m-%d %H:%M:%S')
```

We can simply load from pandas to Spark with `createDataFrame`:

```python
air_quality_sdf = spark.createDataFrame(air_quality_df)
air_quality_sdf.dtypes
```

Once DataFrame is loaded into Spark (as `air_quality_sdf` here), can be manipulated easily using PySpark methods:

```python
air_quality_sdf.select('date', 'NOx').show(5)
```

```text
+-------------------+------------------+
|               date|               NOx|
+-------------------+------------------+
|2001-07-01 01:00:00|            1017.0|
|2001-07-01 02:00:00|409.20001220703125|
|2001-07-01 03:00:00|143.39999389648438|
|2001-07-01 04:00:00| 149.3000030517578|
|2001-07-01 05:00:00|124.80000305175781|
+-------------------+------------------+
only showing top 5 rows
```

### From Spark to Hive

To persist a Spark DataFrame into HDFS, where it can be queried using default Hadoop SQL engine (Hive), one straightforward strategy (not the only one) is to create a temporal view from that DataFrame:

```python
air_quality_sdf.createOrReplaceTempView("air_quality_sdf")
```

Once the temporal view is created, it can be used from Spark SQL engine to create a real table using `create table as select`. Before creating this table, I will create a new database called `analytics` to store it:

```python
sql_drop_table = """
drop table if exists analytics.pandas_spark_hive
"""

sql_drop_database = """
drop database if exists analytics cascade
"""

sql_create_database = """
create database if not exists analytics
location '/user/cloudera/analytics/'
"""

sql_create_table = """
create table if not exists analytics.pandas_spark_hive
using parquet
as select to_timestamp(date) as date_parsed, *
from air_quality_sdf
"""

print("dropping database...")
result_drop_db = spark.sql(sql_drop_database)

print("creating database...")
result_create_db = spark.sql(sql_create_database)

print("dropping table...")
result_droptable = spark.sql(sql_drop_table)

print("creating table...")
result_create_table = spark.sql(sql_create_table)
```

```text
dropping database...
creating database...
dropping table...
creating table...
```

Can check results using Spark SQL engine, for example to select ozone pollutant concentration over time:

```python
spark.sql("select * from analytics.pandas_spark_hive") \
.select("date_parsed", "O_3").show(5)
```

```text
+-------------------+------------------+
|        date_parsed|               O_3|
+-------------------+------------------+
|2001-07-01 01:00:00| 9.010000228881836|
|2001-07-01 02:00:00| 23.81999969482422|
|2001-07-01 03:00:00|31.059999465942383|
|2001-07-01 04:00:00|23.780000686645508|
|2001-07-01 05:00:00|29.530000686645508|
+-------------------+------------------+
only showing top 5 rows
```

## pyarrow: Apache Arrow

Apache Arrow, is a in-memory columnar data format created to support high performance operations in Big Data environments (it can be seen as  the parquet format in-memory equivalent). It is developed in C++, but  its Python API is amazing as you will be able to see now, but first of all, please install it:

```bash
conda install pyarrow -y
```

In order to establish a native communication with HDFS I will use the interface included in pyarrow. Only requirement is setting an environment variable pointing to the location of `libhdfs`.  Remember we are in a Cloudera environment. In case you are using Horton  will have to find proper location (believe me, it exists).

### Establish connection

```python
import pyarrow as pa
import os
os.environ['ARROW_LIBHDFS_DIR'] = \
'/opt/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/lib64/'
hdfs_interface = pa.hdfs.connect(host='localhost', port=8020, user='cloudera')
```

### List files in HDFS

Let’s list files persisted by Spark before. Remember that those files  has been previously loaded in a pandas DataFrame from a local file and then loaded into a Spark DataFrame. Spark by default works with files partitioned into a lot of `snappy` compressed files. In HDFS path you can identify database name (`analytics`) and table name (`pandas_spark_hive`):

```python
hdfs_interface.ls('/user/cloudera/analytics/pandas_spark_hive/')
```

```text
['/user/cloudera/analytics/pandas_spark_hive/_SUCCESS',
 '/user/cloudera/analytics/pandas_spark_hive/part-00000-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00001-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00002-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00003-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00004-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00005-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00006-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet',
 '/user/cloudera/analytics/pandas_spark_hive/part-00007-b4371c8e-0f5c-4d20-a136-a65e56e97f16-c000.snappy.parquet']
```

### Reading parquet files directly from HDFS

To read parquet files (or a folder full of files representing a table) directly from HDFS, I will use PyArrow HDFS interface created before:

```python
table = hdfs_interface \
.read_parquet('/user/cloudera/analytics/pandas_spark_hive/')
```

### From HDFS to pandas (.parquet example)

Once `parquet` files are read by PyArrow HDFS interface, a Table object is created. We can easily go back to pandas with method `to_pandas`:

```python
table_df = table.to_pandas()
table_df.head()
```

```text
   id          date_parsed                 date    BEN  ...
0   0  2001-06-30 23:00:00  2001-07-01 01:00:00  30.65  ...
1   1  2001-07-01 00:00:00  2001-07-01 02:00:00  29.59  ...
2   2  2001-07-01 01:00:00  2001-07-01 03:00:00   4.69  ...
3   3  2001-07-01 02:00:00  2001-07-01 04:00:00   4.46  ...
4   4  2001-07-01 03:00:00  2001-07-01 05:00:00   2.18  ...
```

And that is basically where we started, closing the cycle Python -> Hadoop -> Python.

### Uploading local files to HDFS

All kind of HDFS operations are supported using PyArrow HDFS interface, for example, uploading a bunch of local files to HDFS:

```python
cwd = Path('./data/')
destination_path = '/user/cloudera/analytics/data/'

for f in cwd.rglob('*.*'):
    print(f'uploading {f.name}')
    with open(str(f), 'rb') as f_upl:
        hdfs_interface.upload(destination_path + f.name, f_upl)
```

```text
uploading sandp500.zip
uploading stations.csv
uploading madrid.h5
uploading diamonds_train.csv
uploading diamonds_test.csv
```

Let’s check if files have been uploaded properly, listing files in destination path:

```python
hdfs_interface.ls(destination_path)
```

```text
['/user/cloudera/analytics/data/diamonds_test.csv',
 '/user/cloudera/analytics/data/diamonds_train.csv',
 '/user/cloudera/analytics/data/madrid.h5',
 '/user/cloudera/analytics/data/sandp500.zip',
 '/user/cloudera/analytics/data/stations.csv']
```

### From HDFS to pandas (.csv example)

For example, a `.csv` file can be directly loaded from HDFS into a pandas DataFrame using `open` method and `read_csv` standard pandas function which is able to get a buffer as input:

```python
diamonds_train = pd.read_csv(hdfs_interface.open('/user/cloudera/analytics/data/diamonds_train.csv'))
diamonds_train.head()
```

```text
   carat        cut color clarity  depth  table  price     x     y     z
0   1.21    Premium     J     VS2   62.4   58.0   4268  6.83  6.79  4.25
1   0.32  Very Good     H     VS2   63.0   57.0    505  4.35  4.38  2.75
2   0.71       Fair     G     VS1   65.5   55.0   2686  5.62  5.53  3.65
3   0.41       Good     D     SI1   63.8   56.0    738  4.68  4.72  3.00
4   1.02      Ideal     G     SI1   60.5   59.0   4882  6.55  6.51  3.95
```

In case you are interested in all methods and possibilities this library has, please visit: [https://arrow.apache.org/docs/python/filesystems.html#hdfs-api](https://arrow.apache.org/docs/python/filesystems.html#hdfs-api)

## python-hdfs: HDFS

Sometimes it is not possible to access `libhdfs` native HDFS library (for example, performing analytics from a computer that is not  part of the cluster). In that case, we can rely on WebHDFS (HDFS service REST API),  it is slower and not suitable for heavy Big Data loads, but an interesting option in case of light workloads. Let’s install a WebHDFS Python API:

```bash
conda install -c conda-forge python-hdfs -y
```

### Establish WebHDFS connection

To establish connection:

```python
from hdfs import InsecureClient
web_hdfs_interface = InsecureClient('http://localhost:50070', user='cloudera')
```

### List files in HDFS

Listing files is similar to using PyArrow interface, just use `list` method and a HDFS path:

```python
web_hdfs_interface.list('/user/cloudera/analytics/data')
```

```text
['diamonds_test.csv',
 'diamonds_train.csv',
 'madrid.h5',
 'sandp500.zip',
 'stations.csv']
```

### Uploading local files to HDFS using WebHDFS

More of the same thing:

```python
cwd = Path('./data/')
destination_path = '/user/cloudera/analytics/data_web_hdfs/'

for f in cwd.rglob('*.*'):
    print(f'uploading {f.name}')
    web_hdfs_interface.upload(destination_path + f.name, 
                              str(f),
                              overwrite=True)
```

```text
uploading sandp500.zip
uploading stations.csv
uploading madrid.h5
uploading diamonds_train.csv
uploading diamonds_test.csv
```

Let’s check the upload is correct:

```python
web_hdfs_interface.list(destination_path)
```

```text
['diamonds_test.csv',
 'diamonds_train.csv',
 'madrid.h5',
 'sandp500.zip',
 'stations.csv']
```

Bigger files can also be handled by HDFS (with some limitations). Those files are from Kaggle [Microsoft Malware Competition](https://www.kaggle.com/c/microsoft-malware-prediction), and weighs a couple of GB each:

```python
web_hdfs_interface.upload(destination_path + 'train.parquet', '/home/cloudera/analytics/29_03_2019/notebooks/data/microsoft/train.pq', overwrite=True);
web_hdfs_interface.upload(destination_path + 'test.parquet', '/home/cloudera/analytics/29_03_2019/notebooks/data/microsoft/test.pq', overwrite=True);
```

### From HDFS to pandas using WebHDFS (.parquet example)

In this case, it is useful using PyArrow `parquet` module and passing a buffer to create a Table object. After, a pandas DataFrame can be easily created from Table object using `to_pandas` method:

```python
from pyarrow import parquet as pq
from io import BytesIO

with web_hdfs_interface.read(destination_path + 'train.parquet') as reader:
    microsoft_train = pq.read_table(BytesIO(reader.read())).to_pandas()
    
microsoft_train.head()
```

```text
                  MachineIdentifier  ProductName EngineVersion  ...
0  0000028988387b115f69f31a3bf04f09  win8defender   1.1.15100.1  ...
1  000007535c3f730efa9ea0b7ef1bd645  win8defender   1.1.14600.4  ...
2  000007905a28d863f6d0d597892cd692  win8defender   1.1.15100.1  ...
3  00000b11598a75ea8ba1beea8459149f  win8defender   1.1.15100.1  ...
4  000014a5f00daa18e76b81417eeb99fc  win8defender   1.1.15100.1  ...
```

## impyla: Hive + Impala SQL

Hive and Impala are two SQL engines for  Hadoop. One is MapReduce based (Hive) and Impala is a more modern and  faster in-memory implementation created and opensourced by Cloudera.  Both engines can be fully leveraged from Python using one of its  multiples APIs. In this case I am going to show you `impyla`, which supports both engines. Let’s install it using conda, and do not forget to install `thrift_sasl` 0.2.1 version (yes, must be this specific version otherwise it will not work):

```bash
conda install impyla thrift_sasl=0.2.1 -y
```

### Establishing connection

```python
from impala.dbapi import connect
from impala.util import as_pandas
```

#### From Hive to pandas

API follow classic ODBC stantard which will probably be familiar to you. `impyla` includes an utility function called `as_pandas` that easily parse results (list of tuples) into a pandas DataFrame. Use  it with caution, it has issues with certain types of data and is not  very efficient with Big Data workloads. Fetching results both ways:

```python
hive_conn = connect(host='localhost', port=10000, database='analytics', auth_mechanism='PLAIN')

with hive_conn.cursor() as c:
    c.execute('SELECT * FROM analytics.pandas_spark_hive LIMIT 100')
    results = c.fetchall()
    
with hive_conn.cursor() as c:
    c.execute('SELECT * FROM analytics.pandas_spark_hive LIMIT 100')
    results_df = as_pandas(c)
```

Raw results are pretty similar to those you can expect using, for example, Python standard `sqlite3` library:

```python
results[:2]
```

```text
[(datetime.datetime(2001, 7, 1, 1, 0),
  '2001-07-01 01:00:00',
  30.649999618530273,
  nan,
  6.909999847412109,
  42.63999938964844,
  nan,
  nan,
  381.29998779296875,
  1017.0,
  9.010000228881836,
  158.89999389648438,
  nan,
  47.5099983215332,
  nan,
  76.05000305175781),
 (datetime.datetime(2001, 7, 1, 2, 0),
  '2001-07-01 02:00:00',
  29.59000015258789,
  nan,
  2.5899999141693115,
  50.36000061035156,
  nan,
  nan,
  209.5,
  409.20001220703125,
  23.81999969482422,
  104.80000305175781,
  nan,
  20.950000762939453,
  nan,
  84.9000015258789)]
```

And its pandas DataFrame version:

```python
results_df.head()
```

```text
   pandas_spark_hive.id  pandas_spark_hive.date_parsed  pandas_spark_hive.date  ...
0   2001-07-01 01:00:00            2001-07-01 01:00:00                     ...
1   2001-07-01 02:00:00            2001-07-01 02:00:00                     ...
2   2001-07-01 03:00:00            2001-07-01 03:00:00                     ...
3   2001-07-01 04:00:00            2001-07-01 04:00:00                     ...
4   2001-07-01 05:00:00            2001-07-01 05:00:00                     ...
```

#### From Impala to pandas

Working with Impala follows the same pattern as Hive, just make sure you connect to correct port, default is 21050 in this case:

```python
impala_conn = connect(host='localhost', port=21050)

with impala_conn.cursor() as c:
    c.execute('show databases')
    result_df = as_pandas(c)

result_df
```

```text
              name                                  comment
0       __ibis_tmp
1  _impala_builtins  System database for Impala builtin functions
2        analytics
3              db1
4          default                       Default Hive database
5          fhadoop
6             juan
```

## Ibis Framework: HDFS + Impala

Another alternative is Ibis Framework, a high level API to a relatively vast collection of datasources, including HDFS and Impala. It is build around the idea of using Python objects and  methods to perform actions over those sources. Let’s install it the same  way as the rest of libraries:

```bash
conda install ibis-framework -y
```

Let’s create both a HDFS and Impala interfaces (impala needs an hdfs interface object in Ibis):

```python
import ibis

hdfs_ibis = ibis.hdfs_connect(host='localhost', port=50070)
impala_ibis = ibis.impala.connect(host='localhost', 
                                  port=21050, 
                                  hdfs_client=hdfs_ibis, 
                                  user='cloudera')
```

Once interfaces are created, actions can be performed calling methods, no need to write more SQL.  If you are familiar to ORMs (Object Relational Mappers), this is not  exactly the same, but the underlying idea is pretty similar.

```python
impala_ibis.invalidate_metadata()
impala_ibis.list_databases()
```

```text
['__ibis_tmp',
 '_impala_builtins',
 'analytics',
 'db1',
 'default',
 'fhadoop',
 'juan']
```

### From Impala to pandas

Ibis natively works over pandas, so there is no need to perform a conversion. Reading a table returns a pandas DataFrame object:

```python
table = impala_ibis.table('pandas_spark_hive', 
                          database='analytics')
table_df = table.execute()
```

`table_df` is a pandas `DataFrame` object.

### From pandas to Impala

Going from pandas to Impala can be made using Ibis selecting the  database using Impala interface, setting up permissions (depending on  your cluster setup) and using the method `create`, passing a pandas DataFrame object as an argument:

```python
analytics_db = impala_ibis.database('analytics')
hdfs_ibis.chmod('/user/cloudera/analytics', '777')
analytics_db.create_table(table_name='diamonds', 
                          obj=pd.read_csv('data/diamonds/diamonds_train.csv'),
                          force=True)
```

Reading the newly created table back result in:

```python
analytics_db.table('diamonds').execute().head(5)
```

```text
   carat        cut color clarity  depth  table  price     x     y     z
0   1.21    Premium     J     VS2   62.4   58.0   4268  6.83  6.79  4.25
1   0.32  Very Good     H     VS2   63.0   57.0    505  4.35  4.38  2.75
2   0.71       Fair     G     VS1   65.5   55.0   2686  5.62  5.53  3.65
3   0.41       Good     D     SI1   63.8   56.0    738  4.68  4.72  3.00
4   1.02      Ideal     G     SI1   60.5   59.0   4882  6.55  6.51  3.95
```

### Final words

Hope you liked this tutorial. Using those methods you can vanish the wall between local computing using Python and Hadoop distributed computing framework. In case you have any questions about the concepts  explained here, please write a comment below or send me an email.