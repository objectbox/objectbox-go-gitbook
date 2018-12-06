---
description: >-
  ObjectBox queries help you quickly find objects matching criteria you've
  specified.
---

# Queries

Using queries is simple: from your entity's `Box`, call `Query()` with conditions as arguments:

```go
query := box.Query(Device_.Location.HasPrefix("US-", false))
devices, err := query.Find()
```

### Building queries

The query in the code above uses a function `HasPrefix` on a device location. Where does this come from? ObjectBox generates a `Device_` struct for you to reference available properties conveniently. This also allows code completion in your IDE and avoids typos: correctness is checked at compile time \(string based queries would only be checked at run-time\).

Let's say you have the following entity defined in your package:

```go
type Device struct {
	Id       uint64
	Name     string
	Location string
	Profile  uint32
}
```

Using this input, the ObjectBox code generator creates a variable `Device_` in the same package:

```go
var Device_ = struct {
	Id       *objectbox.PropertyUint64
	Name     *objectbox.PropertyString
	Location *objectbox.PropertyString
	Profile  *objectbox.PropertyUint32
}{...}
```

You can use `Device_` to construct type-specific conditions in place and combining them, forming the full query. The following example looks for devices located in the U. S. with profile number 42.

```go
box.Query(Device_.Profile.Equals(42), Device_.Location.HasPrefix("US-", false))
```

{% hint style="info" %}
If you frequently call the same query, you should cache the built query variable and re-use it.
{% endhint %}

### Notable conditions/operators <a id="notable-conditions"></a>

In addition to expected conditions like `Equals()`, `NotEquals()`, `GreaterThan()` and `LessThan()` there are also conditions like:

* `Between()` to filter for values that are between the given two \(inclusive\)
* `In()` and `NotIn()` to filter for values that match any in the given set,
* `HasPrefix()`, `HasSuffix()` and `Contains()` for extended String filtering.

### Working with query results

You have a few options how to handle the results of a query:

* `Find()` returns a slice of the matching objects,
* `FindIds()`fetches just the IDs of the matching objects as a slice, which can be more efficient in case you don't need the whole object,
* `Remove()` deletes all the matching objects from the database \(in a single transaction\),
* `Count()` gives you the number of the objects that match the query,
* `Describe()` is a utility function which returns a human-readable representation of the query.

### More to come <a id="ordering-results"></a>

ObjectBox core can do much more with the queries, such as ordering, limits & offsets, setting parameters on reusable queries, aliases, etc. These are not yet supported by our Go API, but you can take a peek at [https://docs.objectbox.io/queries](https://docs.objectbox.io/queries) to get the idea what's coming in the future releases. 

Feel free to open a [feature request on GitHub](https://github.com/objectbox/objectbox-go/issues) with if you have an idea or a proposal.

