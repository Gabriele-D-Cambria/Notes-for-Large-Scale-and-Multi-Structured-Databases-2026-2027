---
title: Redis
---

# 1. Index

<!--toc:start-->

- [1. Index](#1-index)
- [2. Redis](#2-redis)
  - [2.1. Connecting to Redis](#21-connecting-to-redis)
  - [2.2. Redis Keys](#22-redis-keys)
  - [2.3. Redis Commands](#23-redis-commands)
  - [2.4. Transactions](#24-transactions)
  - [2.5. Java Client for Redis (Jedis)](#25-java-client-for-redis-jedis)
  - [2.6. When to Use Redis](#26-when-to-use-redis)
  <!--toc:end-->

# 2. Redis

**`Redis`** is an acronym for _REmote Dictionary Server_.
It is an open-source, **in-memory database** that supports various data structures
as well as **atomic operations** on them.

Because of this, it is mainly used as a **cache**, **message broker**, and **streaming
engine**.

It leverages the speed of main memory and alleviates load off the central application
database, especially for data that changes infrequently and is accessed often.

`Redis` is typically used under this workflow:

1. Look for the data in `Redis`.
1. If data is found, return it.
1. If data is not found, query the database for it.
1. Store the data in `Redis` for future requests.

However, `Redis` gives enough guarantees to be used as a primary database for
some use cases.

It also supports **periodic data persistence** to disk, as well as
transactions, pub/sub messaging, _keys with limited time-to-live replications_,
scaling and more.

## 2.1. Connecting to Redis

We can download and install `Redis` from the official website: [https://redis.io/download](https://redis.io/download).

By default, `Redis` allows connection only from `127.0.0.1`, but for
development purposes, we can allow connections from any IP address by changing
the `bind` directive in the `Redis` configuration file (`/etc/redis/redis.conf`):
<small>(on arch the package is called `valkey`)</small>

```conf
# bind 127.0.0.1
bind 0.0.0.0
```

Once we update the configuration file, we **must restart** the `Redis` service.

To connect to `Redis`, we can use the `redis-cli` command-line tool:

```bash
redis-cli ping
```

If everything is working correctly, we should see the response `PONG`.

## 2.2. Redis Keys

Keys are **unique** in `Redis`. They are used to locate a specific value.

`Redis` is _binary safe_, meaning that we can use both a string or the content
of a JPEG file as a key, as well as an _empty string_. However, it is recommended
to use **human-readable strings** as keys.

Very long keys can have a negative impact on performance, so it is recommended to
instead use hash functions to generate shorter keys.

```sh
sales-app:invoice:10012:status = "PAID"
shopping-card:1002 = "[{product_id: 1001, quantity: 2}]"
```

Try to stick with a schema for the keys, such as `EntityName:EntityId:EntityAttribute`.

`Redis` supports a maximum key length of **512 MB**. However, it is
recommended to keep keys under **1 KB** in size.

## 2.3. Redis Commands

The following are some of the most commonly used `Redis` commands:

```sh
# Set a new key-value pair
SET <key> <value>

# Get the value of a key
GET <key>

# Set a key with an expiration time
EXPIRE <key> seconds> NX
SET <key> <value> EX seconds

# See the time to live of a key
TTL <key>

# To delete a key
DEL <key>
```

For integer values, we can use the following commands:

```sh
SET counter 10

# Increment the value of the key by 1
INCR counter

# Increment the value of the key by 5
INCRBY counter 5

# Decrement the value of the key by 1
DECR counter

# Decrement the value of the key by 3
DECRBY counter 3
```

`Redis` also supports **Pipelining**, a technique that allows to **improve performance**
by sending _more commands at once_, without waiting for the response of each one.

To do pipelining we can invoke:

```sh
redis-cli --pipe
```

## 2.4. Transactions

`Redis` supports transactions, which allow us to execute a group of commands in
a **serialized and sequential manner**.

To do so we use a few commands:

```sh
# Start a transaction
MULTI

# ... Commands that will be Queued ...

# Discard the transaction and all commands queued
DISCARD

# ... Commands that will be Queued ...

# Triggers the execution of all commands in the transaction
EXEC
```

If we have multiple clients trying to modify the same key, it is important
to use the `WATCH` command to **monitor** the key for changes.

If the key is modified by another client before the transaction is executed,
the transaction will be aborted (returns a `(nil)` response).

```sh
WATCH <watched_key>

MULTI

# ... Commands that will be Queued ...

EXEC
```

## 2.5. Java Client for Redis (Jedis)

`Jedis` is a Java client library for `Redis`. It provides a simple and
easy-to-use API which commands are implemented as methods of the
`redis.clients.jedis.Jedis` class, with specification on the official
documentation [https://javadoc.io/doc/redis.clients/jedis/latest/index.html](https://javadoc.io/doc/redis.clients/jedis/latest/index.html).

On GitHub, we can find the project at [https://github.com/redis/jedis](https://github.com/redis/jedis).

Jedis is supported by Maven, and it includes **by default** the `GSON` dependency,
which is useful for serializing and deserializing JSON data.

```xml
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>7.0.0</version>
</dependency>
```

To establish a connection to `Redis` a first code snippet would look like this:

```java
import redis.clients.jedis.Jedis;

String redisHost = "localhost";
Integer redisPort = 6379;

Jedis jedis = new Jedis(redisHost, redisPort);

// Stuff here

jedis.close();
```

We can also use a `try-with-resources` statement to automatically close the connection:

```java
import redis.clients.jedis.Jedis;

String redisHost = "localhost";
Integer redisPort = 6379;

try (Jedis jedis = new Jedis(redisHost, redisPort)) {
    // Stuff here
}
```

If we want to improve the performance of our application, we can use a
`JedisPool` to manage a pool of connections:

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

try (JedisPool pool = new JedisPool(redisHost, redisPort);
    Jedis jedis = pool.getResource()) {
  // Stuff Here
}
```

We can execute the commands using the `Jedis` instance:

```java
String key = "my-app:some-key";

jesis.set(key, "Hello World!");
jedis.expire(key, 60); // Set the key to expire in 60 seconds

String valule = jedis.get(key);
long ttl = jedis.ttl(key);
```

It also supports transactions using the `Transaction` class:

```java
String key1 = "my-app:some-key-1";
String key2 = "my-app:some-key-2";

jedis.watch(key1);

Transaction transaction = jedis.multi();
transaction.set(key1, "Hello World!");
transaction.set(key2, "Hello Redis!");

/**
 * The output will be a list with the output of
 *  each command in the transaction, in the order they were queued.
 *
 * If a key marked with `WATCH` is modified by another client before the
 *  transaction is executed, the function will return `null`
 */
List<Object> results = transaction.exec();
```

## 2.6. When to Use Redis

`Redis` is a great choice for application that deals with **volatile data**, for
example when we have a _shopping cart_, when the data is **frequently
accessed** but **rarely modified**, for example a list of countries, or when we
**only access data by identifier** and we want to _reduce response time_.

Several big companies use `Redis` in production, including **GitHub**,
**Twitter**, **StackOverflow**.

For example, **Trenitalia** uses `Redis` for the shopping-cart, using the time-to-live
feature to automatically delete the cart after 10 minutes if not purchased yet.
