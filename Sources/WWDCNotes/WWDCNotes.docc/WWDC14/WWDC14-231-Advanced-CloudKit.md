# Advanced CloudKit

Dive deep into CloudKit! Learn how to perform advanced operations on records and store private data and gain a deeper understanding about custom record zones, ensuring data integrity, and effectively modeling your data.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/231", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



CloudKit API is designed to be asynchronous, all calls return through a callback, because they all require a network connection.

The main API (“operational API”) is based on [`NSOperation`](https://developer.apple.com/documentation/foundation/nsoperation). You use it by creating special [`NSOperation`](https://developer.apple.com/documentation/foundation/nsoperation) objects for a given use case, e.g. [`CKFetchRecordsOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordsoperation), and specifying parameters and callbacks in their properties. Apart from the final result callback, you can set callbacks e.g. for reporting download progress or to get records one by one as they’re downloaded.

Operation lifecycle (cancelling, suspending etc.) can be managed through standard [`NSOperation`](https://developer.apple.com/documentation/foundation/nsoperation) methods and [`NSOperationQueue`](https://developer.apple.com/documentation/foundation/nsoperationqueue).

There are separete fetch/modify operation types for records, subscriptions, zones, users and notifications. You can set dependencies between operations (even if they’re in different queues), e.g. make a fetch operation and then a modify operation that needs to wait for the object to load.

Operations can also have different priority levels.

### Starting an operation:

(Note: this wasn’t actually in the video, but I think it should have been, because it's not obvious at all.)

How to start an operation once you prepare the [`CKOperation`](https://developer.apple.com/documentation/cloudkit/ckoperation) object:

1) Use the database’s built-in queue:

```swift
let fetchOperation = ...
CKContainer.default().privateCloudDatabase.add(fetchOperation)
```

2) Use your own operation queue and assign a reference to the database:

```swift
let operationQueue = NSOperationQueue()

let fetchOperation = ...
fetchOperation.database = CKContainer.default().privateCloudDatabase
operationQueue.addOperation(fetchOperation)
```

## Custom zones

Custom zones (in the private database) let you compartmentalize data and add some special features. Records can’t be moved between zones or have cross-zone relationships.

There are some operations that can only be done in custom zones:

### Atomic commits:

- Objects in the CloudKit database have relationships between them, and you want to keep all data consistent
- Atomic commits are kind of like transactions in a relational database: batch operations succeed or fail together
- Only available in the private database (because public database may be accessed by millions of users at the same time)
- If an operation fails, you get a [`CKErrorPartialFailure`](https://developer.apple.com/documentation/cloudkit/ckerror/code/partialfailure) response, with the user info containing info about errors on specific records ([`CKPartialErrorsByItemID`](https://developer.apple.com/documentation/cloudkit/ckpartialerrorsbyitemidkey))
- Error [`CKErrorBatchRequestFailed`](https://developer.apple.com/documentation/cloudkit/ckerror/code/batchrequestfailed) means that this record wasn’t saved because of a problem with another record in the batch

### Delta downloads:

- They allow you to download a list of all changes since the last time the app was online, to let you perform a full sync
- When a device connects, you can send a “change token” to the server asking for all changes since that version
- This lets you implement an offline cache of the whole dataset and sync any changes when possible

To do that:

- track all local changes
- send changes to the server when connected
- resolve conflicts
- fetch server changes with [`CKFetchRecordChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordchangesoperation)
- remember the received new server change token and send it back next time

### Zone subscriptions:

- Let you subscribe for notifications about any change in the zone
- When you get a notification, you request a delta download


## Advanced record operations

### Record changes:

When you change some fields in a CKRecord, the changes are automatically tracked locally and only the changed fields are transmitted when you save it. By default CloudKit performs a “locked update”, which makes sure that the update is only saved on the server if the record wasn’t modified in the meantime by another client (this uses record change tokens). After you execute a save, the server returns your record with a new change token - so you should use that returned version for any subsequent changes.

You have a choice between:

- Unlocked update ⭢ just overwrites server data regardless what is there
- Locked update ⭢ if the record was changed in the meantime, you get back an error ([`CKErrorServerRecordChanged`](https://developer.apple.com/documentation/cloudkit/ckerror/code/serverrecordchanged))

The `userInfo` of the [`CKErrorServerRecordChanged`](https://developer.apple.com/documentation/cloudkit/ckerror/code/serverrecordchanged) error contains info that lets you perform a 3-way merge:

- [`CKRecordChangedErrorClientRecordKey`](https://developer.apple.com/documentation/cloudkit/ckrecordchangederrorclientrecordkey) - what you tried to save
- [`CKRecordChangedErrorAncestorRecordKey`](https://developer.apple.com/documentation/cloudkit/ckrecordchangederrorserverrecordkey) - the original version
- [`CKRecordChangedErrorServerRecordKey`](https://developer.apple.com/documentation/cloudkit/ckrecordchangederrorancestorrecordkey) - what is currently on the server 

Based on the values from these 3 copies of the record you can decide what state the record should be in, and then retry the save.

You can modify the behavior with “save policies”:

- [`SaveIfServerUnchanged`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation/recordsavepolicy/ifserverrecordunchanged) ⭢ default, performs a locked update and sends only changed keys
- [`SaveChangedKeys`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation/recordsavepolicy/changedkeys) ⭢ unlocked update, sends only changed keys
- [`SaveAllKeys`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation/recordsavepolicy/allkeys) ⭢ unlocked update, overwrites all keys in the record (note: this doesn’t affect keys that aren’t present in the local copy at all)

You should almost always use the default locked update ([`SaveIfServerUnchanged`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation/recordsavepolicy/ifserverrecordunchanged)), use unlocked updates only to forcefully resolve serious conflicts. Use [`SaveAllKeys`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation/recordsavepolicy/allkeys) if the user requests to overwrite server data with local data.

### Partial records:

The [`desiredKeys`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordsoperation/3003361-desiredkeys) field present in most operation types lets you specify that you only want to download selected keys from the server. This is useful if the whole record is very large and you don’t need all of it. Partial records can be normally saved after a change.


## CloudKit data modeling

### References:

Forward reference ⭢ a parent object keeps an array of references to children in its property

Backward reference ⭢ only child objects have a reference to the parent

It’s recommended to use backward references - with a forward reference you need to update the parent object every time a new child is added, and you will run into conflicts if multiple clients are adding records. To get a list of all children using backward references, make a query for all child records with a predicate like “owner = X”.

References give you cascading deletes - when you delete the parent object, all child objects and their children are deleted. If an object has two parent references, it’s deleted when the first parent is deleted.

When batch uploading a tree of objects, CloudKit makes sure that parent objects are uploaded first so that you don’t get inconsistent data during upload (important in the public database).

### Your data objects:

CloudKit is only a transport mechanism and requires you to keep and manage your own local copy of all data.

It’s recommended that you don’t subclass CK* objects to build your models - make your own completely independent model classes and translate to/from CloudKit objects when fetching and saving.

### Handling push notifications:

You need to remember that push notifications in general aren’t guaranteed to be delivered. The server only stores one push per client, so if you reconnect e.g. after a flight, you might miss some previous notifications.

You can find pushes that you’ve missed in a “Notification Collection” where every notification is saved. The Notification Collection works kind of like delta updates - you ask for notifications since a given change token and you get a list of everything added since then. You can mark a notification as read, which notifies all other clients that they can ignore it.

You should check the Notification Collection every time you get a push, since you never know what you might have missed (this doesn’t only happen with airplane mode).


## The iCloud Dashboard

The dashboard lets you browse data saved by your app - the whole public database and the private database for your developer account (but not anyone else’s private database). You can view saved records, run queries with any filters, and add new records.

You can define roles in the public database and define for each model who can create/read/modify records (e.g. specify that records are publicly readable but only an admin can create them). You will also see a list of all user ids and first/last names of those users that marked themselves as discoverable.

### Schema:

The CloudKit database has two separate “environments”: development and production.

The schema for each record type is “just in time” during development, i.e. when you save a new type of record, it automatically creates a new schema for that record type, recording every field type, and when you save a record with a new field, it adds a field to the list.

However, once you’re ready to release a new version of your app, you need to save the schema to production and at that point it’s locked - a production version of the app can’t save records or fields that aren’t defined in the schema.

CloudKit also automatically creates indexes for each field in each record type - when you’re done with development, you can delete some indexes that you won’t need so they don’t waste space in the production database.


## Tips & tricks

- Please handle all errors :)
- Remember that you can get partial errors (when atomic commits aren’t used), so some records might be saved while others aren’t
- Retry any “server busy” errors ([`CKErrorRetryAfterKey`](https://developer.apple.com/documentation/cloudkit/ckerrorretryafterkey) tells you the amount of time you should wait)
- Don’t waste space in your users’ iCloud in private databases, they may be paying real money for it
- Limits in the public database are mostly to prevent abuse, they should be fine for most normal use (the limits scale with the number of users)
