# hive CLI

## hiverc
> Hive looks for a .hiverc file in your HOME dir that it will run before each session. Also, you can run another file like this with the `hive -i path/to/file` option.

## one liners
Use this flag to exit Hive CLI immediately and return the results to stdoutput

```bash
hive -e "SELECT * from table LIMIT 3"

# silent mode (removes "time taken" line)
hive -e -S "SELECT * from table LIMIT 3"

# save results to file
hive -e "SELECT * from table LIMIT 3" > results.txt

# find property name you are searching for
hive -e "set" | grep prop_name
```

## run script

```bash
hive -f /path/to/script.hql

# from inside Hive session
source /path/to/script.hql;
```

## variables in Hive
> Hive offers several namespaces where you can define variables.

> hivevar: user-defined variables
> hiveconf: configuration properties
> system: config defined by Java
> env(read-only): shell environment vars

> hivevar is the default option! Others can be accessed by ${hiveconf:var_name}

### defining vars before running Hive
Using these options you can define variables to use in Hive scripts.

```bash
# use this to run a Hive session
hive --define x=1  # ${x} will be replaced with 1

# or this to run the script
hive --define x=1 my_script.hql
```

### defining vars from inside the session
Using these options you can define variables while running a Hive session

```bash
set env:HOME;   # displays the variable value, ie /home/user
set hivevar:x=1 # sets ${x} to 1
set x=1         # also sets ${x} to 1
```

### calling variables (variable substitution)
```sql
SET a=1;
SELECT ${hivevar:a};  -- outputs var a from namespace hivevar 
```

# configuration

# compression
```sql
-- compresses output, I guess for faster network downloads
SET hive.exec.compress.output=true;

-- deprecated (from 2.7.4?), use mapreduce.map.output.compress
-- compression btw map and reduce, lowers network overhead
SET mapred.compress.map.output=true;

-- set codec, snappy is fastest but lowest compression (others: gzip, bzip2)
SET mapred.map.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;

-- as far as I understand, this converts joins btw tables into a join btw an in-memory map and table. (not 100% sure)
SET hive.auto.convert.join=true;
-- apparently you also need another config
set hive.auto.convert.join.noconditionaltask.size = 10000;

-- substitution of vars?
SET hive.variable.substitute=true;

-- when possible, execute tasks in parallel (default is true)
SET hive.excec.parallel=true;
```

# data types

## collections

### STRUCT
Something similar to a Pyton dict / JS object, but you define types beforehand. The fields are also predefined and are accessed using `name.prop`.

```sql
CREATE TABLE struct_demo
(
  id BIGINT,
  name STRING,
-- A STRUCT as a top-level column. Demonstrates how the table ID column
-- and the ID field within the STRUCT can coexist without a name conflict.
  employee_info STRUCT < employer: STRING, id: BIGINT, address: STRING >,
-- A STRUCT where one of the fields is another STRUCT.
  current_address STRUCT < street_address: STRUCT <street_number: INT, street_name: STRING, street_type: STRING>, country: STRING, postal_code: STRING >
)
STORED AS PARQUET;
```

### MAP
Key-value tuples, values accessed using `name['key']`. Unlike STRUCT, with MAP you can have different keys for each row. You just need to specify which data types for key and value.

```sql
create TABLE map_demo
(
  country_id BIGINT,

  metrics MAP <STRING, BIGINT>,
-- MAP whose value part is an ARRAY.
  notables MAP <STRING, ARRAY <STRING>>,
-- MAP that is a field within a STRUCT.
-- (The STRUCT is inside another ARRAY, because it is rare
-- for a STRUCT to be a top-level column.)
-- For example, city #1 might have points of interest with key 'Zoo',
-- representing an array of 3 different zoos.
-- City #2 might have completely different kinds of points of interest.
-- Because the set of field names is potentially large, and most entries could be blank,
-- a MAP makes more sense than a STRUCT to represent such a sparse data structure.
  cities ARRAY < STRUCT <
    name: STRING,
    points_of_interest: MAP <STRING, ARRAY <STRING>>
  >>
)
STORED AS PARQUET;
```

### ARRAY
Ordered sequence of the same type.

```sql
CREATE TABLE array_demo
(
  id BIGINT,
  name STRING,
-- An ARRAY of scalar type as a top-level column.
  pets ARRAY <STRING>,
-- An ARRAY with elements of complex type (STRUCT).
  places_lived ARRAY < STRUCT <
    place: STRING,
    start_year: INT
  >>
)
STORED AS PARQUET;
```

# queries

## optimization

### optimize joins with "streamtable" or table order
Lets consider we have a table named 'foo' with 1.5 Billion+ records which we are joining with another table called 'bar' with 100 records.

Since Hive streams right-most table(bar) and buffer(in-memory) other tables(foo) before performing map-side/reduce-side join. Hence, if you buffer 1.5 Billion+ records, your join query will fail as buffering 1.5 Billion records will definitely results in Java-Heap space exception.

