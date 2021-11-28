#  hdfs (Hadoop file system)
HDFS seems like a big data dump

input data into HDFS using

 - GUI like Ambari
 - CLI
 - HTTPS proxies
 - Java interface
 - NFS gateway (mount it like a disk)

You can manipulate HDFS similar to your own local file system. For example, change ownership permissions

```
hdfs dfs -chmod -R 777 hdfs:///user/hive/warehouse/db_name.db/tbl_name
```

A similar command will show you the partitions of a particular table and you can see the size of the partition. Optimal size is from a 100MB up to a couple of GB. (I've heard 500MB at work)

```
hdfs dfs -ls hdfs:///user/hive/warehouse/db_name.db/tbl_name/partition_name

hdfs dfs -cat hdfs:///user/akhaili/pltv/pred_test_holdout.csv | head

```

# map/reduce
**mapper**: function applied to each row to return a key-value pair
**shuffle and sort**: arrange all values for a key together and distribute them across the cluster (uses smt like modulo operator to send them to reducers). Also sorts the values by key. 
**reducer**: aggregate values per key into one value

You can use more than one map/reduce step and you can skip steps (eg map, reduce then reduce again). An example included using an additional reduce step to sort by the values instead of keys (reversing key-value)

Uses Java internally, but you can stream to Python using stdin and stdout the results. You lose types this way, everything is a string. Or you can use cross-platform formats like JSON

#  pig
Pig is an SQL-like language (but procedural) for data transformation and storage. Running on Tez it is fast (it is an optimizer), but Spark has something similar (DAG)

#  spark
DAG optimizes spark jobs (directed acyclic graph)

Spark components: Spark streaming, MLLib, GraphX, Spark SQL

# Hive vs Impala
Impala runs all the time unlike Hive, which makes it faster for BI-style queries

# NoSQL
When you need high transaction rates and horizontal scaling. Except HBASE, these are systems outside of Hadoop, but can connect to it. 

## hbase
NoSQL database capable of CRUD operations. Key - values store, ie row-columns. Implemented in Hadoop, so no new servers required. 

Open source implementation of Google's BigTable

Main advantage is sparsity. Every row has several column families, but the columns inside the families can be arbitrary (and empty)

## Cassandra
Built for availability: has no master node (single point of failure)

Has a limited querying language, has to be on primary key, no joins (noSQL!)

CAP theorem says you can only have two of three: consistency, availability and partition-tolerance. Partition-tolerance is required for big data (database is easily split up across servers). Cassandra chooses availability.

Cassandra replicates data across nodes which may be out of sync (this means low consistency). When querying, you can specify the level of consistency ("three nodes have to agree on the data")

## MongoDB
More flexible data model. JSON structure, and you can have set multiple keys (ie, indexes), even keys involving more than one field

Has a primary node (availability!) that gets re-elected among secondary servers in case of failure

Scaling is done by sharding: using an index to split your data among replica sets (ie sets of primary and secondary nodes)

Can replace Hadoop in many ways. Eg can execute Spark jobs directly

Can do simple transformations using own querying language in Javascript

# query engines

## Drill
Write SQL queries for various non-relational data sources: MongoDB, HBase, flat JSON files, Parquet files

Based on Google Dremel

## Phoenix
SQL querying for HBase

## Presto
Write SQL queries for different databases, similar to Drill. Has a Cassandra connector unlike Drill. Developed by Facebook.

# tools for managing your cluster

## Yarn
Manages cluster resources. Before this was done by MapReduce, but the introduction of Yarn enabled mapreduce alternatives like Spark and Tez.

Whereas HDFS is the storage layer, Yarn is the compute layer. Mapreduce, Spark, etc are built on top of Yarn. Yarn splits the computation across the cluster, whereas HDFS does so for stored data.

You can have more than one resource manager, ie have some redundancy using Zookeper.

Yarn can use different scheduling options, ie how to queue the applications. 

## TEZ
Alternative to mapreduce, uses directed acyclic graphs (DAGs). Can run Pig, Hive or even mapreduce jobs on it. Spark also uses DAGs

It just makes things faster, as a user you don't have a direct contact with it unless you are creating applications like Spark

## Mesos
A resource manager, more general than YARN. A general container managment system. Created by Twitter, not a direct part of Hadoop but can replace YARN

You can also have Mesos distribute resources to YARN *and* other uses, meaning you can use your cluster for both Hadoop and other applications. For this you need the Myriad package. It's all about optimizing your resource allocation. 

Works different than YARN. YARN takes an application and finds resources for it. Mesos makes "offers" of resources which the application can refuse. You also specify the scheduling algo. Scales better than YARN

Mesos can handle processes that last "forever" or very short processes, not just long analytical processes. 

Docker / Kubernetes are an alternative to Mesos, but Mesos works well with Hadoop related technologies like Spark. But has some limitations compared to Yarn, so best to combine them. 

## Zookeper
Keeps track of your system: master node, tasks... Ensures high availability.

## Oozie
System for running and scheduling Hadoop tasks

Workflow is a multi stage Hadoop job. It is a DAG specified through xml where you specify each node (job). Jobs that dont depend on each can be run in parallel using fork (split) and join (wait for for to finish). The job.properties file will define variables the workflow needs.

Oozie coordinators schedule workflow executions and can wait for some data to become available. Oozie bundles are collections of coordinators that can be managed together.

# streaming data to the cluster
The problem of streaming data and processing it on the fly

## Kafka
General purpose (not just Hadoop)  publish/subscribe messaging system. Consumers can subscribe to topics.

## Flume
Made to be used with Hadoop, connects easily to HDFS and Hbase. A buffer between data sources and your cluster, because HDFS does not work with spiky data.

Organized into "agents"
 - source: where the data is coming from. Eg HTTP, Kafka, Thrift...
 - channel: how the data is transferred from source to sink (memory or files)
 - sink: where the data is going. Connected to only one channel. Can be organized into groups

Flume does not keep data after it's handled to a sink. This is different from Kafka that does keep the data for a specified amount of time.

Using Avro you can connect Flume agents to one anothers.

# stream processing
If we have stream data, why would we process it in batch mode? Stream processing deals with data as it comes in. Also allows you to react real-time (eg to user behavior or IoT devices)

We can look at data being processed from a 

## spark streaming
Split data into batches for a given time increment (eg 1 second) and processes it.

Works on DStreams (Discretized Streams) which generate RDDs for each time step. They can be acted on in a similar way as RDDs and you can access RDDs underlying them. I guess they abstract the fact that a streaming job never stops, but I don't get why you can't just use RDDs.

You can maintain state with DStreams, eg a running total over some keys. Use windowing (eg last hour) for that. Slide interval = how often you compute the window calculation.

A newer alternative to DStreams is Structured Streaming which uses DataSets (similar to DataFrames). DataSets have explicit type information. In the streaming context, these are tables that never end. Since MLLib is moving towards DataSets, this means that machine learning could be done with streaming data.

Also, Structured Streaming can process event data, not just micro-batches (N sec intervals)

## Apache Storm
Alternative to Spark Streaming, proceses individual events, not micro-batches.

Terminology:

 - a stream consists of tuples of data
 - spouts are sources of data (eg Kafka)
 - bolts process the streamed data
 - topology is a graph of spouts and bolts that process your stream

Two APIs:

 - Storm Core is a lower level API that guarantees "at least once" processing, meaning a particual stream could be processed twice
 - Trident guarantess "exactly once" and is higher level, but can only do micro-batches (same as Spark Streaming)

## Flink
Can process event-by-event, but guarantees "only once" processing, unlike Storm. It's also faster and offers a high-level API

Can process data based on event times, not when the data was received (ie, data arriving out of order)

Has an ecosystem similar to Spark (machine learning, SQL, graph data), but is not as mature




