---
description: >-
  ObjectBox DB FAQ for Golang. Find answers to: How is ObjectBox different from
  BoltDB (bolt/bbolt) or Badger? Couldn't I just use JSON to store data? and
  many more questions.
---

# FAQ

## Couldn't I just use JSON to store data? (or anything file-based)

**Note:** The same also applies other simple file based approaches using CSV, XML, or object collections stored using binary serializations like Protocol Buffers, BSON, MessagePack, ...

It's perfectly fine to store data as JSON if you have a limited number of objects to manage. The more scalable approach to manage data, however, is to use a database like ObjectBox:

* ObjectBox uses a binary encoding which is faster
* Individual object changes: Let's say you have a list of objects and you change a single value. Using JSON, you typically write the entire list. ObjectBox touches a single object only.
* Random access: In ObjectBox you can get single objects efficiently without need to parse the entire JSON file.
* Memory efficiency: In close collaboration with the OS, ObjectBox can "page" through large data sets which would not fit into memory
* Queries & Indexing: ObjectBox comes with automatic indexing; which will drastically improve queries.
* ObjectBox offers ACID transactions to keep your data safe and consistent.

## Is ObjectBox ACID compliant? Is it an in-memory database?

ObjectBox comes with [ACID transactions](transactions.md). It has hard durability (the "D" in ACID) semantics. For synchronous transactions (e.g. what happens under the hood for Box.Put()), data is stored durable once it returns. Unlike ObjectBox, many NoSQL DBs have relaxed durability semantics  (e.g. a time window of second where data can be lost).

ObjectBox is not an in-memory database. The latter have high RAM requirements because they have to keep ALL data in memory. ObjectBox usage of RAM is flexible; it doesn't need much but makes use of RAM if it is available. That is why ObjectBox is usually as fast as an in-memory database.

## macOS: I'm getting "unexpected  signal" crashes when turning on "-trace"

We saw this happening using Go via `brew`. Using the latest Go version from the official download page resolved the issue.

## I'm getting "exit status 3221225781" on Windows

Exit status 3221225781 is a secret code :) for a missing DLL on windows. So even if your code has been compiled successfully, Windows can't find the DLL when launching the application (or tests). The DLL needs to be copied somewhere Windows will recognize it. See [Installation on Windows](install.md#objectbox-library-on-windows) for instructions.

## I'm getting errors during installation

You may encounter various seemingly unrelated errors when trying to install ObjectBox on a system without a C/C++ compiler (or when trying to cross-compile). These may be, for example:

* `... objectbox-go/objectbox/condition.go:21:14: undefined: QueryBuilder`
* `cannot find module for path github.com/objectbox/objectbox-go/internal/generator`
* `pkg/mod/github.com/objectbox/objectbox-go@v1.0.0/objectbox/model.go:30:2: cannot find package`

These errors are caused by Go failing to compile the package correctly. Because ObjectBox is using a native library, it requires CGO, which in turn needs a C/C++ compiler. Please make sure you have a working C/C++ compiler installed. See the [installation instructions](install.md#linux-macos) for further details.

{% hint style="info" %}
Trying to cross-compile ObjectBox may also be giving you similar errors. The reason is the same - not having the right C/C++ cross-compiler set up.&#x20;

Currently, cross-compilation is not tested/supported so please use native platform compilation for now and [upvote this GitHub issue](https://github.com/objectbox/objectbox-go/issues/18) if cross-compilation is important for you.
{% endhint %}

## How is ObjectBox different from BoltDB (bolt/bbolt) or Badger?

BoltDB and Badger are **key/value stores**. These are database primitives using bytes for keys and values. Many databases, including ObjectBox, build on top of a K/V layer to provide a higher level interface than "just bytes". One approach to do that is a separate ORM layer (like GORM, Storm, etc.). ObjectBox Go takes a slightly different approach, integrating both together to provide a concise and type-safe interface with great performance all in one package.

Therefore, we call ObjectBox an **object database**. You interact with it using objects; the same structs you use in your Go code. Just like that, no tearing apart for SQL whatsoever required.

Also, ObjectBox knows the "inside" of objects; and allows to query for struct fields ("properties"). It also manages indexing for you, you just have to [specify which properties should be indexed](entity-annotations.md#basic-annotations-for-entity-properties).
