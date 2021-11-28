# literature

 - https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html
 - https://www.udemy.com/the-ultimate-hands-on-hadoop-tame-your-big-data/

# installing locally

 - download from Apache: https://spark.apache.org/downloads.html
 - set ~/.bash_profile like below. For me, setting the JAVA_HOME directory was the thing that made or broke the thing

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export SPARK_HOME=~/Downloads/spark
export PATH=$SPARK_HOME/bin:$PATH
export PYSPARK_PYTHON=python3
```

# main ideas

## resilient distributed datasets (RDDs)
> Distributed collections that are automatically parallelized across the cluster.

> You can usually change the number of partitions in most of the Spark functions (that is the numTasks param)

## Spark application
> Every application consists of a driver program that launches parallel operations on the cluster. This driver could contain your applications main function, or could be the interactive shell.

> Driver accesses Spark through a SparkContext object which is created automatically in the shell but needs to be initialized if running a script/application.

## optimization: logical and physical plan

Code is turned first into a logical plan (order of execution, doing tasks in parallel), then physical plan (distributing the task on the cluster). Logical plan goes through these stages:

- unresolved
- resolved (done by the **analyzer**: do the tables exist? Do they have the columns?
- optimized (done by **Catalyst Optimizer**): pushing down predicates, etc

# practical stuff

## running a Python-Spark script/app
The code below will initialize Spark dependencies.

```python
import pyspark

conf = pyspark.SparkConf()
conf.setMaster('local')  # set cluster URL
conf.setAppName('name')
sc = SparkContext(conf=conf)
sc.stop()  # stops Spark from inside the application

# from Spark 2.0 you create a SparkSession that contains both SQL and rdd
# if 'name' exists, will continue where left off
spark = SparkSession.builder.appName('name').getOrCreate()
spark.sparkContext  # contains context

# run script in CLI
# spark-submit my_script.py
```

# reading data

```python
data = sc.textFile('/mydir/path_to_text.txt')  # or s3 or hdfs ('hdfs://file.txt')
data = sc.parallelize([1, 2, 3])
# from Hive
hc = HiveContext(sc)
data = hc.sql('SELECT * FROM table;')

# also from JDBC, Cassandra, HBase, Elasticsearch, JSON, csv, ...
```

## creating pair (key: value) RDDs
```python
# need to return a tuple
pairs = lines.map(lambda string: (string.split(' '), string))
```

# partitioning
> In a distributed program, you need to optimize (minize) communications between the nodes. Laying out data in a way that does this will greatly speed up your data processing in Spark. Spark allows you to say which "rows" should go together on a cluster.

> This only makes sense if you are using a dataset more than once and is available for paired RRDs only (key-value). You partition by keys so that you know where (on which cluster) each key is. E.g., when doing a join, this would mean that the partitioned RDD would not have its members change locations (ie, travel through the network). Only the non-partitioned one would.

> Range-partitioning uses key values in a straightforward way (eg, dates). Hash-partitioning first applies a hashing algorithm, then bins the hashes. It is useful when you do not have an obvious partitioning key or when you want to distribute data evenly.

> Operations that benefit from partitioning are all that shuffle (move around) data across the network. Joins, grouping, reduceByKey, etc. Single RDD operations (eg, reduceByKey) benefit from the fact that all keys of the same value can be aggregated on a single cluster, whereas operations on multiple RDDs benefit from the fact that the partitioned RDD stays in place (if both are partitioned, first one will be used). If both RDDs have the same partitioning (eg if one is a result of mapValues on the other), then processing will be especially fast.

> Functions that break partitioning are the ones that cannot guarantee the key will be the same, eg `map()`. It is better then to use their keyed equivalents if possible like `mapValues()`.

## partitionBy()
Use this method to partition data by key.

```python
part = rdd.partitionBy(100)  # partition into N groups, N is usually the nr of nodes or more
part.persist()  # partitionBy returns a new RDD, need to persist or this would repeat on each action
```

## custom partitioner
Instead of the provided hash- or range- partitioning, you can create your own. Eg, you can hash the domain of the url instead of the whole url.

```
import urlparse
def hash_domain(url):
    return hash(urlparse.urlparse(url).netloc)
rdd.partitionBy(20, hash_domain)  # pass the hashing func
```

## repartitioning data
```
rdd.repartition(num)  # splits the RDD into N partitions, expensive
rdd.coalesce(num)  # only when num is lower than current nr of partitions, optimized
rdd.getNumPartitions()
```

# transformations and actions

## actions
Compute a result based on the RDD, where the result is not itself an RDD.

```python
# getting data out
rdd.first()  # returns first elem
rdd.take(5)  # returns 5 elems
rdd.takeOrdered(5, key=sortingFunc)  # sorts
rdd.collect()  # returns all elems
rdd.top(2)  # N top elems based on ordering

