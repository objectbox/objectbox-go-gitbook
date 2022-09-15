---
description: >-
  ObjectBox manages its data model (schema) mostly automatically. ObjectBox db
  supports automatic schema migration to make data persistence as easy as
  possible for application developers.
---

# Schema changes

The data model is defined by the entity structs you define. When you **add or remove** entities or properties of your entities, **ObjectBox takes care** of those changes without any further action from you.

For other changes like **renaming or changing the type**, ObjectBox needs **extra information** to make things unambiguous. This works using unique identifiers (UIDs) specified by the [uid annotation](entity-annotations.md#uid), as we will see below.

## Renaming Entities and Properties <a href="#renaming-entities-and-properties" id="renaming-entities-and-properties"></a>

So why do we need that UID annotation? If you simply rename an entity struct, ObjectBox only sees that the old entity is gone and a new entity is available. This can be interpreted in two ways:

* The old entity is removed and a new entity should be added, the old data is discarded. This is the **default behavior** of ObjectBox.
* The entity was renamed, the old data should be re-used.

So to tell ObjectBox to do a rename instead of discarding your old entity and data, you need to make sure it knows that this is the same entity and not a new one. You do that by attaching the internal UID to the entity.

For properties, the process is the same, but instead of the comment, you just use standard Go tags. We are showing both cases in the following example.

&#x20;**Step 1:** Add an empty \`objectbox:"uid"\` annotation to the entity/property you want to rename:

```go
// `objectbox:"uid"`
type OldEntityName struct {
	Id  uint64
}

type Task struct {
	Id  uint64
	OldPropertyName string `objectbox:"uid"`
}
```

&#x20;**Step 2:** Re-generate ObjectBox code for the project using `go generate ./...` in your project directory. The generation will fail with an error message that gives you the current UID of the entity/property:

{% code title="output for empty "uid" annotation on an entity" %}
```
can't merge binding model information: uid annotation value must not be empty 
(model entity UID = 1306759095002958910) on entity OldEntityName
```
{% endcode %}

{% code title="output for empty "uid" annotation on a property" %}
```
can't merge binding model information: uid annotation value must not be empty on property OldPropertyName, entity Task:
    [rename] apply the current UID 9141374017424160113
    [change/reset] apply a new UID 6050128673802995827
```
{% endcode %}

{% hint style="info" %}
Note how for a property, the output is slightly different and, besides support for renaming, it provides a newly generated UID you can use to effectively reset (clean) the stored data on the property. See [Reset data - new UID on a property](schema-changes.md#reset-data-new-uid-on-the-property) for more details.
{% endhint %}

&#x20;**Step 3:** Apply the UID printed in the error message to your entity/property:

```go
// `objectbox:"uid:1306759095002958910"`
type OldEntityName struct {
	Id  uint64
}

type Task struct {
	Id  uint64
	OldPropertyName string `objectbox:"uid:9141374017424160113"`
}
```

&#x20;**Step 4:** The last thing to do is the actual rename on the language level:

```go
// `objectbox:"uid:1306759095002958910"`
type RenamedEntity struct {
	Id  uint64
}

type Task struct {
	Id  uint64
	RenamedProperty string `objectbox:"uid:9141374017424160113"`
}
```

&#x20;You can now use your renamed entity/property as expected and all existing data will still be there.

&#x20;Note: Instead of the above you can also find the UID of the entity/property in the `objectbox-model.json` file yourself and add it together with the @Uid annotation before renaming your entity/property.

## Changing Property Types

{% hint style="warning" %}
ObjectBox does not support migrating existing property data to a new type. You will have to take care of this yourself, e.g. by keeping the old property and adding some migration logic.
{% endhint %}

### **N**ew property, different name

{% hint style="info" %}
This solution useful if you need data migration or just want to keep the old data around.
{% endhint %}

```go
type Task struct {
	Id  uint64
	OldProperty string 
}

// becomes

type Task struct {
	Id  uint64
	OldProperty string 
	NewProperty int
}

// Note, if the property already had an UID annotation, 
//  don't add the same UID to the new property - skip the annotation instead.
```

### **Reset data - new UID on a property**

{% hint style="warning" %}
This solution useful if you don't care about the original data at all = it will be lost.
{% endhint %}

&#x20;**Step 1:** Add an empty \`objectbox:"uid"\` annotation to the **** property you want to reset:

```go
type Task struct {
	Id  uint64
	Property string `objectbox:"uid"`
}
```

&#x20;**Step 2:** Re-generate ObjectBox code for the project using `go generate ./...` in your project directory. The generation will fail with an error message that gives you the current UID of the property:

{% code title="" %}
```
can't merge binding model information: uid annotation value must not be empty on property Property, entity Task:
    [rename] apply the current UID 9141374017424160113
    [change/reset] apply a new UID 6050128673802995827
```
{% endcode %}

&#x20;**Step 3:** Apply the UID printed in the error message to your property (and change its type):

```go
type Task struct {
	Id  uint64
	Property int `objectbox:"uid:6050128673802995827"`
}
```

You can now use the property in your entity as if it was a new one.&#x20;

{% hint style="warning" %}
The original property data isn't really removed right away on old stored objects but will be empty when read from DB and overwritten (thus finally lost) next time an object is written.
{% endhint %}
