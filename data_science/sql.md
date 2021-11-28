# config and admin

## config file
> Located in /etc/mysql/my.cnf

## grants
This command will show you the grants that you have

```sql
show grants;

-- also
show grants for current_user;
show grants for user_name;
```

# syntax cheat sheet

## select statements
```sql
SELECT Units, Price
FROM tSales
WHERE Units > 1 AND StoreName = 'Zagreb Ilica'
ORDER BY Price ASC;
```

### constants in the select clause
if you include a constant (e.g., null) in the select clause, this will be added to every tuple

### order by
orders the results, can use multiple variables

```sql
SELECT ProductName
FROM tProducts
ORDER BY Price, Brand
```

### distinct
eliminates duplicates
```sql
SELECT DISTINCT Units, Price ...
```

### changing names for readability
```sql
FROM tSales s  -- (name for tSales in this query is now S)
WHERE s.Units > 1
```

```sql
-- also necessary for comparing tuples from the same table
SELECT Units, Price
FROM tSales S1, tSales S2
WHERE S1.Price = S2.Price AND S2.BillID > S1.BillID
```

```sql
SELECT Units, Price * 2 as doublePrice  -- renaming columns
```

## set operators
```sql
SELECT Units, Price FROM tSales
union
SELECT Units, Price FROM tSalesAdditional

SELECT Units, Price FROM tSalesBeginningOfYear
intersect
SELECT Units, Price FROM tSalesWholeYear

SELECT Units, Price FROM tSalesWholeYear
except
SELECT Units, Price FROM tSalesJanuary
```

### logical tests with sets
```sql
-- is in the set
SELECT Units
FROM tSales
WHERE ProductID IN (SELECT ProductID FROM tProducts WHERE Brand = 'Nike')

-- there is at least one
SELECT ProductID
FROM tProducts
WHERE EXISTS (SELECT * FROM tProducts WHERE Brand = 'Marko Brand')

-- is true for all
SELECT ProductID
FROM tProducts
WHERE Price => all (SELECT Price FROM tProducts)

-- is true for at least one
SELECT ProductID
FROM tProducts
WHERE Price => any (SELECT Price FROM tProducts)
```

## subqueries

### in the where clause
```sql
SELECT ProductName
FROM tProducts
WHERE ProductID in (SELECT ProductID FROM tSales WHERE DateTime='2014-04-01')
```

### in the from clause
gives back a relation that you can query

### subqueries in the select clause
has to return one row per row of the other selected variables

## joins
```sql
-- implicit join - gives a cross product of the tables
SELECT Units
FROM tSales, tProducts
WHERE tSales.ProductID = tProducts.ProductID

-- natural join keeps tuples with same values in common attributes, eliminates duplicate columns
SELECT Units
FROM tSales NATURAL JOIN tProducts

-- inner (theta) join
SELECT *
FROM tSales (INNER) JOIN tProducts ON tProducts.Price > 1000

-- with the "using" clause. You cannot have both on and using
SELECT *
FROM tSales (INNER) JOIN tProducts USING (ProductID)

-- outer joins (left, right, full). Tuples that dont satisfy the condition are padded with nulls
SELECT *
FROM tSales LEFT OUTER JOIN tLoyalty USING BillID
```

## aggregating data (= statistics)

 - functions: count (distinct is useful with this), avg, sum, min, max
 - group by clause gives the statistic for separate groups
 - if in the select clause you put a non-grouping variable the query will return a random value

```sql
SELECT sum (Amount)
FROM tSales inner join tProducts using (ProductID)
GROUP BY BrandName

-- having clause, needs group by and applies to all tuples under the group by variables
SELECT BrandName
FROM tSales inner join tProducts using (ProductID)
GROUP BY BrandName
HAVING Amount > 250
```

# recursion in SQL
You need recursion when you don't know how many steps an algorithm needs to take: e.g., find all ancestors of a person in a parent-child relation

 - achieved using the WITH statement
 - you can use WITH for other purposes, it sets up a temporary table (like a view)

```sql
with  [recursive]
R1 [(list of attribute names)] as [query] -- if it's recursive, you can reference R1),
R2 as (query),...
-- then query involving R1, R2 and other tables
```


- typically in the recursive query you would have a base query (no R) and a recursion query (includes R), thus adding new tuples to the results until some limit is reached. Since union eliminates duplicates, a natural limit is when there are no new tuples left to add
- attribute names in the query with the recursion can be used as variables later, example:

