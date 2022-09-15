---
description: >-
  ObjectBox db supports relations. Learn all about To-One, To-Many, One-to-Many,
  Many-to-Many relations and lazy loading for ObjectBox Go. Relations are
  initialized eagerly by default.
---

# Relations

Objects may reference other objects, for example using a simple reference or a list of objects. In database terms, we call those references **relations**. The object defining the relation we call the **source** object, the referenced object we call **target** object. So the relation has a direction.

If there is one target object, we call the relation **to-one.** And if there can be multiple target objects, we call it **to-many**.&#x20;

{% hint style="info" %}
Relations are initialized eagerly by default - i.e. the targets are loaded & as soon as the source object is read from the database. Lazy/manual loading of to-many relations is possible, using an annotation.&#x20;
{% endhint %}

## To-One Relations

<figure><img src=".gitbook/assets/to-one-relations-2.png" alt="To-One-relations"><figcaption><p>To-One Relations</p></figcaption></figure>

You define a to-one relation using \`link\` annotation on a field that is a pointer or value type of another entity. Consider the following example - the Order entity has a to-one relation to the Customer entity.

{% code title="model.go" %}
```go
type Order struct {
	Id        uint64
	Customer  *Customer `objectbox:"link"`
	Notes     string
}

type Customer struct {
    Id    uint64
    Name  string
}
```
{% endcode %}

Now let's add a customer with a few orders.

{% code title="main.go" %}
```go
// note that here we're creating a new customer
// but we could have also reused an existing one
var customer = &model.Customer{Name: "ACME Inc."}

var box = model.BoxForOrder(ob)

// Insert a new order. ObjectBox also inserts the customer automatically 
// because it's new (customer.Id == 0 at this point)
box.Put(&model.Order{
    Notes: "first order, new customer",
    Customer: customer,
})

...

// Add another order. Now the customer.Id is already > 0 
// so it's not inserted again, just referenced
box.Put(&model.Order{
    Text: "second order, existing customer",
    Customer: customer,
})
```
{% endcode %}

After the `box.Put` has been executed on the first order, the `customer.Id` would be `1` because we're using pointers (`Customer *Customer` field) so Put could update the variable when it has inserted the Customer. Note that this wouldn't be possible if we were using copies (`Customer Customer` field) and in that case you should insert the customer manually into it's box first (or use an existing customer selected from the database).

We can also **read**, **update** or **remove** the relationship to a customer:

{% code title="main.go" %}
```go
var box = model.BoxForOrder(ob)

order, _ := box.Get(1) // Read
// at this point, order.Customer is already loaded automatically (eager-loading)

order.Customer = nil   // Remove the relation
box.Put(order)         // Update

// or do an update to a different customer
customers, _ := model.BoxForCustomer(ob).GetAll()
order.Customer = customers[2]
box.Put(order)
```
{% endcode %}

Note that removing the relation does not remove the customer from the database, it removes only the link between this specific order and the customer.

## To-Many Relations

There is a slight difference if you require a one-to-many (1:N) or many-to-many (N:M) relation. \
A 1:N relation is like the example above where a customer can have multiple orders, but an order is only associated with a single customer. An example for an N:M relation are students and teachers: students can have classes by several teachers but a teacher can also instruct several students.

### One-to-Many (1:N)

<figure><img src=".gitbook/assets/one-to-many.png" alt="one to many"><figcaption><p>One-to-Many (1:N)</p></figcaption></figure>

Currently, one-to-many relations are defined implicitly as an opposite relation to a [to-one relation ](relations.md#to-one-relations)as defined above. This is useful for queries, e.g. to select all customers with an order placed within the last seven days.

### Many-to-Many (N:M)

<figure><img src=".gitbook/assets/many-to-many2.png" alt="many to many"><figcaption><p>Many-to-Many (N:M)</p></figcaption></figure>

To define a to-many relation, you can use a slice of entities - no need to specify the `link` annotation this time because ObjectBox wouldn't know how to store a slice of structs by itself anyway so it assumes it must be a many-to-may relation. They're stored when you put the source entity and loaded when you read it from the database, unless you specify a `lazy` annotation in which case, they're loaded manually, using `Box::GetRelated()`.

Assuming a students and teachers example, this is how a simple student class that has a to-many relation to teachers can look like:

{% code title="model.go" %}
```go
type Teacher struct {
	Id    uint64
	Name  string
}

