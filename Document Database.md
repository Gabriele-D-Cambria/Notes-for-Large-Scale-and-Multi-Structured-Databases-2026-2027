---
title: Document Database
---

# 1. Index

- [1. Index](#1-index)
- [2. Document Database](#2-document-database)
  - [2.1. XML Documents](#21-xml-documents)
    - [2.1.1. XML Ecosystem](#211-xml-ecosystem)
    - [2.1.2. XML Databases](#212-xml-databases)
    - [2.1.3. Drawbacks](#213-drawbacks)
  - [2.2. JSON Documents](#22-json-documents)
    - [2.2.1. JSON Databases](#221-json-databases)

# 2. Document Database

Document Databases are **non-relational** databases that store data
as structured documents, typically in formats like `XML` or `JSON`.

They ensure **high flexibility**, allowing to handle _complex data
structures_.

Like _key-value databases_, document databases are **schema-less**.
Unlike _key-value databases_, they allow **complex operations**
such as _queries_ and _filtering_, with some even allowing
**ACID transactions**.

## 2.1. XML Documents

`XML` stands for _eXtensible Markup Language_. Like all markup
language the documents content follow a specific **structure**.

Specifically, `XML` uses **tags** to provide _extra information_
giving a reader more information on what the _data means_.

Its capable of representing almost **any form of information**
in a structured way.

Along with `CSS` (_Cascading Style Sheets_) it allowed **second
generation websites** to separate _content_ from _presentation_.

Being based on _Unicode Text_, it is the basis for many data
**interchange protocols**, such as `SOAP` (_Simple Object
Access Protocol_) to which it was a foundation, because
of its efficiency in transmission.

Today it is a **standard** also for many document types,
including word processing documents and spreadsheets (`docx`,
`xlsx` and `pptx` are all `XML` based).

Other advantages are:

- **Platform independence**: One `XML` document can be displayed in
  differently in different media and software platform.
- **Modularization**: `XML` documents can be easily split into smaller
  parts and recombined.

An example of an `XML` document is:

```xml
<songs>
  <item>
    <title>Kind of Blue</title>
    <artist>Miles Davis</artist>
    <tracks>
      <track length="9:22">So What</track>
      <track length="9:22">Freddie Freeloader</track>
      <track length="5:37">Blue in Green</track>
      <track length="11:33">All Blues</track>
      <track length="7:27">Flamenco Sketches</track>
    </tracks>
  </item>
  <item>
    <title>Blue Train</title>
    <artist>John Coltrane</artist>
    <tracks>
      <track length="10:43">Blue Train</track>
      <track length="9:21">Moment's Notice</track>
      <track length="7:15">Locomotion</track>
      <track length="8:43">I'm Old Fashioned</track>
      <track length="11:12">Lazy Bird</track>
    </tracks>
  </item>
</songs>
```

### 2.1.1. XML Ecosystem

When dealing with `XML` documents there are several things to consider.

The first is the **XML Schema**. It defines the structure of an XML document,
including the elements and attributes that can appear in it, their data types,
and the relationships between them. It ensures that XML documents adhere to a
specific format and can be validated against the schema.

```xml
<xs:element name="book">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="title" type="xs:string"/>
      <xs:element name="author" type="xs:string"/>
      <xs:element name="year" type="xs:gYear"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```

Another thing is **XSLT**, a language used to transform XML documents into other
formats, such as `HTML` or `PDF`. It allows developers to create templates
that define how the data in an XML document should be presented.

```xml
<xsl:template match="book">
  <div>
    <h1><xsl:value-of select="title"/></h1>
    <p>Author: <xsl:value-of select="author"/></p>
    <p>Year: <xsl:value-of select="year"/></p>
  </div>
</xsl:template>
```

When we want to navigate through elements and attributes
in an XML document, the syntax used is called **XPath**:

```xpath
/book/title
/book/author
//book[@id='b1']
/book/chapter[1]/title
```

We can query this language using the `XQuery` language:

```xquery
for $b in /books/book
where $b/year > 2010
return $b/title
```

### 2.1.2. XML Databases

The tools described above are used to create and interact with **XML
Databases** from the _client layer_.

They interact with the _processing layer_, that parses the documents
and executes the queries, and the _storage layer_, that stores the
actual documents creating and updating _indexes_, for text search, and _Document
Object Model_ (_DOM_), for a tree-based representation.

RDBMSs did not get replaced by XML databases, instead they started supporting them
by allowing the storage of XML documents in _BLOB_ columns.

Their typical use cases are:

- **Content Management**: Store and organize documents (e.g., articles,
  reports, manuals) in a structured way.
- **Data Interchange**: Exchange structured data (e.g., SOAP web services)
- **Enterprise Documents**: Store documents (e.g., docx, xlsx, pptx)
- **Integration**: Use `XML` with existing RDBMSs to store and query
  structured data.

### 2.1.3. Drawbacks

`XML` is very powerful and flexible, but it comes with some drawbacks.

The first is that it is **verbose** and **repetitious**, thus increasing the amount
of storage required. This property makes `XML` _documents_ **computationally
expensive to parse** and **wasteful of space**.

In general, they are used as **content-management systems**, for example as a
collection of text files. All the other possible usage got replaced by other formats,
like `JSON` or `YAML`, since they are more compact and easier to parse.

## 2.2. JSON Documents

`JSON` stands for _JavaScript Object Notation_. It is a lightweight data-interchange
format that is easy for humans to read and write, and easy for machines to
parse and generate.

Thanks to their integration with `JavaScript`, `JSON` documents have often been
preferred over `XML` documents, especially for **data interchanging** in web
applications.

Despite its name, `JSON` is (mostly) a language-independent way of specifying
objects as _name-value_ pairs.

```json
{
  "skillz": {
    "web": [
      {
        "name": "html",
        "years": 5
      },
      {
        "name": "css",
        "years": 3
      }
    ],
    "database": [
      {
        "name": "sql",
        "years": 7
      }
    ]
  }
}
```

An **object** is an _unordered set of name/value pairs_:

- The pairs are enclosed within braces `{}`
- There is a colon `:` between the name and the value
- Pairs are separated by commas `,`

An **array** is an _ordered collection of values_:

- The pairs are enclosed within brackets `[]`
- Values are separated by commas `,`

`JSON` values can be `strings`, `numbers`, `objects`, `arrays`, `boolean`, or `null`.

If we want to compare `JSON` with `XML`, we can say that they both:

- Are human and machine readable
- Are hierarchical
- Are language independent
- Are supported in APIs of many programming languages
- Have simple syntax

As for the differences, beside having a different syntax, `JSON` is
_less verbose_ and _easier to parse_, including _arrays_.

For example, we can compare the two formats representing the same data:

<div class="grid2">
<div class="">

```xml
<users>
  <user>
    <username>SammyShark</username>
    <location>Indian Ocean</location>
  </user>
  <user>
    <username>PollyParrot</username>
    <location>Caribbean Sea</location>
  </user>
  <user>
    <username>WallyWalrus</username>
    <location>Arctic Ocean</location>
  </user>
  <user>
    <username>BettyBeluga</username>
    <location>Atlantic Ocean</location>
  </user>
</users>
```

</div>
<div class="">

```json
{
  "users": [
    {
      "username": "SammyShark",
      "location": "Indian Ocean"
    },
    {
      "username": "PollyParrot",
      "location": "Caribbean Sea"
    },
    {
      "username": "WallyWalrus",
      "location": "Arctic Ocean"
    },
    {
      "username": "BettyBeluga",
      "location": "Atlantic Ocean"
    }
  ]
}
```

</div>
</div>

From this example we can compute that the XML document size is `416 bytes`,
while the `JSON` document size is `273 bytes`, which is a $\sim 34\%$ reduction.

One `JSON` drawback is that the names **must not be** `JavaScript`
reserved words, while `XML` allows any name.

### 2.2.1. JSON Databases

A `JSON` **document** is the basic unit of storage.
It includes one or more **key-value pairs**, and may also contain
_nested documents_ and/or _arrays_.

```json
{
  "id": 123,
  "name": "Alice",
  "address": {
    "city": "Wonderland",
    "zip": "12345"
  },
  "phones": ["123-456-7890", "987-654-3210"],
  "email": "alice@ex.com",
  "age": 30,
  "active": true
}
```

_Arrays_ as well can contain simple values or nested documents.

We call **Collection** a set of documents sharing some _common purpose_, even
if they have different fields and structures.
