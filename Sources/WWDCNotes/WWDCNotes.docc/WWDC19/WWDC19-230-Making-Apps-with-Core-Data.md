# Making Apps with Core Data

Core Data helps manage the flow of data throughout your app. Hear about new features in Core Data that make your code simpler and more powerful, including derived attributes, history tracking, change notifications and batch operations. Learn more about using these facilities and the new diffing APIs in UIKit and Foundation to make your apps run more efficiently.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/230", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(matteom)
   }
}



## Getting started

> [Sample app: Creating blog posts with tags][app]

![][demo]

## Modeling data

- The app has types for posts, media attachments, tags.
- Attachments might be large, so we store data separately.

![][shape]

Once you define the shape of your data, it is straightforward to translate it into a Core Data model.

![][model]

Here we define data entities and relationships.

- The relationship between an Attachment and ImageData is one-to-one. Core Data can delete the image data when an attachment is deleted (cascade deletion rule).
- The relationship between an `Post` and `Attachment` is one-to-many.
- The relationship between an `Post` and `Tag` is many-to-many.

## The Core Data stack

![][stack]

- Model: [`NSManagedObjectModel`][NSManagedObjectModel]
- Persistent store coordinator: [`NSPersistentStoreCoordinator`][NSPersistentStoreCoordinator]
  - responsible for managing persistence stores
  - (most of the time) this is a database that lives on the file system
  - It's possible to have many stores at once, including our own custom made types that derive from [`NSPersistentStore`][NSPersistentStore]
  - The coordinator has to know about the model to make sense of stores

- Managed object context: [`NSManagedObjectContext`][NSManagedObjectContext]
  - anything that has to do with the store has to go through the context. E.g., create and store objects, perform fetch requests
  - contexts needs to know a coordinator in order to do their work
  - contexts provide us with seamless access to managed data

- Persistent container: [`NSPersistentContainer`][NSPersistentContainer]
  - While the types above are all sufficiently interdependent, the coordinator provides one type that encapsulates them all and represents the entire stack: the persistent container
  - By using a persistent container, we can set up a stack in just a few lines of code, especially if the model lives in our bundle

[NSManagedObjectModel]: https://developer.apple.com/documentation/coredata/nsmanagedobjectmodel
[NSPersistentStoreCoordinator]: https://developer.apple.com/documentation/coredata/nspersistentstorecoordinator
[NSPersistentStore]: https://developer.apple.com/documentation/coredata/nspersistentstore
[NSManagedObjectContext]: https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext
[NSPersistentContainer]: https://developer.apple.com/documentation/coredata/nspersistentcontainer

### Setting up the stack

By name:

```swift
let container = NSPersistentCloudKitContainer(name: "CoreDataCloudKitDemo")
container.loadPersistentStores { _, error in /* ... */ }
```

With a model generated in code:

```swift
let container = NSPersistentCloudKitContainer(name: "CoreDataCloudKitDemo", managedObjectModel: model)
container.loadPersistentStores { _, error in /* ... */ }
```

### Configuring Managed Object Contexts

Query generations provide a stable view of the stores data, allowing safe and consistent access to objects, even if they're changed or deleted by another actor.

```swift
try container.viewContext.setQueryGenerationFrom(.current)
```

Contexts can be configured to keep themselves up-to-date as changes are saved by their siblings.

```swift
context.automaticallyMergesChangesFromParent = true
```

The most important thing to remember when using a context is that all store requests and interactions with managed objects must be done in the context queue.

There's a blocking invariant, as well as an asynchronous version of the perform API. The container offers a convenience method for performing background tasks that creates a background context for you.

```swift
context.performAndWait {
    /* ... */
}
context.perform {
    /* ... */
}
container.performBackgroundTask { context in
    /* ... */
}
```
## Adding data

Use the initializer provided by the managed object subclasses that were generated from the Core Data model by Xcode.

```swift
context.perform {
    let post = Post(context: context)
    post.title = "Hello, world!"
    try? context.save()
}
```

To insert objects by the hundreds or thousands, use batch insertions.

```swift
let rawPostsData: Data = // Server response ...
if let postDicts = try? JSONSerialization.jsonObject(with:rawPostsData) as? [[String : Any]] {
    context.perform {
        let insertRequest = NSBatchInsertRequest(entity: Post.entity(), objects: postDicts)
        let insertResult = try? context.execute(insertRequest) as! NSBatchInsertRequest
        let success = insertResult.result as! Bool
    }
}
```

The keys in the JSON objects need to line up with the names of your attributes in the model.

![][batch]

- If you have unique constraints then any existing objects matching the dictionary will be pulled out of the database and updated with new values.
- Attributes that are optional or configured with default values can be omitted from the dictionary.
- In the case of updating an object with unique constraint, the existing values will not be changed.
- Batch insertions can't be used to set relationships. But if a batch insert updates an existing object due to a unique constraint, the existing relationships will be left alone.

