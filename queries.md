---
description: >-
  Easily build Go queries in ObjectBox with the ObjectBox builder APIs.
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

### Reusing Queries and Parameters <a id="reusing-queries-and-parameters"></a>

If you frequently run a `Query` you should cache the `Query` object and re-use it. To make a `Query` more reusable you can change the values, or query parameters, of each condition you added even after the `Query` is built. Let's see how.

Assume we want to find a list of `User` with specific `FirstName` values. First, we build a regular `Query` with an `equal()` condition for `FirstName`. Because we have to pass an initial parameter value to `equal()` but plan to override it before running the `Query` later, we just pass an empty string:

```go
var caseSensitive = false
var query = box.Query(User_.FirstName.Equals("", caseSensitive))
```

Now at some later point we want to actually run the `Query`. To set a value for the `FirstName` parameter we call `setStringParams()` on the `Query` and pass the `FirstName` property and the new parameter value:

```go
query.SetStringParams(User_.FirstName, "Joe")
joes, _ := query.Find()
```

### Alias/As <a id="alias"></a>

So you might already be wondering, what happens if you have more than one condition using the same property? For this purpose you can **assign each condition an alias** by calling `Alias()` right after specifying the condition:

```go
var query = box.Query(
		User_.Age.GreaterThan().Alias("min age"),
		User_.Age.LessThan().Alias("max age"))
		
// Then use the alias when setting the parameter value
query.SetInt64Params(objectbox.Alias("min age"), 50)
query.SetInt64Params(objectbox.Alias("max age"), 100)
```

There's also an alternative, syntax for a aliases that makes it easier to maintain the code because it avoids repeating string constants:

```go
var minAgeAlias = objectbox.Alias("min age")
var maxAgeAlias = objectbox.Alias("max age")
var query = box.Query(
		User_.Age.GreaterThan().As(minAgeAlias),
		User_.Age.LessThan().As(maxAgeAlias))
		
// Then use the alias when setting the parameter value
query.SetInt64Params(minAgeAlias, 50)
query.SetInt64Params(maxAgeAlias, 100)
```

### Limit, Offset, and Pagination <a id="limit-offset"></a>

Sometimes you only need a subset of a query, for example the first 10 elements. This is especially helpful \(and resourceful\) when you have a high number of entities and you cannot limit the result using query conditions only. The built `Query` has  `.Offset()` and `.Limit()` methods to help you do that

```go
query := box.Query(User_.FirstName.Equals("Joe", false))
joes, err := query.Offset(10).Limit(5).Find()
```

`Offset(n uint64):` the first `n` results are skipped.  
`Limit(n uint64):` at most `n` results of this query are returned.

### Ordering results <a id="notable-conditions"></a>

In addition to specifying conditions you can order the returned results:

```go
query := box.Query(User_.FirstName.Equals("Joe", false), User_.Age.OrderDesc())
joes, err := query.Find()
```

You can combine multiple order parameters and options \(some options are only available for certain data types, e.g. strings have case-sensitive ordering option\), such as:

```go
query := box.Query(
    User_.FirstName.Equals("Joe", false), 
    User_.LastName.OrderDesc(false), // caseSensitive bool argument
    User_.Age.OrderAsc()
)
joes, err := query.Find()
```

### Notable conditions/operators

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
* `Limit()` and `Offset()` let you select just part of the result \(e. g. for paging\)
* `DescribeParams()` is a utility function which returns a human-readable representation of the query.

### Querying linked objects \(relations\) <a id="ordering-results"></a>

After creating a relation between entities, you might want to add a query condition for a property that only exists in the related entity. In SQL this is solved using JOINs. ObjectBox provides query links instead.   
Let's see how this works using an example.

Assume there is a `Person` that can be associated with multiple `Address` entities:

```go
//go:generate go run github.com/objectbox/objectbox-go/cmd/objectbox-gogen

type Person struct {
	Id       uint64
	Name     string
	Address  []*Address
}

type Address struct {
	Id     uint64
	Street string
	ZIP    string
}
```

To get a `Person` with a certain name that also lives on a specific street, we need to query the associated `Address` entities of a `Person`. To do this, use the `Person_.Address.Link(cs ...Conditions)` method of the generated `Person_` variable to tell that the `addresses` relation should be queried and what conditions should be used to filter the addresses:

```go
// get all Person objects named "Elmo" which have an address on "Sesame Street"
var query = BoxForPerson(ob).Query(
	Person_.name.Equals("Elmo", true),
	Person_.Address.Link(Address_.Street.Equals("Sesame Street", true)),
)
var elmosOnSesameStreet = query.Find()
```

What if we want to get a list of `Address` instead of `Person`? No problem, links are smart enough to know there's also an implicit relation in the opposite direction. Note the different `box` we're using here:

```go
// get all Address objects on "Sesame Street" linked from a Person named "Elmo"
val builder = box.query().equal(Address_.Street, "Sesame Street")
var query = BoxForAddress(ob).Query(
    Address_.Street.Equals("Sesame Street", true),
	Person_.Address.Link(Person_.Name.Equals("Elmo", true)),
)
var addressesSesameStreetWithElmo = query.Find()
```

## PropertyQuery

If you only want to return the values of a particular property and not a list of full objects you can use a [PropertyQuery](https://pkg.go.dev/github.com/objectbox/objectbox-go/objectbox#PropertyQuery). After building a query, simply call `query.Property(Property)` . For example, instead of getting all Users, to just get their email addresses:

```go
query := userBox.Query()
emails, err := query.Property(User_.Email).FindStrings(nil)
```

{% hint style="warning" %}
The returned items are **not in any particular order**, even if you did specify an order when building the query.
{% endhint %}

### Handling null values

The argument to `FindStrings()` \(and similar for other types\) is a value to be used if the given field is `nil` in the database. **By default, i.e. when you pass \`nil\` as the argument, these values are not returned.** However, you can specify a replacement value to return if a property is null:

```go
// includes 'unknown' instead of each null email
emails, err := userBox.Query().Property(User_.Email).FindStrings("unknown")
```

### Distinct values

The property query can also only return distinct values:

```go
pq := userBox.Query().Property(User_.FirstName)

// returns ['joe'] because by default, the case of strings is ignored.
err := pq.Distinct(true) // args: Distinct(value bool)
names := pq.FindStrings(nil)

// returns ['Joe', 'joe', 'JOE']
pq.DistinctString(true, true) // args: DistinctString(value, caseSensitive bool)
names = pq.FindStrings(nil)
```

### Aggregating values

Property queries also offer aggregate functions to directly calculate the minimum, maximum, average, sum and count of all found \(non-null\) values:

* `Min()` / `MinDouble()`: Finds the minimum value for the given property over all objects matching the query.
* `Max()` / `MaxFloat64()`: Finds the maximum value.
* `Sum()` / `SumFloat64()`: Calculates the sum of all values. _Note: the integer version detects overflows and returns an error in that case._
* `Average()` : Calculates the average \(always a `float64`\) of all values.
* `Count()`: returns the number of results. This is faster than finding and getting the length of the result array. Can be combined with `Distinct()` to count only the number of distinct values.