```
/*So, to overcome this limitation and free the user to remember the order of joining tables based on their record-size, Hive provides a key-word which tells Hive Analyzer to stream table foo.
*/

-- unoptimized query
select foo.a,foo.b,bar.c
from foo
join bar
  on foo.a=bar.a;

-- optimized
select /*+ STREAMTABLE(foo) */
  foo.a, foo.b, bar.c
from foo
join bar
  on foo.a=bar.a;
```

# free text, will turn into chunks later

```sql
explain [query]  -- will show you the query plan

```

stage 0 outputs text
stage 1 is the transformation

with GROUP BY, the mapper will already perform partial aggregation to lower the amount of files sent over the network (reduce operator `mode: mergepartial` vs `mode: hash` which is the full merge)

The map stage organizes data into keys and their values. These are then sent to reducers which aggregate the values for each key into one value. Which reducer they get sent to is calculated using the modulo operator on the key `key % nr_of_reducers`.

each mapper works with a file. Bigger files work better.

results of reduce get written to the filesystem, results of map get spread across the network

Joins are done like this: the data is presorted at the mapping stage. Since it is sorted, the reducer can go across keys of one table and add the values for the same key from the other table, until the key changes (bcz of the sorting). For this, you should signify to Hive which table is the bigger one.

 - put the biggest table as the last one
 - (if you do outer joins) use `/*+ STREAMTABLE(table_alias) */`

But even if you do this, you need to make sure that the streamtable has all the keys that joins will be done on. If that is not possible, it is preferrable to do subquery joins on the smaller tables.

# performance
intuitions:

 - if you look at data more than once, you can improve performance
 - MapReduce takes time for:
     + input / output
     + small jobs (because starting mappers/reducers costs resources)
     + things that are bigger than you need (obvious)

temporary tables (if same name as actual table, name will point to temp)

`set hive.exec.parallel=TRUE`, set it in your dotfiles so it is run always. But it's default already on our cluster.

It's best to filter (`WHERE`) before joining. This is done automatically for inner joins, but it's more complicated for outer joins. You can check these with `EXPLAIN`. The null table (the one that can have nulls) is usually filtered, but the other one you should filter in a subquery. Also, subquerying will select only a few columns (usually you will have something under `SELECT`) which will reduce IO.

Streamtable: tell Hive which table is the big one when doing joins. Streamtable is read row-by-row whereas the other ones are in memory - as much as it can fit. If table is bigger than available memory, then you need to fetch the rest of the data when you're done. This switching is what makes it better to use small tables in memory (less IO) vs big ones.

 - but if you join multiple tables on different keys you will lose the streamtable hint (since these multiple joins are separate mapreduce jobs)
 - you can give the hint for each subquery

Each mapper / reducer has around 4GB and needs some for the computation.

`LIMIT` happens at the end of things so usually it does not help you speed up the query (only the last fetch). Only works with `SELECT *`

Use `DISTRIBUTE BY` (distribute same keys to same machine) and `SORT BY` (order inside the cluster) to put keys together on individual machines

 - `SORT BY` very useful when using `RANK` (the B.com function). (`RANK` will count incrementally until the value changes, eg 'marko marko marko parko parko' => `1 2 3 1 2`)
 - the Hive `RANK` function takes an `OVER` clause and takes no arguments. Here `PARTITION BY` = `DISTRIBUTE BY` and `ORDER BY` = `SORT BY`

# programming tricks

## temporary macros
`CREATE TEMPORARY MACRO macroname(inputs) output;`. Eg good for long case statements

```sql
CREATE TEMPORARY MACRO selection(N int)
    CASE
        WHEN 0 THEN 'zero'
        WHEN 1 THEN 'one'
        ELSE 'not one or zero'
    END

-- selection(1) will evaluate to 'one'
```

# table creation tricks
Partition by splits data into separate files (ie, stuff that should go together in the file system).

 - do not use variables that have high cardinality (a big number of small files -> Hadoop is bad at dealing with a large number of small files)
 - do not use variables with high skew (small nr of values have many rows)
 - do not partition too many times (again, leads to small files)

# querying tricks

## grouping sets
`GROUPING SETS` will rollup (aggregate) across several grouping variables in a single statement (eg, you get both yearly and monthly data, or group by both hotel_cc1 and booker_cc1)

 - Keep in mind, `NULL` is the placeholder. If you have an actual `NULL` value and want data for that, use `GROUPING_ID`
 - `WITH CUBE` will use all of the combinations of the `GROUP BY` variables
 - `WITH ROLLUP` is for when the `GROUP BY` variables are nested (year, month, day)
 
## windowing functions

 - `LAG` will look backwards inside the window specified by the `OVER` clause
 - `SUM(variable) OVER ...`
