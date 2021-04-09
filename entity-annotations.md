---
description: How to persist objects with entity annotations in ObjectBox Go.
---

# Entity Annotations

## ObjectBox - Database Persistence with Entity Annotations <a id="objectbox-database-persistence-with-entity-annotations"></a>

ObjectBox is an object-oriented database persisting Go objects.  Sometimes we call those persist-able Go objects "**entities**" to distinguish from non-persisted objects. To let ObjectBox know which structs are entities you add go:generate command to their source file and annotations \(Go tags\) to some fields. Then ObjectBox can do its magic with your entities.

Here is an example:

```go
//go:generate go run github.com/objectbox/objectbox-go/cmd/objectbox-gogen

type Task struct {
	Id           uint64
	Text         string
	DateCreated  time.Time
	DateFinished time.Time
}
```

{% hint style="info" %}
When you run `go generate ./...` in your project, it finds all files with a `//go:generate` comment and executes the program in that comment, `objectbox-gogen`. This means you can include multiple entities \(structs\) inside a single file without repeating the comment. Having multiple entities in a single file may help increase generation performance on large projects because the generator can cache information about referenced types during runtime.
{% endhint %}

## id - Object IDs <a id="property-indexes-with-index"></a>

 In ObjectBox, every object has an ID of type `uint64` to efficiently get or reference objects. ObjectBox recognizes this automatically if your entity has a `uint64`field named ID \(case insensitive\). 

```go
type Task struct {
	Id uint64
}
```

Alternatively, you can use the `id` annotation on a `uint64` property with any name in your entity:

```go
type Group struct {
	GroupID uint64 `objectbox:"id"`
}
```

Note that in case this Id is zero on an instance you are inserting, ObjectBox considers the object as a new one during `Put()` and automatically assigns an ID.

If your application requires other ID types \(such as a string UID given by a server\), you can model them as standard properties. An example:

```go
type Task struct {
	Id  uint64 `objectbox:"id"`
	Uid string
}
```

### assignable - self-assigned IDs

When you put a new object you do not assign an ID. By default **IDs for new objects are assigned by ObjectBox**. If you **need to assign IDs by yourself**, use the `objectbox:"id(assignable)"` annotation. This will allow putting an object with any valid ID. You can still set the ID to zero to let ObjectBox auto-assign a new ID.

## index - property Indexes <a id="basic-annotations-for-entity-properties"></a>

Annotate a property with \`objectbox:"index"\` to create a database index for the corresponding database column. This can improve performance when querying for that property.

```go
type Task struct {
	Uid string `objectbox:"index"`
}
```

{% hint style="info" %}
Indexing is currently not supported for `byte[]`, `float` and `double`
{% endhint %}

### Index types \(String\) <a id="index-types-string"></a>

ObjectBox can use either actual **value** of the property or its **hash** to build an index. Because `string` properties are typically taking more space than scalar values, ObjectBox is using hash for strings by default.

You can instruct ObjectBox to use a different index type :

```go
type Task struct {
	Uid string `objectbox:"index:hash64"`
}
```

ObjectBox supports these index types:

* **Not specified** Uses best index based on property type \(uses `hash` for `string`, `value` for others\).
* **"value"** Uses property values to build index. For `String,` this may require more storage than a hash-based index.
* **"hash"** Uses 32-bit hash of property values to build index. Occasional collisions may occur which should not have any performance impact in practice. Usually a better choice than `hash64`, as it requires less storage.
* **"hash64"** Uses long hash of property values to build the index. Requires more storage than `hash` and thus should not be the first choice in most cases.

**Limits of hash-based indexes:** Hashes work great for equality checks, but not for **"starts with"** type conditions. If you frequently use those, you should use value-based indexes instead.

## unique - unique constraints  <a id="unique-constraints"></a>

Annotate a property with \`objectbox"unique"\` to enforce that values are unique before an entity is inserted/updated:

```go
type Task struct {
	Uid string `objectbox:"unique"`
}
```

A `put()` operation will abort and return an error if the unique constraint is violated.

## Complex fields \(structs\)

When your field contains another struct that is not a Relation, ObjectBox will, by default, store it embedded, using prefix-based field naming. The `Task` in the following example will internally end up with four fields, "id", "meta\_created", "meta\_modified" and "text". 

```go
// NOTE this is be placed in a separate field because it's not an entity
type Metadata struct {
	Created  int64
	Modified int64
}