## Fetching data

### Fetching an object

We use a fetch request to get objects out of the store and use it to configure a view.

```swift
let fetchRequest : NSFetchRequest<Tag> = Tag.fetchRequest()

fetchRequest.predicate = NSPredicate(format: "name = %@", tagName)

if let tag = try? fetchRequest.execute().first {
    tagLabel.text = tag.name
    tagLabel.textColor = tag.color as? UIColor
}
```

- If the tag's name or color change, our managed object context will make sure that the objects' properties get updated.
- To observe those changes use KVO.
- The Combine framework makes KVO easier to use in Swift. Check out [Combine in Practice][combine]

```swift
let fetchRequest : NSFetchRequest<Tag> = Tag.fetchRequest()

fetchRequest.predicate = NSPredicate(format: "name = %@", tagName)

if let tag = try? fetchRequest.execute().first {
    nameSubscription = tag.publisher(for: \.name)
        .assign(to: \.text, on: tagLabel)
    
    colorSubscription = tag.publisher(for: \.color)
        .map({ $0 as? UIColor })
        .assign(to: \.textColor, on: tagLabel)
}
```

### Fetching many objects

Sort results with sort descriptors.

```swift
fetchRequest.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
```

Batch fetching to avoid filling up memory and keep the app responsive.

```swift
fetchRequest.fetchBatchSize = 50
```

To monitor changes, use an `NSFetchedResultsController`.

```swift
let fetchRequest: NSFetchRequest<Post> = Post.fetchRequest()
fetchRequest.sortDescriptors = [NSSortDescriptor(key: "title", ascending: true)]
fetchRequest.fetchBatchSize = 50

let controller = NSFetchedResultsController(fetchRequest: fetchRequest,
    managedObjectContext: moc,
    sectionNameKeyPath: nil, cacheName: nil)

controller.delegate = self try! controller.performFetch()
try! controller.performFetch()
```

A fetched results controller communicates changes to a fetch requests through a delegate protocol.

```swift
controllerWillChangeContent(:)
controller(:didChange:atSectionIndex:for:)
controller(:didChange:at:for:newIndexPath:)
controllerDidChangeContent(:)
```

There is also a delegate method that vends an instance of `NSDiffableDataSourceSnapshot`, which you can use with a `DiffableDataSource` for a table view. Check [Advances in UI Data Sources][data-sources].

```swift
func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>,
    didChangeContentWith snapshot: NSDiffableDataSourceSnapshotReference<NSManagedObjectID, NSString> ) {
        collectionViewDataSource.applySnapshot(snapshot as! NSDiffableDataSourceSnapshot)
}

```

There is also a delegate method that gives you a summary of all the changes to the fetched results in one shot.

- It returns a `CollectionDifference`, which encodes the difference between two collections.
- Introduced in [Swift Evolution Proposal 240][proposal-240].
- Check [Advances in Foundation][foundation].

```swift
func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>,
    didChangeContentWith diff: CollectionDifference<NSManagedObjectID>) {
    collectionView.performBatchUpdates({
        for change in diff {
            switch change {
            case .insert(offset: let newRow, element: _, associatedWith: let assoc):
                if let oldRow = assoc {
                    collectionView.moveItem(
                        at: IndexPath(row: oldRow, section: frcSection),
                        to: IndexPath(row: newRow, section: frcSection))
                } else {
                    collectionView.insertItems(
                        at: [IndexPath(row: newRow, section: frcSection)])
                }
            case .remove(offset: let oldRow, element: _, associatedWith: let assoc):
                if assoc == nil {
                    collectionView.deleteItems(
                        at: [IndexPath(row: oldRow, section: frcSection)])
                }
	    	}
        }
	}, completion: nil)
}
```

- We start by kicking off a batch update, and looping over the changes in the diff.
- `CollectionDifferences` supports two kinds of changes, Insert and Remove which may refer to each other through association. 
- In the first case, we have an insertion that has an associated removal.
- In the second case, we have an insert of an object that was not previously part of the fetched results. So we tell the `CollectionView` to add it.
- Finally, we match all removes that weren't part of an associated move by filtering for new associations, and then remove those from the `CollectionView`.

## Denormalization

In some cases data can difficult to fetch.

- You might be unable to build a fetch request.
- You might end up with performance problems.

At a certain point, you need to give up some modeling purity in order to accomplish your goals. You do that with denormalization.

- keep copies of your data, so access can be more efficient.
- This comes with the price of some additional overhead maintaining that extra data. But there are many of circumstances where this tradeoff is a no-brainer.
- Databases indexes are a good example.

Check [Core Data Best Practices][best-practices].

