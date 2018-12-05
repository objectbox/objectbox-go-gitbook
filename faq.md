---
description: Frequently asked questions about ObjectBox Go.
---

# FAQ

## How is ObjectBox different from BoltDB \(bolt/bbolt\) or Badger?

BoltDB and Badger are **key/value stores**. These are database primitives using bytes for keys and values. Many databases, including ObjectBox, build on top of a K/V layer to provide a higher level interface than "just bytes".

ObjectBox is an **object database**. You interact with it using objects; the same structs you use in your Go code. Just like that, no tearing apart for SQL whatsoever required.

Also, ObjectBox knows the "inside" of objects; and allows to query for struct fields \("properties"\). It also manages indexing for you, you just have to [specify which properties should be indexed](entity-annotations.md#basic-annotations-for-entity-properties).

## Couldn't I just use JSON to store data? \(or anything file-based\)

**Note:** The same also applies other simple file based approaches using CSV, XML, or object collections  stored using binary serializations like Protocol Buffers, BSON, MessagePack, ...

It's perfectly fine to store data as JSON if you have a limited number of objects to manage. The more scalable approach to manage data, however, is to use a database like ObjectBox:

* ObjectBox uses a binary encoding which is faster
* Individual object changes: Let's say you have a list of objects and you change a single value. Using JSON, you typically write the entire list. ObjectBox touches a single object only.
* Random access: In ObjectBox you can get single objects efficiently without need to parse the entire JSON file.
* Memory efficiency: In close collaboration with the OS, ObjectBox can "page" through large data sets which would not fit into memory
* Queries & Indexing: ObjectBox comes with automatic indexing; which will drastically improve queries.
* ObjectBox offers ACID transactions to keep your data safe and consistent.

## Is ObjectBox ACID compliant? Is it an in-memory database?

ObjectBox comes with [ACID transactions](transactions.md). It has hard durability \(the "D" in ACID\) semantics. For synchronous transactions \(e.g. what happens under the hood for Box.Put\(\)\), data is stored durable once it returns. Unlike ObjectBox, many NoSQL DBs have relaxed durability semantics  \(e.g. a time window of second where data can be lost\).

ObjectBox is not an in-memory database. The latter have high RAM requirements because they have to keep ALL data in memory. ObjectBox usage of RAM is flexible; it doesn't need much but makes use of RAM if it is available. That is why ObjectBox is usually as fast as an in-memory database.  