# aggregation
rdd.count()  # counts nr of elems
rdd.countByValue()  # counts nr of each value

# these take func as arg
rdd.reduce(func)
rdd.fold(identity_val, func)  # takes an identity elem (eg zero for summation)
rdd.aggregate(identity_val, func1, func2)  # return type != input type (or is)
rdd.foreach(func)  # do smt with each elem, but no return
```

### aggregate vs reduce
Aggregate is more flexible than reduce.

```
# aggregate can return a diff rdd type than input
# an example of a running average
rdd.aggregate((0, 0),  # the initial value
              lambda acc, value: (acc[0] + value, acc[1] + 1),  # aggr accumulator and single value
              lamda acc1, acc2: (acc1[0] + acc2[0], acc1[1] + acc2[1]))  # aggr two accumulators (since they are calc on diff nodes)
```

### actions on pair RDDs
```
rdd.countByKey()
rdd.collectAsMap()  # returns tuples as dicts (1, 2) => {1: 2}
rdd.lookup(key)     # return value for key
```

## transformations
Transformations return RDDs that are a transformation of the passed one. They are computed in a *lazy* fashion: only when they are used in an action. That way you save space, storing only the final transformation of the RDD.

When an action is done, the RDD is recomputed: its initial state is not stored. If you want to save a particular state of it, use `rdd.persist()`.

Because of this laziness property, it is best to think of RDDs as *instructions for computing data* (including reading it and transforming), not actual data.

```python
# these require a function
rdd.filter(lambda elem: elem > 5)  # func needs to return T/F
rdd.map(func)
rdd.flatMap(func)  # mapping func can return iterators which are flattened
rdd.groupBy(func)  # func returns the key used for grouping

# set operations
rdd.union(rdd_2)  # keeps duplicates
rdd.intersection(rdd_2)  # removes duplicates
rdd.distinct()  # returns unique elems
rdd.subtract(rdd_2)
rdd.cartesian(rdd_2)

# other
rdd.sample(withReplacement=False, fraction=0.1, seed=10)
```

### narrow vs wide transformations
Narrow transformations don't reshuffle data -> each input partition contributes to only one output partition. For example, a filter (`where`).

Wide transformations involve shuffling of the data, which makes them slower. An example would be ordering.

### transformations on pair RDDs
These are specific to pair RDDs (2-sized tuples). Other transformations also work on pair RDDs.

```
# map reduce (commonly you first create a new rdd, then map it to smt useful)
rdd.reduceByKey(func)  # returns {(key, result), ...}
rdd.foldByKey(identity_val, func)
rdd.mapValues(func)  # apply func to value keeping key the same
rdd.flatMapValues(func)  # func can return iterator which will be flattened but with the key
rdd.combineByKey(createCombiner,  # calls with value, return empty accumulator
                 mergeValue,      # calls with acc, val, returns merge
                 mergeCombiners)  # calls with acc1, acc2, returns merge

# grouping
rdd.groupByKey()  # returns {(key, [elem1, elem2]), ...}
rdd.cogroup(rdd2) # combines RDDs with same keys: (key, ([values], [values]))

# getting values
rdd.keys()  # returns an RDD of keys
rdd.values()

# sorting
rdd.sortByKey(ascending=True, keyfunc)  # can provide own sort func
rdd.sortBy(sort_func)  # not sure if exists: saw it on Udemy, can't find online

# SQL-like operations
rdd.join(rdd2)  # also leftOuterJoin and rightOuterJoin
rdd.subtractByKey(rdd2)  # removes elems if key in rdd2
rdd.cogroup(rdd2)  # returns key, (values_1, values_2) where values are lists from the respective RDDs
```

### persisting RDDs
Persist transformations so that they are not recomputed every time you conduct an action on the RDD. `rdd.persist()` does not force a computation, you still have to conduct an action. You can choose the persistence level (memory vs disk, serialized vs unserialized).

```
import pyspark 
rdd.persist(pyspark.StorageLevel.DISK_ONLY)
rdd.unpersist()
```

### computationally complex transformations
> shuffling of the dataset is expensive
> 
> - `rdd.distinct()`
> - `rdd.intersect(rdd_2)`
> - `rdd.subtract(rdd_2)`

### issue: passing functions together with object
Watch out when a function is a member of an object

```
class My_class():
    def __init__(self, query):
        self.query = query

    # WRONG: self will get passed together with the func
    #        will need to get serialized/pickled, which
    #        could fail
    def get_matches(rdd):
        return rdd.filter(lambda x: self.query in x)

    # RIGHT
    def get_matches(rdd):
        query = self.query
        return rdd.filter(lambda x: query in x)