// this is an Entity, with code genereated using objectbox-gogen
type Task struct {
    Id   uint64
    Meta Metadata
	Text string 
}
```

### inline

You can modify the behavior by specifying the struct fields to be "inlined" in which case, the fields would be named "id", "created", "modified" and "text". 

```go
type Task struct {
    Id   uint64
    Meta Metadata `objectbox:"inline"`
	Text string 
}
```

{% hint style="info" %}
It's important to keep distinction between prefixed and inlined fields in mind if you want to move fields around between two embedded/included structs or rename them.
{% endhint %}

## Other annotations <a id="basic-annotations-for-entity-properties"></a>

```go
// `objectbox:"uid:1306759095002958910"`
type Task struct {
	Text  string `objectbox:"name:text"`
	Date  uint64 `objectbox:"date"`
	notes string `objectbox:"-"`
	DateCreated  int64 `objectbox:"date uid:7144924247938981575"`
}
```

### converter

Defines converter for custom types, see [Custom types docs](custom-types.md) for more information.

### date

Informs ObjectBox that it should store the given property as a DateType - it expects timestamp since UNIX epoch in milliseconds.

If you use a `time.Time` field, it's automatically recognized as a date and the code generator will use the built-in converter and store the field internally as a Unix timestamp.

{% hint style="warning" %}
Because ObjectBox stores dates internally as Unix timestamps with **millisecond** precision, the built-in converter falls-back to that precision when working with `time.Time` struct. 

If you require greater precision, define  your own converter with any built-in supported type that can accommodate your storage format \(e.g. `int`, `string`, `[]byte`, etc.\).
{% endhint %}

### date-nano

Similar to `date` but storing the timestamp as nanoseconds since UNIX epoch.

### id-companion

To enable Time Series \(TS\) for an entity type, use the id-companion annotation on a `date` or `date-nano` field. TS enabled types require this special companion property. For more information about how to use ObjectBox TS \(Time Series\), please refer to the [C++ TS APIs](https://cpp.objectbox.io/time-series-data) for now.

### lazy

Specifies that the "To-Many" relation on the current field should not be called right away when the object is read but manually, using GetRelated. See [Relations docs](relations.md#to-many-relations) for more details.

### link

Declares the struct field that is itself a struct \(or a pointer to one\) as a relation, instructing ObjectBox to create a link between the Entity where this field is contained and the Entity of the field \(type of the struct field\). See [Relations docs](relations.md) for more details on how relations work and how you can define them.

### name 

Lets you define under what name the property is stored in the database. This allows you to rename the Go field without affecting the property name on the database level. To rename the property in the DB, you should use the [uid annotation ](entity-annotations.md#uid)instead.

### type

Used in conjunction with converter to specify the underlying type stored in the database \(type returned by the converter\). See [Custom types docs](custom-types.md) for more information.

### "-"

Marks properties that should not be persisted \(saved into DB\) and are only used in your program during runtime.

## Triggering generation <a id="triggering-generation"></a>

Once your entity schema is in place, you can trigger the code generation by running `go generate` inside the directory that contains the files with the entities.

### uid

ObjectBox keeps track of entities and properties by assigning them unique IDs \(UIDs\) during the code-generation phase. All those UIDs are stored in a file `objectbox-model.json` in your package, which you should add to your version control system \(e.g. git\). 

If you specify the \`objectbox"uid:....."\` tag on a property \(or as a special comment on an entity struct\), ObjectBox would be able to uniquely identify it even after you change the name and would update the database accordingly on the next application launch. 

For more information, look at the following page dedicated to schema updates.

{% page-ref page="schema-changes.md" %}

Additionally, If you are interested, we have [in-depth documentation on UIDs and concepts](https://docs.objectbox.io/advanced/meta-model-ids-and-uids) in the Java/Android docs. 