type Student struct {
    Id        uint64
    Name      string
    Teachers  []*Teacher
}
```
{% endcode %}

**Adding** the teachers of a student works exactly like with a list:

{% code title="main.go" %}
```go
var teacher1 = &model.Teacher{Name: "John Wise"}
var teacher2 = &model.Teacher{Name: "Peter Clever"}

var student1 = &model.Student{
    Name: "Martin Curious",
    // we can create the slice in place
    Teachers: []*model.Teacher{teacher1, teacher2},
}

// or append to it
var student2 = &model.Student{Name: "Earl Eager"}
student2.Teachers = append(student2.Teachers, teacher2)

// puts students and teachers
var box = model.BoxForStudent(ob)
box.Put(student1)
box.Put(student2)
```
{% endcode %}

Similar to the to-one relations, related entities are inserted automatically if they are new. If the teacher entities do not yet exist in the database, the to-many will also put them. If they already exist, the to-many will only create the relation (but not put them).&#x20;

To **get** the teachers of a student we just access the list:

{% code title="main.go" %}
```go
var student1 = model.BoxForStudent(ob).Get(1);

for _, teacher := range student1.Teachers {
    fmt.PrintLn(teacher.Name)
}
```
{% endcode %}

**Remove** and **update** work similar to insert - you just change the `student.Teachers` slice to reflect the new state (i.e. remove element, add elements, etc) and `box.Put(student)`. Note that if you want to change actual teacher data (e.g. change teachers name), you need to update the teacher entity itself, not just change it in one of the student.Teachers slice.

### Lazy loading

In case the slices might contain many objects and you don't need to access the slice of the related objects each time you work with the source object, you may consider enabling the so called lazy-loading. You do that by specifying the \`lazy\` annotation on the field. Consider the updated model of the previous example:&#x20;

{% code title="model.go" %}
```go
type Teacher struct {
	Id    uint64
	Name  string
}

type Student struct {
    Id        uint64
    Name      string
    Teachers  []*Teacher `objectbox:"lazy"`
}
```
{% endcode %}

This way, when you read a `Student` object, the `Teachers` field would be `nil` and you can work with the student as you wish, changing it and saving and the list of assigned teachers wouldn't change as long as the `Teachers` field stays `nil`. If it wasn't `nil`, but a slice of Teachers instead, ObjectBox would recognize this as an update of the field and replace the relational links.

#### Reading a lazy-loaded slice

To access the list of `Teachers`, we need to first load them. ObjectBox has generated a helper method just for that

{% code title="main.go" %}
```go
var box = model.BoxForStudent(ob)

var student1 = box.Get(1);

// at this point `student1.Teachers == nil`, so if we need it, we must load it first
box.GetRelated(student1) // loads all lazy-loaded relations

// or alternatively load just the Teachers property 
// (useful if there were other lazy-loaded relations we didn't care about this time)
box.GetRelated(student1, Student_.Teachers)

// now the teachers are loaded and we can access them as usual
for _, teacher := range student1.Teachers {
    fmt.PrintLn(teacher.Name)
}
```
{% endcode %}

#### Updating a lazy-loaded slice

To update the list of `Teachers`, we can either overwrite the slice with completely new data (new slice), or if we want to keep the original data and update it, e.g. change a few items, we need to load them first the same way as when [reading (above)](relations.md#reading-a-lazy-loaded-slice).

{% code title="main.go" %}
```go
var box = model.BoxForStudent(ob)

var student1 = box.Get(1);

// propagate student1.Teachers based on the current data in DB
box.GetRelated(student1, Student_.Teachers)

// add a new teacher to the existing
student1.Teachers = append(student1.Teachers, &model.Teacher{Name: "Peter Clever"})

// save the updated list, including a new teacher
box.Put(student1)
```
{% endcode %}
