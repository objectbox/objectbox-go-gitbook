---
description: >-
  While you can use ObjectBox without doing transactions manually, sometimes
  they come in handy.
---

# Transactions

## Basics

ObjectBox is a fully transactional database satisfying [ACID](https://en.wikipedia.org/wiki/ACID) properties. A transaction can group several operations into a single unit of work that either executes completely or not at all. If you are looking for a more detailed introduction to transactions in general, please consult other resources like Wikipedia on [database transactions](https://en.wikipedia.org/wiki/Database_transaction). 

You may not notice it, but almost all interactions with ObjectBox involve transactions. For example, if you call `box.Put()` a write transaction is used. Also if you `box.Get()` an object or query for objects, a read transaction is used. All of this is done under the hood and transparent to you. It may be fine to completely ignore transactions altogether in your app without running into any problems. With more complex apps however, it’s usually worth learning transaction basics to make your app more consistent and efficient.

## Explicit transactions <a id="explicit-transactions"></a>

{% hint style="danger" %}
This API is currently not published for general use and is subject to change.
{% endhint %}

We learned that all ObjectBox operations run in implicit transactions – unless an explicit transaction is in progress. In the latter case, multiple operations share the \(explicit\) transaction. In other words, with explicit transactions you control the transaction boundary. Doing so can greatly improve efficiency and consistency in your app.

The `ObjectBox::RunInTxn()` takes a function as and argument and runs it inside a transaction.

The advantage of explicit transactions is that you can perform any number of operations and use objects of multiple boxes. In addition, you get a consistent \(transactional\) view on your data while the transaction is in progress.

Example for a write transaction which just inserts 1 000 000 objects:

```go
var insert = uint64(1000000)
ob.RunInTxn(false, func(tx *objectbox.Transaction) (error) {
	cursor, _ := tx.CursorForName("Task")
	for i := insert; i > 0; i-- {
		cursor.Put(&model.Task{})
	}
	return nil
})
```

Understanding transactions is essential to master database performance. If you just remember one sentence on this topic, it should be this one: a write transaction has its price.

Committing a transaction involves syncing data to the physical storage, which is a relatively expensive operation for databases. Only when the file system confirms that all data has been stored in a durable manner \(not just memory cached\), the transaction can be considered successful. This file sync required by a transaction may take a couple of milliseconds. Keep this in mind and try to group several operations \(e.g.`Put`calls\) in one transaction.

## Read Transactions <a id="read-transactions"></a>

In ObjectBox, read transactions are cheap. In contrast to write transactions, there is no commit and thus no expensive sync to the file system. Operations like `Get` , `Count` , and queries run inside an implicit read transaction if they are not called when already inside an explicit transaction \(read or write\). Note that it is illegal to `Put` when inside a read transaction.

While read transaction are much cheaper than write transactions, there is still some overhead to start a read transaction. Thus, for a high number of reads \(e.g. hundreds, in a loop\), you can improve performance by grouping those reads in a single read transaction \(see explicit transactions below\).

## Multiversion concurrency <a id="multiversion-concurrency"></a>

ObjectBox gives developers [Multiversion concurrency control \(MVCC\)](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) semantics. This allows multiple concurrent readers \(read transactions\) which can execute immediately without blocking or waiting. This is guaranteed by storing multiple versions of \(committed\) data. Even if a write transaction is in progress, a read transaction can read the last consistent state immediately. Write transactions are executed sequentially to ensure a consistent state. Thus, it is advised to keep write transactions short to avoid blocking other pending write transactions. For example, it is usually a bad idea to do networking or complex calculations while inside a write transaction. Instead, do any expensive operation and prepare objects before entering a write transaction.

Note that you do not have to worry about making write transactions sequential yourself. If multiple threads want to write at the same time \(e.g. via `put` or `RunInTxn`\), one of the treads will be selected to go first, while the other threads have to wait. It works just like a `mutex.Lock()`

### Locking inside a Write Transaction <a id="locking-inside-a-write-transaction"></a>

{% hint style="danger" %}
Avoid locking \(e.g. via `mutex.Lock()`\) when inside a write transaction when possible.
{% endhint %}

Because write transactions run exclusively, they effectively acquire a write lock internally. As with all locks, you need to pay close attention when multiple locks are involved. Always obtain locks in the same order to avoid deadlocks. If you acquire a lock “X” inside a transaction, you must ensure that your code does not start another write transaction while having the lock “X”.