```

## explain
Returns the execution plan for transformations

```
df.explain()
```

# dataframes
> A replacement for RDDs:
> 
> - contains Row objects
> - can run SQL queries, but also run functions which are more programmatical
> - has a schema (efficient storage)
> - persistence still applies

## creating dataframes

DataFrames are a collection of records that satisfy a particular schema. You can create them by reading files or querying existing data, or by creating rows directly using `pyspark.sql.Row`.

```python
# reading from Hadoop
spark.sql('SELECT * FROM table')

# reading from a file
data = spark.read.json(data_file)

# Row constructor, list or rdd of Rows can be made into a df
row = Row(1, 2)
spark.createDataFrame(list_of_rows)  # also from pandas df

# create your own schema (instead of inferring it)
import pyspark.sql.types as pt
manual_schema = pt.StructType(
    # name, type and if NULLable
	pt.StructField('id', pt.IntegerType(), True)
)
spark.read.format('json').schema(manual_schema).load('file.json')
spark.createDataFrame(list_of_rows, manual_schema)
```

## inspecting dataframes

```python
df.printSchema()  # similar to describe
df.schema  # same, but returns StructType
```

## mixing SQL and spark df code

```python
# will create temporary table that you can query using SQL
data.createOrReplaceTempView('table_name')
```

## columns

```python
# refers to a column, has meaning only in the context of a df, resolved by analyzer
col('name')

# more directly specifies a column that belongs to df
df.col('name')
```

## dataframe operations

```python
import pyspark.sql.functions as sf

# select takes strings or column type
df.select('colname')
df.select(sf.col('colname'))
# and can do aggregations
df.selectExpr('avg(colname)')

df.where(df('colname' > 200))  # or use SQL: df.where('colname > 200')
df.groupBy(df('colname)')).mean()
df.rdd()  # returns an rdd
df.join(df2, 'key')
df.distinct()

# sampling
df.sample(fraction=0.1, withReplacement=False, seed=1)
dataframes = df.randomSplit([0.2, 0.8], seed=1)

df.sort(sf.col('column_name').asc())  # sort ascending
df.sortWithinPartitions('column_name')  # for optimization
```

### tip: filters (and other) saved into variables

Seems quite useful if you want to create filter programmatically. Otherwise, just use SQL expressions.

```python
import pyspark.sql.functions as sf

# filters saved as variables
filter1 = sf.col('StockCode') == "DOT"
filter2 = sf.col('UnitPrice') > 600
df.where(filter1 & filter2).show()

# computation saved as variable
value = sf.pow(sf.col('Quantity') * sf.col('UnitPrice'), 2) + 5
df.select('Quantity', value)
```

### statistical functions

Some interesting ones I found

```python
df.describe()  # shows descriptive stats
df.crosstab('col1', 'col2')
```



## partitioning

```python
df.rdd.getNumPartitions()

# partitioning will incur a full shuffle
df.repartition(5)  # or with a colname; or both (num, colname)

# coalescing will try to combine partitions3
df.repartition(5, 'country').coalesce(2)
```



## user defined functions (UDFs)

```python
from pyspark.slq.types import IntegerType
hiveContext.registerFunction('square', lambda x: x * x, IntegerType())
df = hiveContext.sql('SELECT square("numeric column" FROM table')
```

# configuring spark

```
# number of partitions during a shuffle, default is 200
# not sure when you would want to lower / increase this
spark.conf.set('spark.sql.shuffle.partitions', '200')
```

# performance, unsorted

Using SQL in `spark.sql` or  `pyspark.sql.functions.expr` has the same performance as using DataFrame objects and Python code.

Filter / where operations are optimized in the logical plan and it doesn't matter which order you use and whether you combine them or separate.

```python
df.where('x = 1').where('y = 1')
# is the same as
df.where('x = 1 AND y = 1')
```

# unsorted

Need to use `&` instead of `and` in where clause. Also need to use parenthesis since `&` takes precedence over `==`.

```python
df.where(
    res.partner_id.isin(test_pids)
    & (res.created > start_date)
    ).select('created', 'partner_id'
    ).head(n=4)
