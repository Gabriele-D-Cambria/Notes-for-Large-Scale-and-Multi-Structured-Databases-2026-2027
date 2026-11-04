---
title: Column Database
---

# 1. Index

<!--toc:start-->

- [1. Index](#1-index)
- [2. Column Database](#2-column-database)
  - [2.1. Columnar Compression](#21-columnar-compression)
  - [2.2. Single Row Operation Penalty](#22-single-row-operation-penalty)
  - [2.3. Big Table: Google NoSQL Database](#23-big-table-google-nosql-database)
    - [2.3.1. Keyspaces](#231-keyspaces)
      - [2.3.1.1. Row Keys](#2311-row-keys)
      - [2.3.1.2. Columns and Column Families](#2312-columns-and-column-families)
      - [2.3.1.3. Example: Ordering and Compression](#2313-example-ordering-and-compression)
  - [2.4. Cassandra and HBase](#24-cassandra-and-hbase)
    - [2.4.1. Partitioning in Cassandra and HBase](#241-partitioning-in-cassandra-and-hbase)
    - [2.4.2. Column Architecture: HBase](#242-column-architecture-hbase)
    - [2.4.3. Column Architecture: Cassandra](#243-column-architecture-cassandra)
    <!--toc:end-->

# 2. Column Database

Column Databases are a type of DBMS used to handle **huge amount of data** in
_masterless distributed configurations_.

They are used in order to perform two types of processing:

- **OLTP** (Online Transaction Processing): This is the traditional type of processing
  used in relational databases (_ACID transactions_). It is used to handle a
  large number of small transactions, such as inserting, updating, or deleting records.
- **OLAP** (Online Analytical Processing): This is a type of processing used for
  fast analyze large amounts of data in an interactive way. It is typically
  used in **Business Intelligence** applications.

Differently from relational databases, it **does not store data in rows**, but rather
in **columns**.

Historically, this was a huge change, since all OLTP processing was mainly oriented
towards handling **one record at a time** processing.

In this **record-oriented** approach, _CRUD_ operations (Create, Read, Update, Delete)
were the _most time-critical ones_.

In particular, **reporting programs** were typically iterated through entire tables
while running in background (in batch mode). Still, **queries time response**
was not a critical issue, so it was not considered a prominent problem.

However, when _business intelligence_ applications started to spread, **OLAP**
processing assumed a **big relevance**, making queries time response a
critical issue.

This happended because when processing data for making analytics, we are typically
not interested in retrieving all information about a record, but rather on
**one specific attribute of a set of records**.

With row-based storage, this means that, in order to recover a single
attribute, we have to first **access the whole record**.

This is a huge waste of time, since we are not interested in the other attributes.

If all the values of a specific attribute were to be stored together on the disk
(or on a specific block), we could access them all at once, without having to
read the other attributes, resulting in a **faster response time**.

For example, supposing we have a sql table with 6 attributes (`ID`, `Name`, `DOB`,
`Salary`, `Sales`, `Expenses`), and we want to know the sum of all the `Salaries`
of all the employees.

```sql
SELECT SUM(salary)
FROM salePersons;
```

The above query will **scan all the blocks** of the table (supposing we have
one record per block), and for each record, it will **read all the
attributes** in order to extract the `Salary` attribute.

In case of **columnar storage**, we would occupy just 6 blocks (one for each attribute),
and on it we will have stored all the values of that attribute for all the records.

This means that, in order to compute the sum of all the salaries, we will have
to read **just one block** (the one containing all the salaries), and we will
not waste time reading the other attributes.

## 2.1. Columnar Compression

Data compression algorithms basically **remove redundancy** within data values.
Their compression ratio is typically proportional to the amount of redundancy
in the data.

This happens before **compression algorithms** detect redundancy more easily when
the process **small groups of nearby values**.

When using columnar storage, we can store **similar values together** for each columns
(for example we can sort them), and this allows us to **compress data more
efficiently**, achieving _high compression ratio_ with _low computational overheads_.

For instance, the example of having sorted columns allows us to store just the first
value of a columns, and then store just the **difference** $(\Delta)$ from
the preceding value.

## 2.2. Single Row Operation Penalty

As we might have expected, column databases perform **poorly** during _single-row
modification_.

The **overhead** needed for reading rows can be partly reduced by caching multicolumn
data in memory, but this is not a solution for **highly dynamic data**.

In general, in column databases handling a single row means accessing to more
than one block on the disk.

However, modern architectures of column databases are designed to **minimize
this penalty**.

## 2.3. Big Table: Google NoSQL Database

NoSQL databases discussed so far may help us solve the problem of handling
**huge amount of data**.

However, each of them comes with some **limitations**:

- **Key-Value** databases lack support for organizing _many attributes_ and
  keeping **frequently used data** together.
- **Document** databases may not have some of the useful features of relational
  databases, such as a **SQL-like query language** or **secondary indexes**.

To overcome these limitations, in 2006 Google® developed a new type of
database called **Big Table**, for large services such as _Google Earth_ and _Google
Finance_.

Its main features are:

- **Column based** with a **_dynamic_** control over columns, meaning that we
  can add new columns at any time, without having to modify the schema of the
  database. Furthermore, for each row we can have multiple instances of data
  uploaded in different times.
- Data values are accessed by **row** identifier, **column** name, and a
  **time stamp**
- Reads and writes of a row are **atomic**
- **Rows** are maintained in a **sorted order**

<div class="grid2">
<div class="">

Hence, a data record is a **row** composed of values associated to columns in several
**column families**.

Each _family_ consists of a set of **related columns** (i.e., that are
frequently used together).

**Column values** within a column family are **kept together** on the disk, whereas
values of the same row, but belonging to different families, my be stored far.

One limitation of Big Table is that column families **must be defined a priori**.

Applications may **add** and/or **remove** columns inside families.

On the right we can see an use example.

</div>
<div class="">
<figure class="75">
<img class="100" src="./images/column-db/big-table.png"
        alt="Big Table">
<figcaption>

In the end a _column family_ is equivalent to a **key-value** couple,
where the **key** is the _row-identifier_, _column-name_ and _timestamp_.

</figcaption>
</figure>

</div>
</div>

On the image below we can see an example of a column family with two columns,
saved in different time instants.

<img class="" src="./images/column-db/big-table-2.png"
        alt="Big Table 2">

This means that:

- **Row Identifier** $\to$ similar role to relational db _primary key_
- **Column Name** $\to$ similar role to key-value db _key_
- **Timestamp** $\to$ allows for multi versions of a column value to exists

### 2.3.1. Keyspaces

It is the **top-level logical container** that holds _tables_, _column families_,
_row keys_ and related _data structures_ (such as _replication strategies_ and
_replication factors_).

Typically, there is one keyspace per application.

#### 2.3.1.1. Row Keys

Row keys are one of the components used to **uniquely identify** values stored
in a database.

They are also used to **partition** and to **order** data, as data of a
specific row key are **_never deployed across multiple nodes_**.

The algorithms for data partitioning and ordering depend on **specific databases**,
meaning that our data could be partitioned and ordered differently depending on
the database we are using.

As an example, `Cassandra` adopts the **partitioner** algorithm, which is a specific
object that is used for partitioning and ordering data.

Moreover, by default, the partitioner **randomly distributes** rows across nodes
by using a **hash function** on the row key.

#### 2.3.1.2. Columns and Column Families

A column, along with a row key and version/time stamp, is used to **uniquely
identify** a value stored in a database.

The values may be _typed_ (e.g., integer, string, etc.), like in `Cassandra`, or
_untyped_ (e.g., binary data), like in `Big Table`.

Version/Time stamp is a way to **order values of a column**

Columns that are frequently used together are grouped into **column families**,
which are analogous to _tables_ in relational databases.

However, unlike relational databases, rows in column families are **sorted by
row key**, while individual cell values can have multiple timestamped
versions. Also, column families are **defined a priori**, but columns can be modified
dynamically,just by making a reference to them from the _client application_.

<figure class="">
<img class="100" src="./images/column-db/column-family.png"
        alt="Column Family">
<figcaption>

Here we can clearly see that several rows of the same column family can have
different columns.

</figcaption>
</figure>

#### 2.3.1.3. Example: Ordering and Compression

Let's see a practical example with 5 rows and 2 column families, each with up to
3 columns:

<div class="flexbox" markdown="1">
  <table>
    <thead>
        <tr>
          <th rowspan="2">Row Key</th>
          <th colspan="3">Profile (CF)</th>
          <th colspan="3">Activity (CF)</th>
        </tr>
        <tr>
          <th>name</th>
          <th>age</th>
          <th>city</th>
          <th>last_login</th>
          <th>posts</th>
          <th>likes</th>
        </tr>
    </thead>
    <tbody>
      <tr>
        <th>U001</th>
        <td>Alice</td>
        <td>28</td>
        <td>Pisa</td>
        <td>2025-01-15</td>
        <td>12</td>
        <td>134</td>
      </tr>
      <tr>
        <th>U002</th>
        <td>Bob</td>
        <td>34</td>
        <td>Rome</td>
        <td>2025-01-14</td>
        <td>5</td>
        <td>87</td>
      </tr><tr>
        <th>U003</th>
        <td>Carla</td>
        <td>29</td>
        <td>-</td>
        <td>2025-01-15</td>
        <td>-</td>
        <td>21</td>
      </tr><tr>
        <th>U010</th>
        <td>David</td>
        <td>-</td>
        <td>Milan</td>
        <td>2025-01-10</td>
        <td>8</td>
        <td>64</td>
      </tr><tr>
        <th>U020</th>
        <td>Eva</td>
        <td>27</td>
        <td>Pisa</td>
        <td>2025-01-15</td>
        <td>20</td>
        <td>203</td>
      </tr>
    </tbody>
  </table>
</div>

We can see how rows are stored in **increasing lexicographic order** of the
row key.

Compression works by exploiting the way Big Table stores data per column
family (in separate files) and compresses it by exploiting:

- Similarity of values across consecutive rows
- Sparsity (missing cell, posts, etc.)
- Compression algorithms (e.g., _prefix compression_ and _block compression_)

A simplified example of the physical organization of a compressed column
family could be the following:

<img class="" src="./images/column-db/column-family-compression.png"
        alt="Column Family Compression">

To further compress data, we can exploit repetition of a value across rows.
For example, in compressing the `city` column, we can use the following dictionary:

- `-` (missing) $\to$ `0`
- `Pisa` $\to$ `1`
- `Rome` $\to$ `2`
- `Milan` $\to$ `3`

This will mean that instead of having the five strings, we just will be
storing the sequence `12031`.

## 2.4. Cassandra and HBase

`HBase` and `Cassandra` are both **column-oriented databases** that are
designed to handle **large-scale data** in a distributed environment.

However, they have different architectures and design principles.

### 2.4.1. Partitioning in Cassandra and HBase

To see how both `Cassandra` and `HBase` define their _partition key_ and how
they distribute data in the cluster let's see the following table

|                           Aspect                           |                                                   Cassandra                                                    |                         HBase                          |
| :--------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------: | :----------------------------------------------------: |
|                    **_Partition Key_**                     |                               Part of the **Primary Key** (one or more columns)                                |                     **_Row Key_**                      |
|                     **_Primary Key_**                      |       Identifies a row and defines its partitions.<br>Formed as `<partition_key [,clustering_columns]>`.       |                           -                            |
| **_Clustering Column_**<br><small>(Cassandra Only)</small> | Optional columns that identify and _order_ rows within the same partition (e.g., timestamp, customer_id, ...). |                           -                            |
|                **_Partitioning Algorithm_**                |                                      **Hash-Based** (typically `Murmur3`)                                      | **Range-Based** on contiguous interval of the Row Key. |
|                 **_Unit of Partitioning_**                 |                                                 **Partition**                                                  |                       **Region**                       |
|                     **_Row Ordering_**                     |                                   By **clustering columns** with a partition                                   |            Lexographic order of the Row Key            |
|                     **_Replication_**                      |                Partitions replicated on multiple nodes (with a configurable replication factor)                |            Data replicated in storage layer            |

For example, in `Cassandra` we could have the following table schema:

```sql
CREATE TABLE orders (
  customer_id TEXT,
  order_ts TIMESTAMP,
  amount DOUBLE,

  PRIMARY KEY (
    customer_id,  -- partition key
    order_ts        -- clustering columns
  )
);
```

Partitioning by `customer_id` will mean that all the record with a
certain value (e.g., `C001`) will be stored **on the same node**, in a format
**_ordered by `order_td`_**.

Instead, for `HBase` suppose to have the following _row key design_:

```log
Row Key: country#customer_id
```

Rows will be ordered lexicographically by the _Row Key_, with each region storing
contiguous ranges of them.

In this case all columns of a row will be stored in the same region.

### 2.4.2. Column Architecture: HBase

`HBase` is a distributed, scalable, and column-oriented database, with an architecture
inspired by a _master-slave_ model, based on the `Hadoop`/`HDFS` environment.

It works by separating the end nodes that handle the user requests in **Region Servers**
(slaves).

The **Regions** in a specific region server contain all the rows between the
specified keys.

The architecture is based on two _masters_:

- **Hmaster**: assign regions to Region Server
- **Zookeeper**: keeps track of the status of the Region Servers and the Hmaster,
  enabling coordination between nodes within a `Hadoop` cluster.

Furthermore, we have:

- **NameNode**: Manages metadata about the location and properties of data
  in `HDFS`, allowing `HBase` to quickly locate where its data files are stored.
- **DataNode**: Stores the actual data blocks for `HBase` tables in `HDFS` and
  serves them to `HBase` when requested.

<img class="" src="./images/column-db/hbase-architecture.png"
        alt="HBase Architecture">

### 2.4.3. Column Architecture: Cassandra

`Cassandra` is a _distributed_, _masterless_, and _peer-to-peer_ column-oriented
database, designed for high availability and fault tolerance.

All `Cassandra` nodes are equal, running the **same software**.

This way, there is no single point of failure, although different nodes may serve
**different function** for the cluster.

This approach comes with several advantages:

1. **Simplicity**: The architecture is simpler, as there is no need for a
   master node to coordinate the cluster.
2. No node can be a **single point of failure**
3. _Scaling up/down_ is fairly **straightforward**, as new nodes can be added
   or removed without disrupting the cluster.
4. Servers **communicate** with each other
5. When a node is **removed** servers hosting _replicas of data from the
   removed node_ will respond to read/write requests, automatically re-balancing
   the load, and introducing little to no down time.
