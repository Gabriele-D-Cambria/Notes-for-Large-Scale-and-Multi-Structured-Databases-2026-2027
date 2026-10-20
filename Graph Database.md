---
title: Graph Database
---

# 1. Index

<!--toc:start-->

- [1. Index](#1-index)
- [2. Graph Database](#2-graph-database)
  - [2.1. Introduction to Graphs](#21-introduction-to-graphs)
    - [2.1.1. Graphs Metrics](#211-graphs-metrics)
    - [2.1.2. Graph Traversal](#212-graph-traversal)
    - [2.1.3. Isomorphism](#213-isomorphism)
    - [2.1.4. Betweenness of a Vertex](#214-betweenness-of-a-vertex)
    - [2.1.5. Flow Networks](#215-flow-networks)
    - [2.1.6. Bipartite and Multi Graphs](#216-bipartite-and-multi-graphs)
  - [2.2. Using Graph Databases](#22-using-graph-databases)
    - [2.2.1. Graph vs Relational Database](#221-graph-vs-relational-database)
    - [2.2.2. Graph Oriented Design](#222-graph-oriented-design)
    <!--toc:end-->

# 2. Graph Database

## 2.1. Introduction to Graphs

**Graph Databases** is a type of NoSQL database that uses graph structures.

In this type of database, instances of entity are represented through
_vertices_ (or _nodes_) that _may gave a **relation**_ with another instance.

The relations are represented through **edges** (or **links**), and they
are typically associated with one or more property (weight, distance, etc.),
representing some value about the _strength_ of the relation.

The edges can be _directed_ or _undirected_. _Directed edges_ are used to represent
a **one-way relationship** between two nodes, while _undirected edges_ are
used to represent a **two-way relationship** between two nodes.

The presence (or absence) of a direction depends on **specific applications**, and
makes a **Graph** either **Directed** or **Undirected**.

<img class="" src="./images/graph-database/directed-undirected-graph.png"
  alt="Directed and Undirected Graphs">

We call the _set of vertices and edges_ that connect two vertices a **path**.
It allows to **capture information about the relations** of those vertices.

On some graph, there may be **multiple paths** between two vertices. In those case,
the problem of **finding the best path** is often taken into consideration.

There also can be **loops**, which are relation of a vertex to itself.

We can perform mathematical operations on graph, such as:

- **Union**: Combines the set of vertices and edges from two or more graphs
- **Intersection**: Is the set of vertices and edges that are _common_ to both
  graphs

In general, the output of both these operations is a new graph (or an empty one).

### 2.1.1. Graphs Metrics

Some useful metrics of graphs include to evaluate time and memory occupancy
required to handle a specific operation on a graph are:

- **Order**: the number of vertices in the graph
- **Size**: the number of edges in the graph

On top of those, other important metrics are the **Degree Centrality** and the
**Closeness Centrality**.

The **Degree Centrality** measures the _degree_ of the vertices (the number of
edges incident to it), measuring their importance within the graph.

Instead, **Closeness Centrality** measures the _average distance_ from a
vertex to all the other, based on the _shortest-paths distances_.

For a _connected graph_, we calculate the closeness centrality of a vertex $v$ as:

$$
  C_C(v) = \frac{n - 1}{\sum_{u \ne v}{d(v,u)}}
$$

Vertices with **high closeness** can reach the others through a **few number of
connections**.

### 2.1.2. Graph Traversal

Graph traversal is the process of **exploring a graph** by moving from one
vertex to another along its edges according to a specific strategy.

There are several traversal strategies, with the most common being:

- **Breadth-First Search (BFS)**: Explores all vertices at the current depth before
  moving to the next level.
- **Depth-First Search (DFS)**: Explores as far as possible along each branch before
  backtracking to visit unexplored vertices.

In both cases, the traversal may visit all vertices or only a subset of them, depending
on the objective.

The main purposes of graph traversal may include:

- The exploration of all vertices reachable from the starting vertex (also called
  **root**)
- Reading or updating properties associated with vertices and edges.
- Discovering connections and paths between vertices.

For example, we might want to explore a network of cities starting from a
specific city, progressively visiting other cities by following existing road connections.

In this case, the traversal strategy determines the order in which vertices
are explored.

### 2.1.3. Isomorphism

Tho graphs are **Isomorphic** if there exists a **one-to-one correspondence** between
their vertices that _preserves adjacency_.

In particular:

- The graphs have the same number of vertices and edges.
- Corresponding vertices have _the same connections_.

The visual representation of isomorphic graphs may differ, but they are considered
equivalent in terms of their structure.

This property is particularly useful because it allows us to **recognize equivalent
graphs structures**, independently of their visual representation.

The related concept of **sub-graph isomorphism** is also important, as it
allows us to identify recurring structural patterns within larger graphs.

### 2.1.4. Betweenness of a Vertex

**Betweenness Centrality** measures how _frequently_ a vertex lies on the
**shortest paths** connecting other pairs of vertices.

Vertices with high betweenness centrality are considered **important connectors**
within the graph, as they often act as **bridges or bottlenecks**, connecting
different regions of a network.

Removing an high betweenness vertex can significantly disrupt the flow of
information or resources, but won't necessarily imply the disconnection of the graph.

### 2.1.5. Flow Networks

A **Flow Network** is a **directed graph** in which each edge has a **non-negative
capacity** representing the **_maximum amount of flow_ it can carry**.

This kind of graph introduces four main concepts:

- **Capacity**: The maximum flow that an edge can support.
- **Flow**: The actual amount of flow passing through an edge. It must be non-negative
  and not exceed its capacity.
- **Source**: A special vertex from which flow originates.
- **Sink**: A special vertex where flow is collected or consumed.

Flow conservation is a **specific but not mandatory requisite** typical to
ordinary vertex (it usually don't apply to _source_ and _sink_). It states
that the total flow entering a vertex must equal to the total outgoing flow.

### 2.1.6. Bipartite and Multi Graphs

In **Bipartite Graphs** there are _two distinct sets of vertices_,
and edges only connect vertices from different sets.

This can be used for modeling relationships between two different types of entities,
such as students and courses, or users and products.

<img class="" src="./images/graph-database/bipartite-graph.png"
      alt="Bipartite Graph">

Multi-Graphs are graphs that allow **multiple edges** between the same pair of vertices.
These may represent different types of relationships or interactions between the
same entities, each equipped with **different sets of properties**.

<img class="" src="./images/graph-database/multi-graph.png"
      alt="Multi Graph">

## 2.2. Using Graph Databases

Supposing we have the following UML class diagram:

<img class="" src="./images/graph-database/uml-class-diagram.png"
      alt="UML Class Diagram">

We now have to decide which model to use to represent the data:

- **Key-Value Database**: It would allow _fast access_ when the key is known, managing
  the relation through the code.
- **Document Database**: It stores aggregates of related attributes, using embedding
  and referencing model association between documents.
- **Graph Database**: It allows to _directly model relationships_ between entities,
  making _connection explicit_. This way it is possible for queries to follow
  patterns and paths between entities.

If the question we have to answer is not _"What are the attributes of X?"_,
but rather _"How is X connected to Y?"_ the best choice is to use a **Graph Database**.

The nomenclature of a graph database is perfect to confuse people:

- **Entity**: is an **_instance_** of a _class entity_ (conceptually a class
  object or a relational table row)
- **Vertex**: represents an entity, with some _properties_ (informations about it
  typically in JSON format)
- **Edge**: specifies a relation between two vertices, along with some properties
  (also in JSON format)

Graph databases are not suited for storing large amount of properties.
When using them we have to pay attention to insert only the **least amount of
useful properties** we want to immediately show to a user.

If in another moment an user wants more informations about one particular entity,
we can always query another database (relational, key-vaue, ...) to get the
rest of the properties.

There is one **crucial aspect** in doing this:

> **We must never make joins between a Graph Database and another type of database
> in one go**.

When designing a graph database, we have to focus on which relations are important
to the application.

Meaning that we should **avoid using undirected edges** when not necessary. This
makes us able to store informations in an easier and more efficient way.

<div class="grid2">
<div class="">

For example, the graph on the right is perfect to retrieve information such as:

- Which course does a specific student attend? (1 hop)
- Which topics are covered by a specific courses? (1 hop)
- Which topics are covered by courses attended by a specific student? (2 hops)

With edges directed as in the image it is for us very hard to efficiently
retrieve information such as:

- Which other students attend the same courses as a specific student?
- Which courses cover a specific topic?

</div>
<div class="">
<img class="80" src="./images/graph-database/graph-database-example.png"
      alt="Graph Database Example">
</div>
</div>

### 2.2.1. Graph vs Relational Database

In relational databases, the relations are typically reconstructed through
joins between rows and tables.

In this paradigm, we have a well-defined schema, integrity constraints and mature
query optimizers that could even make simple joins faster than graph traversals.

The problem shows up when we deal with deeper paths, where we might need repeated
joins or even recursion.

On the other hand, graph databases explicitly represents entities and the connections
between them, allowing for **efficient traversal** of deeper relations.

For example, if we want to _"list all the course a particular student attends"_:

- **Relational Mode**: we perform a join between the `Students` and `Courses`
  tables, which is efficient.
- **Graph Mode**: we start from the vertex representing the student and traverse
  the `attending` edges to reach the connected course vertices, which is also efficient.

### 2.2.2. Graph Oriented Design

When designing a graph database, we have to focus on the **relations** that
are important to the application, and not only just to the records.

The main things we must pay attention to are:

- **Neighborhood**: understand which are the adjacent nodes of a vertex, and how
  they are connected to each other.
- **Reachability**: determine which vertices are reachable from a given vertex,
  and how many hops it takes to reach them.
- **Multi-hop Traversal**: consider the paths that involve multiple hops, and
  how they can be efficiently traversed. This allows to understand similarity
  between vertices, and to find patterns in the graph.
- **Network Analysis**: analyze the overall structure of the graph, including
  its connectivity, clusters, and communities. This can help to identify
  important vertices and edges, and to understand the flow of information or
  resources through the network.

Graphs are able to represents many real-work workloads scenarios:

- **Social Networks**: Users are represented as vertices, and their relationships
  (friendships, follows, likes) are represented as edges. Graph databases can efficiently
  model and query social connections, enabling features like friend recommendations,
  community detection, and influence analysis.
- **Recommendation Systems**: Products, users, and their interactions can be modeled
  as a graph. Graph databases can help to identify similar users or products, enabling
  personalized recommendations based on user behavior and preferences.
- **Fraud Detection**: Graph databases can be used to detect fraudulent
  activities by analyzing the relationships between entities (e.g., users,
  transactions, accounts).
- **Dependency Analysis**: In software development, graph databases can model
  dependencies between components, libraries, or services. This can help to identify
  potential issues, optimize build processes, and manage complex systems.
- **Supply Chain Management**: Graph databases can model the relationships
  between suppliers, manufacturers, distributors, and retailers. This can help to
  optimize supply chain operations, identify bottlenecks, and improve overall efficiency.
