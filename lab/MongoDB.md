---
title: MongoDB
---

# 1. Index

<!--toc:start-->

- [1. Index](#1-index)
- [2. MongoDB](#2-mongodb)
  - [2.1. Data Model](#21-data-model)
    - [2.1.1. JSON vs BSON](#211-json-vs-bson)
    - [2.1.2. Documents](#212-documents)
  - [2.2. Key Features and Core Processes](#22-key-features-and-core-processes)
  - [2.3. Replica and Sharding Architecture](#23-replica-and-sharding-architecture)
  - [2.4. MongoDB vs SQL](#24-mongodb-vs-sql)
  - [2.5. Usage](#25-usage)
    - [2.5.1. Important Files](#251-important-files)
    - [2.5.2. Mongod Database Server](#252-mongod-database-server)
    - [2.5.3. Mongo Shell](#253-mongo-shell)
    - [2.5.4. Mongo ATLAS](#254-mongo-atlas)
    <!--toc:end-->

# 2. MongoDB

`MongoDB` stands for _HuMONGOus DataBase_, and it is a **document-based** NoSQL
database based on the _"high performance, high scalability"_ principle.

It is a source-available cross-platform database available under the _Server Side
Public License (SSPL)_ license, written in `C++` and is running on `Windows`,
`Linux`, and `macOS`.

Its license allows for the following use cases:

- Students can install it and use it for learning.
- Universities can use it in teaching laboratories.
- Companies can use it for internal applications.
- SaaS (Software as a Service) applications can use it as their internal
  database.
- Companies offering `MongoDB` itself as a managed service must comply
  with _SSPL Section 13_ or obtain a _commercial license_.

It allows for different configurations on the **CAP triangle** (mainly
CP and AP).

> [!TIP]
> The official documentation is available at the following links:
>
> - [Manual Overview](https://www.mongodb.com/docs/manual/)
> - [Document Overview](https://www.mongodb.com/docs/manual/core/document/)
> - [ATLAS Architecture Guide](https://www.mongodb.com/docs/manual/administration/architecture/)

## 2.1. Data Model

`MongoDB` is **document-based** database. Documents are saved in `BSON` format,
consisting of field-value pair.

Each document is stored in a **collection**, which are like _tables_ of
relational DBs. The main difference is that the document in a collection
**do not need to have uniform structures**. However, they share the same
**index set**.

Collections are stored in files, which size must not exceed `16MB`. For larger
files, `MongoDB` provides a tool called `GridFS`.

> [!CAUTION]
> For the sake of the project, we **_do not need to use `GridFS`_**.

### 2.1.1. JSON vs BSON

`JSON`, which stands for _JavaScript Object Notation_, is easy for humans to
write/read, and for computers to parse/generate.

`JSON` documents are unordered collections of `field: value` pairs with
6 main data types (`string`, `number`, `object`, `array`, `boolean` and `null`).

It allows for _objects nesting_, and its object fields are not ordered (but array
elements are).

`BSON` instead stands for _Binary JSON_, and it is a **binary-encoded
serialization** of `JSON`-like docs.

It extends the `JSON` model to provide **additional data types**, **ordered fields**,
and to be **efficient for (de)encoding** within different languages.

The `MongoDB BSON` implementation is designed to be **lightweight**, **fast** and
**highly traversable**.

All the `BSON` supported data types are listed in the [official documentation](https://www.mongodb.com/docs/manual/reference/bson-types/).

### 2.1.2. Documents

```bson
{
  _id: 76546567,
  name: "sue",
  age: 26,
  status: "A",
  groups: [ "news", "sports" ]
}
```

There are several advantages of using documents:

- They correspond to **native data types** in many programming languages
- **Embedded documents** and **arrays** reduce need for expensive joins
- **Dynamic schema** supports fluent _polymorphism_.

> [!caution]
> `MongoDB` supports _join_ operation, through the `lookup` operator.  
> However, we **_should avoid using it_ as much as possible**.  
> In the project **_it must be avoided at all costs_**.

By default, each document contains an `_id` field, which has special
characteristics:

- It serves as a **primary key** for the collection
- The value is **unique**, **immutable** and should be any _non-array-type_ or
  _regular expression_.
- Its default data type is `ObjectId`, which is _"small, likely unique, fast to
  generate, and ordered"_

Sorting on ad `ObjectId` value is roughly equivalent to **sorting on
creation time**.

In **sharded collections**, it can be part or not of the _shard key_.

## 2.2. Key Features and Core Processes

It is characterized by **High Performance** by providing:

- **High performance data persistence**
- Support for **embedded data models** reduces I/O activity on database system.
- **Indexes** that support _faster queries_ and can include keys from embedded
  documents and arrays.

We define it as a **Rich Query Language**, since it supports read and write
operations (CRUD) as well as _Data Aggregation_, _Text Search_ and _Geospatial
Queries_.

`MongoDB` also provides a replication facility, called **replica set**, that and
allows for automatic failover and data redundancy though _master/slave_ organization.

As one of its core functionality, it provides _horizontal scalability_ through
**sharding** across a _cluster of machines_, though the creation of
_zones of data_ based on the **shard key**.

> [!note]
> As far as the project is involved, we **need to design the sharding** but
> **_we are not required to actually implement it_**.

The core components in the `MongoDB` package are:

- `mongod`: the main _daemon_ process for the `MongoDB` system. It handles
  data requests, manages data access, and database process. [Here for the offical
  documentation](https://www.mongodb.com/docs/manual/reference/program/mongod/)
- `mongos`: the controller and query router interfaced between client applications
  and the **sharded cluster**. [Here for the offical documentation](https://www.mongodb.com/docs/manual/reference/program/mongos/)
- `mongosh`: the official interactive shell replacing the legacy _mongo client_

## 2.3. Replica and Sharding Architecture

<div class="grid2">
<div class="">

In the image on the right we can see an architecture consisting of a **primary
server** and **two secondary servers**.

If the primary server fails, one of the secondary servers will be automatically
promoted to primary.

Default reads are done on the primary server, but it supports _read-with-preference_,
which allows to read from secondary servers, and _write concern_, which allows to
automatically replicate writes to secondary servers.

</div>
<div class="">
<img class="40" src="./images/mongodb/replica-architecture.png"
      alt="Replica Architecture">
</div>
</div>

`MongoDB` shards data at the **collection level**, distributing the collection data
across the shards in the cluster.

When sharding is enabled, the drivers must have access to:

- **Shard Servers**: they store the actual data. Each server can save
  **one primary shard** and _several secondary shards_ for
  _different sharded collections_.
- **Config Server**: it stores the metadata of the sharded cluster, including
  _the list of shards_, _the sharded collections and their shard keys_ and the _chunk
  distribution_ for each sharded collection.

<img class="" src="./images/mongodb/sharding-architecture.png"
      alt="Sharding Architecture">

## 2.4. MongoDB vs SQL

We can summarize the main differences between `MongoDB` and `SQL` databases in
the following table:

<img class="30" src="./images/mongodb/mongodb-vs-sql.png"
      alt="MongoDB vs SQL">

## 2.5. Usage

### 2.5.1. Important Files

The most important file in the `MongoDB` package is the **default configuration
file**.

- On `Linux`, the default `/etc/mongod.conf` configuration file is included
  when using a package manager to install MongoDB.
- On `Windows`, a default `<install directory>/bin/mongod.cfg` configuration file
  is included during the installation.
- On `macOS`, a default `/usr/local/etc/mongod.conf` (`/opt/homebrew/etc/mongod.conf`
  on Apple M1 processors) configuration file is included when installing from
  MongoDB’s official Homebrew tap.

To understand the configuration file, we can refer to the [official documentation](https://www.mongodb.com/docs/manual/reference/configuration-options/).

### 2.5.2. Mongod Database Server

We consider a **localhost installation**, where `27017` is the **default port**
the standalone `mongod` listen on.

Different OS have different commands for running and stopping the `modngod` process.

> [!note]
> On this notes I will use the `Linux` commands, but you can refer to the
> [official documentation](https://www.mongodb.com/docs/manual/tutorial/manage-mongodb-processes/)
> for other OS.

```sh
sudo systemctl start mongodb.service      # start the service
sudo systemctl stop mongodb.service       # stop the service

sudo systemctl enable mongodb.service     # start the service now and at each boot
sudo systemctl disable mongodb.service    # stop the service now and at each boot
```

### 2.5.3. Mongo Shell

The _mongo shell_ `mongosh` is an **interactive JavaScript** interface to `MongoDB`.

We can use the mongo shell to **query and update data** as well as
perform **administrative operations**.

To connect to a `MongoDB` instance:

```sh
mongosh                               # default localhost and 27017 port
mongosh "mongodb://localhost:27017"   # explicit ip and port
momngosh --port 27017                 # explicit port
```

Instead, to shutdown the server from the shell:

```mongodb
db.shutdownServer();
```

In the following code block we can see some of the most important commands to use
in the `mongosh` shell.

```mongodb
// show all databases
show databases
show dbs

// Display the current database
db

// Switch to a database (create it if it doesn't exist)
use <database_name>

// Create a new collection by inserting a document
db.<collection_name>.insertOne({ /* ... */});
```

In the next chapter we will see more operations and commands to use in the
`mongosh` shell.

### 2.5.4. Mongo ATLAS

`MongoDB Atlas` is the **fully managed cloud platform** by `MongoDB` that allows
us to build, deploy and scale globally distributed databases with built-in
features for _replication_, _sharding_, _security_, _vector search_ and
_real-time analytics_.

The service is available on [the official website](https://www.mongodb.com/cloud/atlas).
