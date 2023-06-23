---
description: >-
  ObjectBox Go is a lean NoSQL Golang database for persisting objects. It is
  designed to give you high performance and easy native Go APIs on any
  POSIX-system and embedded devices.
---

# Golang Database

Your opinion matters to us! To make ObjectBox better for our users, we have set up an [Anonymous Feedback Form](https://forms.gle/bdktGBUmL4m48ruj7). Please do fill this in (it only takes 2 minutes). Every response is highly appreciated. To rate this documentation, you can use the "Was this page helpful?" smiley at the end of each page.

Otherwise, feel free to open an [issue on GitHub](https://github.com/objectbox/objectbox-go/issues) or send us your comments to contact\[at]objectbox.io - Thank you! - and if you like what you see, we also appreciate a shoutout :)

## Changelog

### v1.7.0 (2023-06-23)

* Update [objectbox-c to v0.18.1](https://cpp.objectbox.io/#0.18.1-2023-01-30) bringing many improvements to Go

### v1.6.1 (2022-01-27)

* Improved version check for the dynamically loaded C lib
* Update to objectbox-c v0.15.1 which contains fixes and minor performance improvements

### v1.6.0 (2022-01-14)

* Update [objectbox-c to v0.15.0](https://cpp.objectbox.io/#v0.15.0-2021-12-09) bringing many improvements to Go

### v1.5.0 (2021-08-18)

* update objectbox-c to v0.14.0
* add [PropertyQuery](queries.md#propertyquery) support

### v1.4.0 (2021-04-01)

* add TimeSeries model definition support using `id-companion` and `date-nano` annotations
* avoid `time.Duration.Milliseconds()` not available on old Go Versions
* `NewSyncClient` - flip return values to align with the other "constructors"
* add `NanoTimeInt64*` built-in converters

### v1.3.0 (2021-03-19)

* add [ObjectBox Sync](https://objectbox.io/sync/) client support
* add self-assignable IDs: `objectbox:"id(assignable)"`
* add query `GreaterOrEqual`/`LessOrEqual` for ints and floats
* update objectbox-generator to v0.12.0
* update objectbox-c to v 0.13.0
* fix compiling on old gcc (e.g. the one in CentOS 7)

### v1.2.0 (2020-08-25)

* update to objectbox-c v0.10.0 with latest improvements and fixes
* support comma in addition to space as an annotation separator
* extract code generation into a separate module/project [objectbox-generator](https://app.gitbook.com/s/-LR89ifsSca2Mcwcn53Q/github.com/objectbox/objectbox-generator) and depend on it to preserve existing `go:generate` annotations

### v1.1.2 (2020-03-18)

* ensure Query finalizer is only executed by Go GC after a native call finishes

### v1.1.1 (2020-02-14)

* use temp directories in tests to prevent failure in recent Go versions checking-out modules as read-only

### v1.1.0 (2019-12-16)

* add Box `Insert` and `Update` methods with stricter semantics than `Put`
* add AsyncBox with `Put`, `Insert`, `Update`, `Remove`
* add Query order and parameter alias support - see [Queries](queries.md) docs for more info
* Code generator improvements
  * handle type-checker errors more gracefully (don't fail on failures in unneeded imports)&#x20;
  * add `clean` command-line option to remove all generated files
  * `time.Time` will automatically use a built in converter to Unix timestamp (milliseconds)
  * improve model.json file diff-level compatibility with other language bindings
  * embedded struct and to-one relations cycle detection
  * support changing property type and resetting its stored value
  * nil check for embedded pointer structs in the generated code
  * minor bug fixes when the generated code wouldn't compile in some edge cases
* update to the latest ObjectBox-C library v0.8.1
* deprecate `box.PutAsync()` in favor of `box.async().Put()` i.e. using AsyncBox
* mark `byte` properties as unsigned
* fix getters on objects with missing relations and non-existent IDs in `GetMany`
* better windows installation experience using a PowerShell script
* make `golint` happier :)

### v1.0.0 (2019-07-16)

This is quite a big release, bringing some new features and cleaning up the API.

* explicit transaction support via `ObjectBox::RunInReadTx` and `ObjectBox::RunInWriteTx`
* Go Modules support
* add `objectbox` namespace to tags to align with `reflect.StructTag.Get` unofficial spec
* optional lazy loading on to-many relations - `lazy` annotation
* box additions:
  * `GetMany` , `RemoveMany`, `RemoveIds`, `ContainsIds`, `RemoveIds`
  * to-many relation auxiliary methods: `RelationIds` , `RelationPut`, `RelationRemove`, `RelationReplace`
* switch default/recommended `go:generate` entity generator command from `objectbox-gogen` to  `//go:generate go run github.com/objectbox/objectbox-go/cmd/objectbox-gogen`
* quite a few internal changes, renames and other refactorings (e.g. renamed `PutAll` to `PutMany`, removed `Cursor`, aligned model JSON with other bindings, ...)

### v0.9.0 (2019-04-24)

* Fixed macOS build and 32-bit query support.
* Minor refactoring/linter issues

### v0.9.0-rc (2019-02-22)

As we queued up quite a few changes, we're doing a release candidate first for you to test:

* Improved relations support
* Embedded structs
* Custom [value-converter ](custom-types.md)to store unsupported types & structs that can't be inlined/prefixed
* Recognize and handle type aliases and named types used as entity fields
* New Box methods: CountMax(), IsEmpty(), Contains() and PutAsyncWithTimeout()
* Query support AND & OR (combine conditions)
* New Query methods: [Limit, Offset](queries.md#limit-offset-and-pagination)
* New Query methods: [Set\*Params (type-based) ](queries.md#reusing-queries-and-parameters)- to run a cached query with custom  parameters
* Query LT|GT|Between support for unsigned numbers
* Support numeric string ID in the entity
* Support for`[]string` as a field type
* Optional pass/return-by-value for slice-of-structs in the generated code
* Change strings to use hash-based instead of value-based indexes by default
* A new option to AlwaysAwaitAsync that can be enabled during initialization

### v0.8.0 (2018-12-06)

* New [Query API](queries.md)
* Box and Query do not require manual closing anymore
* Support for [renaming entities and their properties](schema-changes.md) using UIDs

### v0.7.1 (2018-11-30)

* Fixed wrong mapping for Go types (u)int and (u)int8. Luckily we noticed this very early: if you used those types in previous versions, please delete old database files.
* Transactions are now safely aborted in case of panics

### v0.7.0 (2018-11-29)

* Changed file name of generated code, e.g. file endings for model is now ".obx.go"
* Foundation for all query conditions (final query API will come with the next version)
* Put(object) now assigns the new id to the object itself

### v0.6.0 (2018-11-28)

Initial public release
