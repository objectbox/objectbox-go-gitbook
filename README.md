# ObjectBox Go

This is the ObjectBox documentation for our Go API. We strive to provide you with the easiest and fastest solution to store and retrieve data. 

{% hint style="warning" %}
**Pre 1.0 note:** Until we hit 1.0, there still might be API changes. Please [open a GitHub issue](https://github.com/objectbox/objectbox-go/issues) if you have ideas how to improve the API.
{% endhint %}

Your feedback on ObjectBox and this documentation is very welcome. Use the "Was this page helpful?" smiley at the end of each page or send us your comments to contact\[at\]objectbox.io - thank you! :\)

## Changelog

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

