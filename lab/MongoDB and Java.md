---
title: MongoDB and Java
---

# 1. Index

<!--toc:start-->

- [1. Index](#1-index)
- [2. MongoDB Authentication](#2-mongodb-authentication)
- [3. MongoDB Java Driver](#3-mongodb-java-driver)
  - [3.1. Access Databases and Collections](#31-access-databases-and-collections)
  - [3.2. Creating Documents](#32-creating-documents)
    - [3.2.1. Validating Documents](#321-validating-documents)
  - [3.3. Inserting Documents](#33-inserting-documents)
  - [3.4. Query Documents](#34-query-documents)
  - [3.5. Update Documents](#35-update-documents)
  - [3.6. Delete Documents](#36-delete-documents)
  - [3.7. Aggregation Pipelines](#37-aggregation-pipelines)
    - [3.7.1. Match Stage](#371-match-stage)
    - [3.7.2. Group Stage](#372-group-stage)
    - [3.7.3. Project Stage](#373-project-stage)
    - [3.7.4. Sort Stage](#374-sort-stage)
    - [3.7.5. Limit and Skip Stages](#375-limit-and-skip-stages)
    - [3.7.6. Unwind Stage](#376-unwind-stage)
  - [3.8. Create Indexes](#38-create-indexes)
  <!--toc:end-->

# 2. MongoDB Authentication

> [!NOTE]
> This document takes for granted what we have already seen on
> [the previous `MongoDB` notes](./MongoDB.md).

In a real world application, we should always **create a user administrator**
before starting to use the database.

To create an administrator user, we can use the following command inside
`mongosh`:

```mongodb
use admin
db.createUser({
  user: "myUserAdmin",
  pwd: passwordPrompt(),        // or cleartext password
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" }
  ]
})
```

At this point we just need to **restart the `mongod` service**.
From now on, when we want to access as an administrator, we have to use the `--auth`
option when starting the `mongosh` service.

```bash
sudo systemctl restart mongod --auth
```

# 3. MongoDB Java Driver

The driver that allows a Java application to connect to a MongoDB database is called
the **MongoDB Java Driver**.

> [!TIP]
> The following notes refer to the [official documentation](https://www.mongodb.com/docs/drivers/java/sync/current/get-started/).

To use it, it is recommended to use a **dependency manager**, `Maven` in our case,
adding the dependency to the `pom.xml`:

```xml
<dependency>
  <groupId>org.mongodb</groupId>
  <artifactId>mongodb-driver-sync</artifactId>
  <version>5.6.0</version>
</dependency>
```

This uses the **synchronous driver**, where each call waits for
the server answer before continuing. There is also an **asynchronous driver**
that can be used in some cases.

To create a connection we need two classes: `MongoClient` and `MongoClients`.

The `MongoClients` class is a factory for creating `MongoClient` instances:

```java
MongoClient mongoClient = MongoClients.create(
  "mongodb://user:pass@localhost:27017"
);
```

The class supports **connection options**, given in a form of a _GET query_
after the port number. We can also use a `MongoCLientSettings` object to create
a `MongoClient` instance with custom parameters, such as timeouts, connection
pool size, credentials, etc.

```java
MongoClient mongoCLient = MongoClients.create(
  MongoClientSettings.builder()
    .applyConnectionString(new ConnectionString("mongodb://user:pass@localhost:27017"))
    .build()
);
```

In general, a `MongoClient` instance represent a **thread-safe pool of connections**
to the database, where a single instance can be shared across multiple threads,
since concurrent access to the single instance in managed by the class itself.

In general, it is recommended to **create one single instance of `MongoCLient`
and use it _across the entire application_**.

> [!TIP]
> Always call `MongoClient.close()` to clean up resources when the instance
> is no longer needed, like at the end of the application.

## 3.1. Access Databases and Collections

Once we have a `MongoClient`, we can use `getDatabase()` to get a
`MongoDatabase` instance by its name.

If the specified database does not exist, it will be created when we first
store data in it.

```java
MongoDatabase database = mongoClient.getDatabase("mydb");
```

Once we have a database, we can access its collections by name using
`getCollection()`, which returns a `MongoCollection<Document>` instance.

```java
MongoCollection<Document> collection = database.getCollection("test");
```

A typical workflow is the following:

```java
import com.mongodb.ConnectionString;
import com.mongodb.client.*;
import org.bson.Document;

ConnectionString uri = new ConnectionString("mongodb://localhost:27017");

// Create a Client
MongoClient mongoClient = MongoClients.create(uri);

// Connect to the Database
MongoDatabase database = mongoClient.getDatabase("mydb");

// Select a collection
MongoCollection<Document> collection = database.getCollection("test");

// CRUD operations

// Close the collection and release resources
myClient.close();
```

Other useful methods when dealing with collections are:

```java
// Create a new collection
database.createCollection("test");

// List the existing collections
for (String name : database.listCollectionNames()) {
  System.out.println(name);
}

// Drop a collection
MongoCollection<Document> collection = database.getCollection("test");
collection.drop();
```

> [!WARNING]
> Dropping a collection will **permanently delete all its documents and
> indexes**.  
> Only drop collections whose data is no longer needed.

## 3.2. Creating Documents

To create a document we can use the `Document` class.

Suppose, we want to add this `JSON` document to a collection:

```json
{
  "name": "MongoDB",
  "count": 1,
  "versions": ["v3.2", "v3.0", "v2.6"],
  "info": { "x": 203, "y": 102 }
}
```

We can create a new `Document` instance with the following code:

```java
Document doc = new Document("name", "MongoDB")
  .append("count", 1)
  .append("versions", Arrays.asList("v3.2", "v3.0", "v2.6"))
  .append("info", new Document("x", 203).append("y", 102));

// OR parse a string
Document doc = Document.parse("{ \"name\": \"Mario\", \"surname\": \"Rossi\"");
```

### 3.2.1. Validating Documents

We can validate the documents in a collection against a _set of filters_ **during
writes**, with the `ValidationOption` class:

It is most useful for an established application, where we have a good sense of
how to organize our data.

```java
ValidationOptions validationOptions = new ValidationOptions()
  .validator(
    Filters.or(
      Filters.exists("name"),
      Filters.exists("email")
    )
  );

database.createCollection(
  "newCollection",
  new CreateCollectionOptions().validationOptions(validationOptions)
);
```

## 3.3. Inserting Documents

To insert documents we can use the `insertOne()` and `insertMany()` methods:

```java
// Add a single document
collection.insertOne(doc);

// Add a list of documents
List<Document> documents = new ArrayList<>();

// ...
// populate the list
// ...

collection.insertMany(documents);
```

If not specified, the `_id` field will be automatically generated for each document.

## 3.4. Query Documents

To query our database we can use the `find()` method:

```java
try (MongoCursor<Document> cursor = collection.find().iterator()) {
  while (cursor.hasNext()) {
    Document doc = cursor.next();
    System.out.println(doc.toJson());
  }
}
```

To retrieve all the documents inside the cursor, it is possible to:

- Iterate it with a **consumer function**
- Collect the documents into a list with `into()`:

```java
import java.util.function.Consumer;

Consumer<document> printConsumer = doc -> System.out.println(doc.toJson());
collection.find().forEach(printConsumer);

List<Document> results = collection.find().into(new ArrayList<>());
```

To filter the documents we can use the `Filters` helper, which provides a set
of filters to use in the `find()` method:

```java
import static com.mongodb.client.model.Filters.*;

// Equivalent query:
// { i: { $gt : 50, $lte: 100 } }
try (MongoCursor<Documen> cursor =
      collection.find(and(gt("i", 50), lte("i", 100))).iterator()) {
  while (cursor.hasNext()) {
    Document doc = cursor.next();
    System.out.println(doc.toJson());
  }
}
```

## 3.5. Update Documents

To update documents we can use the `updateOne()` and `updateMany()` methods.

They both need two parameters:

1. **Filter**: indicates which documents to update
2. **Update**: the actual modifications. We can use the `Updates` helper class

```java
import static com.mongodb.client.model.Updates.*;

// Set the age of the first "Mario" to 28
collection.updateOne(
  eq("name", "Mario"),
  set("age", 28)
);

// Update the first document with "i = 10", and set it to 110
collection.updateOne(
  eq("i", 10),
  set("i", 110)
);


// Increment "i" by 100 in ALL documents with "i < 100"
UpdateResult result = collection.updateMany(
  lt("i", 100),
  inc("i", 100)
);

System.out.println(updateResult.getModifiedCount() + " documents updated.");
```

The update methods return an `UpdateResult` with information about the
operation, e.g. the number of modified documents.

## 3.6. Delete Documents

To delete one or more documents we can use the `deleteOne()` and `deleteMany()`
methods.

```java
// Delete at most one document with "i = 110"
collection.deleteOne(eq("i", 110));

// Delete all documents with "i >= 100"
DeleteResult result = collection.deleteMany(gte("i", 100));
System.out.println(result.getDeletedCount() + " documents deleted.");
```

The delete methods return a `DeleteResult` with information about the operation.

## 3.7. Aggregation Pipelines

Aggregation operations process the documents of a collection and return computed
results.

With aggregations we can perform all the operations doable with `find()`, plus
additional operations like grouping, sorting, and reshaping.

To perform an aggregation, we first need to create the list of stages to
pass to `MongoCollection.aggregate()`.

We use `BSON` objects to represent the stages, with the help of the
`Aggregates` helper class.

For instance, the shell pipeline:

```mongodb
db.collection.aggregate([
  { $match: { categories: "Bakery" } },
  { $group: { _id: "$stars", count: { $sum: 1 } } }
])
```

Becomes the following Java code:

```java
Bson myMatch = Aggregates.match(Filters.eq("categories", "Bakery"));
Bson myGroup = Aggregates.group("$stars", Accumulators.sum("count", 1));

collection.aggregate(
  Arrays.asList(myMatch, myGroup).forEach(printDocuments);
)
```

To import filters, aggregations, projections and accumulators, we can use the following
imports:

```java
import static com.mongodb.client.model.Aggregates.*;
import static com.mongodb.client.model.Accumulators.*;
import static com.mongodb.client.model.Projections.*;
import static com.mongodb.client.model.Filters.*;
```

This makes our previous code more readable:

```java
Bson myMatch = match(eq("categories", "Bakery"));
Bson myGroup = group("$stars", sum("count", 1));

collection.aggregate(
  Arrays.asList(myMatch, myGroup)
    .forEach( doc -> System.out.println(doc.toJson()));
)

```

### 3.7.1. Match Stage

The `match` stage **filters documents** by setting the pass conditions the document
must satisfy to be passed to the next stage.

```java
// String
Bson myMatch = match(eq("name", "MongoDB"));

// Integers
Bson myMatch = match(gt("pop", 50000));

// Logic Operators
Bson myMatch = match( and( eq("name", "XYZ Bar"), eq("categories", "Coffee")));

collection.aggregate(Arrays.asList(myMatch))
  .forEach( doc -> System.out.println(doc.toJson()));
```

### 3.7.2. Group Stage

The `group` stage groups the documents by the `_id` expression, and returns
**one document per group**.

It supports accumulator expressions to compute values for each group, like
`sum`, `avg`, `max`, etc.

```java
// Group by single field and sum the population
Bson groupSingle = group("$city", sum("totPop", "$pop"));

// Group by multiple fields and sum the population
Bson groupMultiple = new Document(
  "$group", new Document(
    "_id", new Document("city", "$city").append("state", "$state")
  ).append("totalPop", new Document("$sum", "$pop"))
);
```

The last query is equivalent to the following shell query:

```mongodb
{
  $group: {
    _id : { city: "$city", state: "$state" },
    totalPop: { $sum: "$pop" }
  }
}
```

### 3.7.3. Project Stage

The `project` stage passes along only the requested fields to the next stage.

They can be **included**, **excluded** or **computed**:

```java
Bson p1 = project(fields(include("city")));
Bson p2 = project(fields(excludeId(), include("city", "state")));
Bson p3 = project(fieltds(computed("myId", "$_id"), include("city")));
```

### 3.7.4. Sort Stage

The `sort` stage sorts all the documents, **ascending** or **descending**,
based on the specified fields.

```java
import static com.mongodb.client.model.Sorts.*;

Bson s1 = sort(ascending("city"));

Bson s2 = sort(descending("city"));

Bson s3 = sort(orderBy(ascending("city"), descending("state")));

collection.aggregate(Arrays.asList(s3).forEach(doc -> System.out.println(doc.toJson()));
```

### 3.7.5. Limit and Skip Stages

The `limit` stage limits the number of documents passed to the next stage, while
the `skip` stage skips the first `n` documents.

```java
// Find the 2nd and 3rd biggest cities by population
Bson mySort = sort(ascending("pop"));
Bson mySkip = skip(1);
Bson myLimit = limit(2);

collection.aggregate(Arrays.asList(mySort, mySkip, myLimit))
  .forEach(doc -> System.out.println(doc.toJson()));
```

> [!WARNING]
> The order of the stages is important, since each stage **receives the output
> of the previous stage**.

### 3.7.6. Unwind Stage

The `unwind` stage **deconstructs an array field** from the input documents to output,
creating one different document for each element.

Thus, each output document will be identical to the input document, except
for the array field, which will now hold just one document.

```java
// Average score of restaurants by with American Cuisine
Bson m = match(eq("cuisine", "American"));
Bson u = unwind("$grades");
Bson g = greoup("$cuisine", avg("avgScore", "$grades.score"));

collection.aggregate(Arrays.asList(m, u, g))
  .forEach(doc -> System.out.println(doc.toJson()));
```

## 3.8. Create Indexes

We can pass an \*_index specification document_ to `createIndex()` to
create an index on a collection.

It will contain the fields to index and the **index type** (`1` for ascending,
`-1` for descending).

```java
collection.createIndex(
  new Document(<field1>, <type1>).append(<field2>, <type2>)...
);
// OR
collection.createIndex(Indexes.ascending(<field1>, <field2>...));
```