![][denormalization]

- Derived attributes are normalized metadata that's maintained for you by Core Data.
- Derived attributes are defined in the managed object model. The Model Editor in Xcode has an interface for it.
- You can define derived attributes in code using `NSDerivedAttributeDescription`.
- Derivation expressions can refer to any of the properties of an entity, one level deep.

Supported derivations available:

- Outright duplication, e.g., keeping a copy of an attachments identifier and the image data that backs it.
- Simple transformation of fields, e.g., such as lower casing a tag's name, or canonicalizing some Unicode string.
- Aggregate function across a too many relationship. Example above.
- Global functions that take no parameters. Useful for things like keeping track of when an object was last updated.

## Scaling your app

- `PersistentHistory` is a tool to process data added by an importer, or maintain data consistency across multiple active coordinators that are using the same store
- Fetch requests to make it easier to look up only the history you're interested in.

```swift
class func entityDescription(withContext context: NSManagedObjectContext) -> NSEntityDescription?
class var entityDescription: NSEntityDescription? { get }
class var fetchRequest: NSFetchRequest? { get }
```

- `NSPersistentHistoryTransaction` and `NSPersistentHistoryChange` have cross-methods to work with fetch requests.
- These include accessories for an entity description corresponding to the type and a method that produces a new preconfigured fetch requests that will return instances of the type when executed.
- You configure the fetch request to the predicate which gets used as part of a `PersistentHistoryRequest`.

```swift
open class NSPersistentHistoryChangeRequest : NSPersistentStoreRequest {
    open class func fetchHistory(withFetch fetchRequest: NSFetchRequest<NSFetchRequestResult>) -> Self
    open var fetchRequest: NSFetchRequest<NSFetchRequestResult>?
}
```

There's a new convenience initializer on `NSPersistentHistoryChangeRequest` for creating a new instance with a fetch request, as well as immutable property you can use for post-talk configuration.

### Remote change notifications

Cross-coordinator change notifications. To turn them on, there's a new `PersistentStore` option called `NSPersistentStoreRemoteChangeNotificationPostOptionKey`.

```swift
let description: NSPersistentStoreDescription = /* ... */
description.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
```

- Set it in your store description before loading the `PersistentStore`, and the coordinator will listen for remote changes.
- It also tells the coordinator to send remote change notifications whenever it makes any changes.

```swift
func storeRemoteChange(_ notification: Notification) {
    precondition(notification.name == NSNotification.Name.NSPersistentStoreRemoteChange)
    let storeURL = notification.userInfo?[NSPersistentStoreURLKey]!
    let token = notification.userInfo?[NSPersistentHistoryTokenKey]!
    print("Store at \(storeURL) was changed in transaction \(token).")
}
```

- Remote change notifications tell you which store changed with the `NSPersistentStoreURL` key in the user info dictionary.
- If you have `PersistentHistory` enabled, it also includes the new history token created by that transaction.

Because remote change notifications work kind of like push notifications, sometimes changes make it collapse together if there are many at once. And only the last would get delivered.

To get the current `PersistentHistory` token there is a method on the persistent store coordinator.

```swift
extension NSPersistentStoreCoordinator {
    func currentPersistentHistoryToken(from stores: [NSPersistentStore]?) -> NSPersistentHistoryToken?
}
```

## Testing

- Know what your performance goals are. A Contacts app should be testing with at least tens of thousands of objects. The Photos app should be testing millions of objects.
- Integration tests should also get running configurations optimized for detecting and surfacing other kinds. For applications using Core Data, that should include the Framework's built-in concurrency debugging.
- Use in-memory stores when test runtime is important, specifically the SQLiteStores in-memory mode.
- Use the address sanitizer, thread sanitizer, and undefined behavior sanitizer.

```swift
let container = NSPersistentCloudKitContainer(name: "CoreDataCloudKitDemo")
let description = container.persistentStoreDescriptions.first!
description.url = URL(fileURLWithPath: "/dev/null")
container.loadPersistentStores(completionHandler: { (_, error) in
    guard let error = error as NSError? else { return }
    fatalError("###\(#function): Failed to load persistent stores:\(error)")
})
```

[app]: https://developer.apple.com/documentation/coredata/synchronizing_a_local_store_to_the_cloud
[combine]: ../721
[data-sources]: ../220
[proposal-240]: https://github.com/apple/swift-evolution/blob/master/proposals/0240-ordered-collection-diffing.md
[foundation]: ../723
[best-practices]: https://developer.apple.com/videos/play/wwdc2018/224
[demo]: WWDC19-230-demo
[shape]: WWDC19-230-shape
[model]: WWDC19-230-model
[stack]: WWDC19-230-stack
[batch]: WWDC19-230-batch
[denormalization]: WWDC19-230-denormalization
