# Entity Annotations

## ObjectBox - Database Persistence with Entity Annotations <a id="objectbox-database-persistence-with-entity-annotations"></a>

ObjectBox is a database that persists objects. For a clear distinction, we sometimes call those persistable objects **entities**. To let ObjectBox know which structs are entities you add go:generate command to their source file and annotations \(Go tags\) to some fields. Then ObjectBox can do its magic with your entities.

Here is an example:

```go
//go:generate objectbox-gogen

type Task struct {
	Id           uint64 `id`
	Text         string
	DateCreated  int64
	DateFinished int64
}
```

## \`id\` - Object IDs <a id="property-indexes-with-index"></a>

 In ObjectBox, every object has an ID of type `uint64` to efficiently get or reference objects. ObjectBox recognizes this automatically if your entity has a `uint64`field named ID \(case insensitive\). 

```go
type Task struct {
	Id uint64
}
```

Alternatively, you can use the `id` annotation on a `uint64` property with any name in your entity:

```go
type Group struct {
	GroupID uint64 `id`
}
```

Note that in case this Id is zero on an instance you are inserting, ObjectBox considers the object as a new one during `Put()` and automatically assigns an ID.

If your application requires other ID types \(such as a string UID given by a server\), you can model them as standard properties. An example:

```go
type Task struct {
	Id  uint64 `id`
	Uid string
}
```

## \`index\` - property Indexes <a id="basic-annotations-for-entity-properties"></a>

Annotate a property with \`index\` to create a database index for the corresponding database column. This can improve performance when querying for that property.

```go
type Task struct {
	Uid string `index`
}
```

{% hint style="info" %}
\`index\` is currently not supported for `byte[]`, `float` and `double`
{% endhint %}

### Index types \(String\) <a id="index-types-string"></a>

ObjectBox can use either actual **value** of the property or its **hash** to build an index. Because `string` properties are typically taking more space than scalar values, ObjectBox is using hash for strings by default.

You can instruct ObjectBox to use a different index type :

```go
type Task struct {
	Uid string `index:"hash64"`
}
```

ObjectBox supports these index types:

* **Not specified** Uses best index based on property type \(HASH for `string`, VALUE for others\).
* **"value"** Uses property values to build index. For `String,` this may require more storage than a hash-based index.
* **"hash"** Uses 32-bit hash of property values to build index. Occasional collisions may occur which should not have any performance impact in practice. Usually a better choice than HASH64, as it requires less storage.
* **"hash64"** Uses long hash of property values to build the index. Requires more storage than HASH and thus should not be the first choice in most cases.

**Limits of hash-based indexes:** Hashes work great for equality checks, but not for **"starts with"** type conditions. If you frequently use those, you should use value-based indexes instead.

## \`unique\` - unique constraints  <a id="unique-constraints"></a>

Annotate a property with \`unique\` to enforce that values are unique before an entity is inserted/updated:

```go
type Task struct {
	Uid string `unique`
}
```

A `put()` operation will abort and return an error if the unique constraint is violated.

## Other annotations <a id="basic-annotations-for-entity-properties"></a>

```go
type Task struct {
	Text  string `nameInDb:"text"`
	Date  uint64 `date`
	notes string `transient`
}
```

### \`nameInDb\` 

Lets you define under what name the property is stored in the database. This allows you to rename the Go field without affecting the property name on the database level.

### \`transient\`

Marks properties that should not be persisted \(saved into DB\) and are only used in your program during runtime.

### \`date\`

Informs the ObjectBox that it should store the given property as a DateType - it expects timestamp since UNIX epoch in milliseconds.

## Triggering generation <a id="triggering-generation"></a>

Once your entity schema is in place, you can trigger the code generation by running `go generate` inside the directory that contains the files with the entities.

