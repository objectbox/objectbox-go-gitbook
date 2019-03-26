---
description: >-
  Sometimes, you need to store a type that can't be handled out of the box. For
  such cases, you can define a custom converter (a pair of functions) that take
  care of encoding & decoding properties.
---

# Custom types

The following built-in types, their aliases and named types based on them are recognized as ObjectBox and stored as an appropriate internal type:

```go
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
bool
string, []string
byte, []byte
rune
float32, float64
```

## Defining a converter

To add support for a custom type, you can map properties to one of the built-in types using a `converter` annotation. 

For example, you could define a color in your entity using a custom `Color` struct and map it to an `int32`. Or you can map the `time.Time` to an `int64`, though losing some precision - less than a millisecond, i. e. a thousandth of a second\):

```go
type Task struct {
	Id          uint64
	Text        string
	DateCreated time.Time  `date type:"int64" converter:"timeInt64"`
}
```

In the entity definition above, we instruct ObjectBox to store the `DateCreated` field as a `int64` while converting it to/from `time.Time` when using in the program. ObjectBox will generate a binding code that will call the following two functions \(both start with the prefix `timeInt64` specified above\):

```go
// from DB value to runtime value
func timeInt64ToEntityProperty(dbValue int64) time.Time

// from runtime value to DB value
func timeInt64ToDatabaseValue(goValue time.Time) int64
```

Just to complete the example, those functions could be implemented like this:

```go
// converts Unix timestamp in milliseconds (ObjectBox date field format) to time.Time
func timeInt64ToEntityProperty(dbValue int64) (goValue time.Time) {
	return time.Unix(dbValue/1000, dbValue%1000*1000000)
}

// converts time.Time to Unix timestamp in milliseconds 
// i. e. internal format expected by ObjectBox on a date field
func timeInt64ToDatabaseValue(goValue time.Time) int64 {
	var ms = int64(goValue.Nanosecond()) / 1000000
	return goValue.Unix()*1000 + ms
}
```

## Things to look out for

You must **not interact with the database** \(such as using `Box` or `ObjectBox`\) inside the converter. The converter methods are called within a transaction, so for example getting or putting entities to a box will fail.

Your converter implementation must be **thread safe** as it can be called from multiple go routines in parallel. Try to avoid using global variables.

`Query` is unaware of custom types. You have to **use the primitive DB type for queries**.

