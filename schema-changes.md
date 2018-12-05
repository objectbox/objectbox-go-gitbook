# Schema changes

ObjectBox manages its data model \(schema\) mostly automatically. The data model is defined by the entity structs you define. When you **add or remove** entities or properties of your entities, **ObjectBox takes care** of those changes without any further action from you.

For other changes like **renaming or changing the type**, ObjectBox needs **extra information** to make things unambiguous. This works using unique identifiers \(UIDs\) specified by the [\`uid\` annotation](entity-annotations.md#uid), as we will see below.

## Renaming Entities and Properties <a id="renaming-entities-and-properties"></a>

So why do we need that UID annotation? If you simply rename an entity struct, ObjectBox only sees that the old entity is gone and a new entity is available. This can be interpreted in two ways:

* The old entity is removed and a new entity should be added, the old data is discarded. This is the **default behavior** of ObjectBox.
* The entity was renamed, the old data should be re-used.

So to tell ObjectBox to do a rename instead of discarding your old entity and data, you need to make sure it knows that this is the same entity and not a new one. You do that by attaching the internal UID to the entity.

### Step-by-step example

The process works the same if you want to rename a property, but instead of the comment, you just use standard Go tags. We are showing both cases in the examples

 **Step 1:** Add an empty \`uid\` annotation to the entity/property you want to rename:

```go
// `uid`
type OldEntityName struct {
	Id  uint64
}

type Task struct {
	Id  uint64
	OldPropertyName string `uid`
}
```

 **Step 2:** Re-generate ObjectBox code for the project using `go generate ./...` in your project directory. The generation will fail with an error message that gives you the current UID of the entity/property:

```text
can't merge binding model information: uid annotation value must not be empty 
(model entity UID = 1306759095002958910) on entity OldEntityName
```

```text
can't merge binding model information: uid annotation value must not be empty 
(model property UID = 9141374017424160113) on property OldPropertyName, entity Task
```

 **Step 3:** Apply the UID printed in the error message to your entity/property:

```go
// `uid:"1306759095002958910"`
type OldEntityName struct {
	Id  uint64
}

type Task struct {
	Id  uint64
	OldPropertyName string `uid:"9141374017424160113"`
}
```

 **Step 4:** The last thing to do is the actual rename on the language level \(Java, Kotlin, etc.\):

```go
// `uid:"1306759095002958910"`
type RenamedEntity struct {
	Id  uint64
}

type Task struct {
	Id  uint64
	RenamedProperty string `uid:"9141374017424160113"`
}
```

 You can now use your renamed entity/property as expected and all existing data will still be there.

 Note: Instead of the above you can also find the UID of the entity/property in the `objectbox-model.json` file yourself and add it together with the @Uid annotation before renaming your entity/property.

