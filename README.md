# ObjectBox Go

This is the ObjectBox documentation for our Go API. We strive to provide you with the easiest and fastest solution to store and retrieve data. 

{% hint style="warning" %}
**Pre 1.0 note:** Until we hit 1.0, there still might be API changes. Please [open a GitHub issue](https://github.com/objectbox/objectbox-go/issues) if you have ideas how to improve the API.
{% endhint %}

Your feedback on ObjectBox and this documentation is very welcome. Use the "Was this page helpful?" smiley at the end of each page or send us your comments to contact\[at\]objectbox.io - thank you! :\)

## Changelog

### v0.9.0-rc \(2019-02-22\)

As we queued up quite a few changes, we're doing a release candidate first for you to test:

* Improved relations support
* Embedded structs
* Custom [value-converter ](custom-types.md)to store unsupported types & structs that can't be inlined/prefixed
* Recognize and handle type aliases and named types used as entity fields
* New Box methods: CountMax\(\), IsEmpty\(\), Contains\(\) and PutAsyncWithTimeout\(\)
* Query support AND & OR \(combine conditions\)
* New Query methods: [Limit, Offset](queries.md#limit-offset-and-pagination)
* New Query methods: [Set\*Params \(type-based\) ](queries.md#reusing-queries-and-parameters)- to run a cached query with custom  parameters
* Query LT\|GT\|Between support for unsigned numbers
* Support numeric string ID in the entity
* Support for`[]string` as a field type
* Optional pass/return-by-value for slice-of-structs in the generated code
* Change strings to use hash-based instead of value-based indexes by default
* A new option to AlwaysAwaitAsync that can be enabled during initialization

### v0.8.0 \(2018-12-06\)

* New [Query API](queries.md)
* Box and Query do not require manual closing anymore
* Support for [renaming entities and their properties](schema-changes.md) using UIDs

### v0.7.1 \(2018-11-30\)

* Fixed wrong mapping for Go types \(u\)int and \(u\)int8. Luckily we noticed this very early: if you used those types in previous versions, please delete old database files.
* Transactions are now safely aborted in case of panics

### v0.7.0 \(2018-11-29\)

* Changed file name of generated code, e.g. file endings for model is now ".obx.go"
* Foundation for all query conditions \(final query API will come with the next version\)
* Put\(object\) now assigns the new id to the object itself

### v0.6.0 \(2018-11-28\)

Initial public release

