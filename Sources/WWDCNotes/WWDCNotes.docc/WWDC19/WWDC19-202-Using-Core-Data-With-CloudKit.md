# Using Core Data With CloudKit

CloudKit offers powerful, cloud-syncing technology while Core Data provides extensive data modeling and persistence APIs. Learn about combining these complementary technologies to easily build cloud-backed applications. See how new Core Data APIs make it easy to manage the flow of data through your application, as well as in and out of CloudKit. Join us to learn more about combining these frameworks to provide a great experience across all your customers’ devices.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/202", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(davidleee)
   }
}



Data we create on one device is naturally trapped. To solve this, we typically want to turn to cloud storage, because it offers us the promise of moving data from one device, seamlessly and transparently, to all the other devices that we owned.

Cloud storage has benefit even if we only have a single device such as data backup and restore.

## Using Core Data with CloudKit

- Everything should sync
- Sync should be easy

## Existing Technologies

- Core Data provides local persistence
- CloudKit provides distributed persistence
- Both exist on all platforms
- Both support a wide variety of applications

![existing-technologies](https://user-images.githubusercontent.com/8705231/158109333-3f1bd098-d754-4400-a150-230c9c438f96.png)

## What's New

If you ever built a CoreData application before, you would have seen [`NSPersistentContainer`](https://developer.apple.com/documentation/coredata/nspersistentcontainer) here. With the new subclass [`NSPersistentCloudKitContainer`](https://developer.apple.com/documentation/coredata/nspersistentcloudkitcontainer/) , you can add CloudKit functionality to existing CoreData applications by changing as little as one line of code.

```swift
// before
let container = NSPersistentContainer(name: "WWDCDemo")

// after
let container = NSPersistentCloudKitContainer(name: "WWDCDemo")
```

### What Is `NSPersistentCloudKitContainer`?

- Encapsulation of common patterns
- Save thousands lines of code
- Foundation we can build on
- Help us help you! Submit feedback!

#### NSPersistentCloudKitContainer

- A local replica of CloudKit data
- Robust scheduling and error recovery
- Transformation of NSManagedObject to CKRecord

## Life After Adopting NSPersistentCloudKitContainer

### Build Great Apps with Core Data

- Responsive user interfaces with NSFetchResultsController
- Stable views with query generations
- Change processing with history tracking

> more on [Making Apps with Core Data](https://www.wwdcnotes.com/notes/wwdc19/230/)

### Add on to our foundation

- Working with multiple stores
  - Data Segregation
  - Enforcement of different types of constraints
  - Throttling/Coalescing

- Working with the CloudKit Schema
  - Record types and entity names
  - Asset externalization
  - Relationships

- Data Modeling for collaboration
  - Collaboration is not conflict resolution
  - NSPersistentCloudKitContainer resolves conflicts automatically
  - Get better merge behavior with relationships

#### Multiple Stores

![multiple-stores](https://user-images.githubusercontent.com/8705231/158109444-15de4ec7-cb83-40ce-b836-cb4eaa7bd711.png)


By adding new configurations in <kbd>xcdatamodel</kbd> with a few lines of code, you can manage which entities you want to sync.

```swift
let container = NSPersistentCloudKitContainer(name: "CloudKitContainer"）

let local = NSPersistentStoreDescription(url: URL(fileURLWithPath:"/files/local.sqlite")）
local.configuration = "Local"

let cloud = NSPersistentStoreDescription(url: URL(fileURLWithPath: "/files/cloud.sqlite"））
cloud.configuration = "Cloud"
cloud.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(containerIdentifier:"iCloud.com.wwdc.demo")

container.persistentStoreDescriptions = [ local, cloud ]
```

What if we want to share some data in CloudKit across multiply applications we happened to work on? We can do this by just adding three lines of code.

```swift
// ...

let shared = NSPersistentStoreDescription(url: URL(fileURLWithPath: "/files/shared.sqlite"））
cloud.configuration = "Shared"
cloud.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(containerIdentifier:"iCloud.com.wwdc.shared")

container.persistentStoreDescriptions = [ local, cloud, shared ]
```

#### `NSPersistentCloudKitContainer`’s Schema

![schema1](https://user-images.githubusercontent.com/8705231/158109577-adfae8fb-5ef0-4237-9fe9-92ccb3cc30bd.png)

Core Data generates subclasses of `NSManagedObject` for us to use in code. Take `Post` for example, the class Core Data generates and the record along with it that CloudKit generates will look like this:

![schema2](https://user-images.githubusercontent.com/8705231/158109620-c7e554a9-ed39-4a45-9c7f-b945c81bbc91.png)

Core Data owns the `recordID` for all of the objects that it creates in CloudKit, and for each one CloudKit will generates a UUID to use as its record name. When the record name is combined with a zone identifier, you get a `CKRecordID`.

> Funny thing: Core Data will prefix everything with *CD_ to s*egregate the things it manages.

In order to implement entity inheritance, the actual entity name will be stored in `CD_entityName` .

You will not see `CD_xxx` and `CD_xxx_ckAsset` in the same time. If the strings are very short, it will be stored directly in `CD_xxx`.

![schema3](https://user-images.githubusercontent.com/8705231/158109635-2b2c23d0-eed9-469e-bb28-ca7afefb9f3b.png)

But if one of them grows to be very large, approximately larger than 750KB, or if the total size of the record exceeds CloudKit’s maximum 1MB limit, you will begin to see `CD_xxx_ckAsset` fields.

![schema4](https://user-images.githubusercontent.com/8705231/158109655-6067ae3f-3c21-47bf-9bbc-84249584ae42.png)

For a to-one relationship, in this case the `post` field in `Attachment` , the record will look like this:

![schema5](https://user-images.githubusercontent.com/8705231/158109666-b74c4783-39af-4649-9775-dd79e87ae1aa.png)

The UUID of the related record in CloudKit will always be stored on the object it’s linked to, since `CKReference` has some limitations that will not work well for Core Data Clients.

For many-to-many relationships, in this case the one between a `Post` and a `Tag` , a custom joint record will be created:

![schema6](https://user-images.githubusercontent.com/8705231/158109681-69843ac2-ab05-4941-99ae-4878a0021cbf.png)

#### Data Modeling for collaboration

- Avoid collisions on "flat" values
- Instead model values as contributions
- Leverage relationships for eventual consistency
- Order contributions
- Iterate as necessary

![data-modeling](https://user-images.githubusercontent.com/8705231/158109704-6a0a26a1-4400-4901-b6f5-8b08f964d629.png)