```



# spark training B.com

architecture of a Spark program (or Hive, or mapreduce in general I think)

 - application
 - job
 - stage = wide transformations (data needs to be shuffled: eg sort, groupByKey)
 - task = narrow transformations (on one node)

You want to avoid shuffling by partitioning the data, but this may lead to skew - most data on a few nodes bcz some keys have a lot of values.

## broadcast hash join
Copy a smaller table to each worker node so that it does not have to be shuffled. Reduces IO.

`spark.sql.autoBroadcastJoinThreshold` parameter or via `pyspark.sql.functions.broadcast` function:

```python
from pyspark.sql.functions import broadcast
table = broadcast(spark.sql('SELECT * FROM table'))
table.createOrReplaceTempView('alias')  # alias for the table
```

## execution plans
In Spark these are called "DAGs"

## skew
Most data is sent to a few nodes and they take the longest to run.

Default partition in Spark is by file (one file per executor).

Fix skew by using partition by column and number N. This means that each key from the column will be split into N parts, and the executors will be filled with one value from the column until you "run out of" that key. Then the next key, etc.

## source (usually Hive) table properties
How the table was created influences initial steps. Eg., the table might be partitioned by a column, or it might have sort columns. Depends on the table type (orc, parquet, ...).

## partitioning

 - you should aim for cca 20MB per executor. It depends on the complexity of the job, the specs of the system, etc. Repartition to more / less if you have too much / too little data

## caching (...)

 - `.cache()` will persist data on the executors and keep the executors alive for 45mins (that is the default I think). This also means you will keep others from using those executors

## main idea
... for optimizing Spark is to obtain good paralelization. A "small but not too small" number of executors which finish the job in a reasonable amount of time and where each of them takes approximately the same amount of time to finish. Also, don't use Spark for small data!

## memory in Spark executor
overhead + processing + storage (eg caching)

You also have a Yarn memory overhead whose size you can change.

If you do a lot of computation, lower the storage and raise overhead. If you do a lot of caching, raise the storage.

## changing the configuration
In Jupyter (or pyspark CLI) you need to stop the current context, setup the config and start a new one

## using queues
Each team has a queue, you can set which queue you use with config params or CLI params

## shuffle spill
If a job would require more memory as it progresses (eg if you explode the data that you put in an executor), then the executor might run out of memory at some point. This is shuffle spill. If it does, it will write data to disk which will slow down execution. The solution is to repartition using a bigger number of executors so that they don't run out of memory.

## long jobs
For long jobs, the whole cluster might fail and you lose everything.

Solution is to write intermediate results to a table which you try to read at the beginning.

## dynamic vs static allocation
This has more to do with the efficiency of the whole cluster than your own jobs. Dynamic Allocation (of Executors) (aka Elastic Scaling) is a Spark feature that allows for adding or removing Spark executors dynamically to match the workload.

## improving windowing functions
Window functions need to work within the partition, meaning you can't repartition keys with a lot of values across the cluster in case of high skew.

Solution (when possible) is to partition within a partition, eg partition by day within another key (eg country), window within the subpartion, then within the partition. Eg, you might be calculating the cumulative sum of bookings per country ordered by datetime of creation. First you calculate the cumulative sum per day, then per booking ordered by time.

## speed up long joins
(from Tayfun) It's better to do an initial large join in a single query instead of joining separate Spark DFs. Downstream you can group by.

## from Spark ad-hoc training
Repartition so that Spark will equally distribute data; use repartiion(1000) since usually you get no more than 512 executors (is that for Booking, a default or what?)

Cache to store in memory or disk, this lasts for 30 mins

createOrReplaceTempView holds the data for longer. It also allows you to store data to Hive by using the temp table view. Or cleaner use `pivot.write.saveAsTable('<your_schema>.pivot_table_spark', mode='overwrite', format='orc')`

`spark.sql.functions.expr` to call a SQL expression directly

better options than `df.groupby('country').agg({'nr_rooms': 'sum'})`:


```python
# using Spark SQL functions
df4 = (temp_view
 .groupBy(['country'])
 .agg(
       sf.sum('nr_rooms').alias('total_rooms')
      ,sf.count('hotel_id').alias('num_hotels')
    )
)
# using SQL expressions
df4 = (temp_view
 .groupBy(['country'])
 .agg(
       sf.expr('SUM(nr_rooms) AS total_rooms')
      ,sf.expr('COUNT(hotel_id) AS num_hotels')
    )
)

```