```sql
with recursive
Ancestor (a,d) as (
     select parent as a, child as d from ParentOf
     union
     select Ancestor.a, ParentOf.child as d
     from Ancestor, ParentOf
     where Ancestor.d = ParentOf.parent)
select a from Ancestor where d = 'Mary'
```

You can limit the number of results to stop infinitive queries using limit N

# database design

## normal forms
**Functional dependencies** (FD): if two tuples have the same value for A, they will have the same value for B; where A and B can be combinations of values

 - if A determines all other values in the relation, A is key for that relation
 - splitting rule: if A->B1, B2, B3 then A->B1, A->B2, etc. But this does not function on the left side (if A1, A2, A3 -> B, it is not that A1 ->B)
 - closure: for attributes A, closure is all of the attributes B that are determined by them. If the closure of A are all attributes in a relation, A is a key. A way to find all the keys for a relation is if you start with all single attributes, then find their closure; then all pairs of attributes, etc

**Boyce-Codd normal form** (BCNF): each FD is a dependency between a key (can be multiple attributes) and other attributes

- there could be no FDs - this is then in BCNF
- there is an algorithm that will decompose a non-BCNF relation into BCNF relations

**Multivalued dependencies** (A ->> B): if two tuples agree on values A, then there exists a third tuple where it has the B values of the first tuple, and the rest from the second tuple, and, logically, a fourth tuple that has the B values of the second, and the rest from the first). Another way of saying this: if A ->> B, then A determines B-rest combinations, where these are all B-rest combinations. Or another: B and rest values are independent.

 - also called tuple generating dependencies because if they exist, then they suggest that additional tuples exist
 - an example: a student applies to multiple colleges and has more than one hobby. For each college-student combination, you repeat all the hobbies that the student has
 - main idea is that you want to separate independent information (e.g., if a student has multiple hobbies and applies to multiple colleges, you don't want to have these two pieces of data in the same relation)
 - rules:
    - if A ->> B, then A ->> rest
    - every functional dependency is a multivalued dependency (though this is trivial)
    - there are trivial MVDs, they do not "count"

**Fourth normal form** (4NF): for each A ->> B, A is a key.

- implies BCNF
- there could be no MVDs - this is then 4NF
- there is a very similar algorithm to decompose relations into 4NF

### Performance implications:
The normalization rules are designed to prevent update anomalies and data inconsistencies. With respect to performance tradeoffs, these guidelines are biased toward the assumption that all non-key fields will be updated frequently. They tend to penalize retrieval, since data which may have been retrievable from one record in an unnormalized design may have to be retrieved from several records in the normalized form. There is no obligation to fully normalize all records when actual performance requirements are taken into account

- the relations produced through decomposition algorithms might lead to too many joins
- the algorithms do not always lead to the same results, and some of the resulting relations might have more sense than others

## unified modeling language
Unified modeling language (UML) is a graphical diagramming technique that provides a standard way to describe a system. It is consisted of classes, their attributes and methods which operate on them, the methods not being necessary when speaking about modeling only data. Data modeling adds a concept of primary key. It has 5 main concepts:

 - classes (translates to relations in RBDMS)
 - associations between classes (one-to-one, one-to-many, many-to-many; all of which can be complete). A class can also be associated to itself
 - association classes: attributes of associations
 - subclasses (inherit the superclass attributes and can add their own; they are complete / incomplete and disjoint(exclusive) / overlapping)
 - composition: specific type of association, where objects of one class belong (are a part of) to objects of another class (e.g., college - departments)
    - aggregation: type of composition where an object of one class can, but doesn't have to belong to an object of another class

## optimizing performance
Indexes - you build them for specific values in relations, and they allow you to go directly to the specified tuples, instead of scanning the whole relation

 - you can ask an index any condition on the value
 - more than one value can be involved in an index
 - downsides:
    - extra space (because they are persistent data structures)
    - index creation takes up processing time
    - index maintenance - if the values change, the index has to change. Very costly when there is a lot of updating. When a DB has more updating than querying, indexes make little sense
 - benefit of an index depends on:
    - size of table: larger tables profit more
    - query vs update: more querying, more profit
 - physical design advisor: you input the database information and workload (the queries you do), it spits out recommended indexes
 - SQL syntax:
    - `create index indexName on table (value1, value2...)  #1 or more values`
    - `create unique indexName ...`
    - `drop index indexName`
 - composite indexes are read from left to right. If you don't use the leftmost column in your query the index will not work.

## views
Views represent the highest level of abstraction in a database, they are recombinations of data from the relations or other views

 - Why?
    - To hide data from some users
    - Make some queries easier
    - Make the access to db modular
 - Views data is not stored anywhere, but anytime you access the view the statement is rewritten with the "lower level" relations. And it's written in the most efficient way.
 - Syntax: `Create view Name as SQL_statement`
 - You can modify views (actually the relations that they are made of), but sometimes it is not possible to do this unambiguously (eg, when a view is made up of averages)
    - You use instead of triggers to modify views
    - or restrict modifications to "updateable" views (e.g., in MySQL): select on one table, no aggregation, etc.
    - sometimes you can insert smt into a view, but because of the restrictions the tuple wouldn't show in the view. Both options from above allow you to avoid this problem

### materialized views
The view is stored as a table

 - faster querying over the view
 - but takes disk space and greater overhead because you have to recompute the view every time you modify the base tables
 - syntax: create materialized view Name as...
 - how to choose materialized views?
    - the more data, the better the performance
    - the more complex the view (i.e., the query that creates it), the greater the performance gain
    - the query / update trade-off: the greater the ratio in favor of the querying,
    - "incremental maintenance" means updating just the part of the materialized view that got updated
    - automatic query rewriting means that the system will use materialized views when this will speed up a query automatically (SQL server calls this "indexed views")
    - modern systems have algorithms that suggest which materialized views to create

## partitioning
Splitting tables in order to optimize queries

 - horizontal: splitting across rows. When there are more parts of the data that are not queried together (e.g., different regions of your users)
 - vertical: splitting across columns. When there are variables you don't often use.

## database structure

### constraints
Impose restrictions on the state of the database, i.e., the allowed data and its structure

 - types (int, numeric, string...) and structure of the DB already impose restrictions, but constraints are more semantic (e.g., price cannot be > 9999Eur)
 - when smt does not satisfy a constraint, an error is triggered
 - why?
    - catch input and update errors
    - enforce consistency if same data is present in multiple places
    - tell the system about the data so it stores and processes them more efficiently

classification (some of these you can just add next to attribute names when creating a table)
 - not null
 - primary key (has to be unique, can be a combination of attributes)
    - keyword "unique" for a non-primary key, can also be a combination of attributes
 - referential integrity (foreign key)
 - attribute based: constrain the values of an attribute
    - syntax: price int check (price < 9999 and price > 0)
 - tuple based: constrain how the values of a tuple relate to each other
    - syntax: create table...;  check (price<4000 and productType='sneaker')
    - general assertion: use queries to specify constraints across the db (not implemented in any db, wtf?)
 - declaring constraints: initially or after the initializing the db
 - enforcement:
    - check after every modification (not all, only those involved)
    - deferred checking: we conduct modifications that violate constraints, and check for constraints after they're done (we check after transactions)
    - Enforcement: error (= restrict), set null, cascade

Foreign key is a particular type of constraint that says that for every value or a value combination in relation A there needs to be a corresponding set in B

### triggers
Monitor DB changes and initiate actions under specified conditions (event-condition-action rules)

 - often used only to enforce constraints because triggers are more expressive than constraints, but also because with triggers you can "repair" data
 - or you can use them to set some attributes based on others
 - SQL syntax: "[ ]" means optional
    - create trigger triggerName
    - before | after | instead of events
        - events = insert, delete or update
    - [referencing variables] e.g., use old table as oldTable
        - Set names for referring to old/new row/table. The new table is the set of all changed rows.
    - [for each row]
        - Whether you want the trigger to activate once for each row or for the whole table
    - when (condition)

action
  - Keyword for error in SQLite is raise (ignore), ignore for dissalowing the change

## OLAP: online analytical processing
OLAP entails long data transactions, complex queries that touch large portions of the database, infrequent updates. Compared to OLTP (online transaction processing), which entails short transactions, fetches of single rows or similar, and frequent updates (e.g., fetching a username password)

### data warehousing
Bring data from OLTP sources into a single warehouse for OLAP processing

- organizing data for OLAP typically entails organizing it in a star schema:
    - fact table: updated frequently, very large (e.g., sales transactions)
    - dimensions table: describes characteristics of the entities in the fact table (e.g., productID and product characteristics)
        - dates can be dimensions
- typical OLAP query:
    - join the dimensions and facts
    - filter the dimension values you want (e.g., give me only stores in Zagreb)
    - group the data (e.g., data for each store in Zagreb separately)
    - aggregate (e.g., sum the sales)
- OLAP queries use materialized views or special indexes!

### data cube
dimensions form axes of the (n-dimensional) cube, fact data is in cells, aggregated data is on the side

 - you can drill down the data cube (add additional dimensions) or roll up (aggregate over some of the dimensions); these will be very important concepts for when we do the products section
 - specific query syntax with OLAP
    - with cube, you write it after group by and you get the cross-tabluations over every group by attribute, as well as aggregations for smaller numbers of dimensions (e.g., if you add city, item and period, you will get aggregations also over all periods, but specific item and city; etc)
        - with cube (attribute list) returns the aggregations only for the listed attributes
    - the OLAP cube is great for creating a materialized view since it contains both the "raw data" over each attribute combination, but also aggregates over attributes. Retrieving these aggregates from the cube is much more efficient than computing them every time you run the query
    - with rollup gives you aggregations only for some combinations of the aggregation attributes, and it does so hierarchically.
        - example: with 3 attributes, you get aggregation for the third with the first two constant, you get aggregation for the second with the first constant, and you get all three aggregated together. This is for when you have a hierarchical relationship between the attributes (e.g., region, city, district)

# stability
Transactions are a solution to the problems of concurrency (two users trying to modify the same data at once) and system failures. A transaction is a sequence of SQL operations treated as a unit.

 - Each transaction's changes are reflected entirely or not at all. You can then roll back transactions when system fails.
 - Transactions achieve consistency by being serialized, i.e., by being executed one by one. You can relax transaction seriazibility in order to achieve greater performance.

# authorization
- you can set privileges to read, update or delete specific relations and/or attributes in them
- more fine grained authorizations can be set using views
- the creator of a relation is its owner who can grant privileges
    - syntax:
grant privileges on relationName to users [with grant option]
and revoke privileges on relationName from users [cascade | restrict]

# relational algebra
Operators of relational algebra (sets - cannot have duplicates)

- Union, intersection, difference
- Selection
- Projection
- Join
- Extended relational algebra (bags)
    - Duplicate elimination
    - Grouping and aggregation
    - Sort

Union combines tuples from the selected sets and omits duplicates

- Union all includes duplicates

Difference (sql except) x-y gives back tuples from x that do not appear in y

Intersection (sql?) gives everything in x that is also in y. Can be expressed with diffe

Selection (sql where) gives tuples that satisfy a certain condition

Projection eliminates columns

Cross-product produces a table with a combination of all tuples in x and y

Equi join returns tuples from two tables that satisfy a certain condition of equality

Theta join is the more general join where the condition does not have to be one of equality

Outer join gives back tuples that satisfy a certain condition, but when a tuple from x does not have a corresponding tuple in y, it still leaves x but with null for the information that should have come from y

# tips
http://stackoverflow.com/questions/3049283/mysql-indexes-what-are-the-best-practices

PostgreSQL is the open source database that supports functions defined in R

flushing (on default done automatically on many commands) in SQLAlchemy does not commit to the database
http://stackoverflow.com/questions/4201455/sqlalchemy-whats-the-difference-between-flush-and-commit

MySQL requires that foreign keys are indexed in the parent table

preventing SQL injection attacks
use parametrized queries if doing them directly from outside code (PHP, Python...)
http://stackoverflow.com/questions/1633332/how-to-put-parameterized-sql-query-into-variable-and-then-execute-in-python
http://stackoverflow.com/questions/6786034/can-parameterized-statement-stop-all-sql-injection

or use parameters inside stored procedures and don't concate them into dynamic SQL code
http://blogs.msdn.com/b/brian_swan/archive/2011/02/16/do-stored-procedures-protect-against-sql-injection.aspx
http://www.sqlinjection.net/advanced/stored-procedure/

stored procedures MySQL tutorial
http://www.mysqltutorial.org/mysql-stored-procedure-tutorial.aspx

indexes in MySQL
http://stackoverflow.com/questions/3049283/mysql-indexes-what-are-the-best-practices
http://stackoverflow.com/questions/4194943/mysql-make-a-compound-index-of-3-fields-or-make-3-separate-indices
http://stackoverflow.com/questions/707874/differences-between-index-primary-unique-fulltext-in-mysql

primary keys in SQL: composite vs surrogate auto-increment
http://weblogs.sqlteam.com/jeffs/archive/2007/08/23/composite_primary_keys.aspx
http://stackoverflow.com/questions/4737190/composite-primary-key-or-not
http://stackoverflow.com/questions/337503/whats-the-best-practice-for-primary-keys-in-tables
http://stackoverflow.com/questions/11637914/create-userid-even-though-usernames-unique/11639189#11639189
http://stackoverflow.com/questions/12406870/auto-increment-vs-composite-key

primary key is automatically indexed in MySQL (probably elsewhere too)
http://stackoverflow.com/questions/1071180/is-the-primary-key-automatically-indexed-in-mysql