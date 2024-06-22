# Introducing CloudKit

CloudKit is the framework that powers iCloud on iOS and OS X, now available directly in your app. Learn how you can take advantage of its feature-rich API to store and query your own custom data and assets in iCloud.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/208", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



CloudKit lets you write client applications without having to build and host a server part to handle things like database, accounts or push notifications.

Usage is free for the developer up to pretty big limits.

CloudKit gives you more direct access to iCloud servers. It’s the framework that’s used behind the scenes by iCloud Photo Library and iCloud Drive, uses the same iCloud account as iCloud documents or key-value storage.

There are two types of databases:

- public - accessible to everyone
- private - private data of a specific user

CloudKit is only a transport technology, it does not deal with local data persistence - you need to decide how you store the data that you load from the cloud.

To enable iCloud in your app, set it up in the Capabilities tab in Xcode just like with other iCloud APIs.

## Elements of the API

### Containers ([`CKContainer`](https://developer.apple.com/documentation/cloudkit/ckcontainer)):

- Each app’s data is kept in a separate container
- Containers give you the safety that your app’s data will not be mixed with someone else’s app’s data
- A container’s ID needs to be unique in the whole iCloud, so use reverse-domain style identifiers
- By default each app has one container of its own, but apps can additionally use shared containers
- Containers are managed by the developer through the WWDR portal

### Databases ([`CKDatabase`](https://developer.apple.com/documentation/cloudkit/ckdatabase)):

- A container contains one shared public database for everyone, and separate private databases for each user
- An app running on the device has access to one public and one private database
- The database is the initial entry point to CloudKit (from a container)

```objc
CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
CKDatabase *privateDatabase = [[CKContainer defaultContainer] privateCloudDatabase];
```

Private database:

- requires a logged in iCloud account
- data stored counts against the user’s iCloud account quota
- default permission for data is user readable
- the data your users store in your app’s CloudKit container is *not* accessible to you

Public database:

- can be accessed anonymously even if the user isn’t logged in
- data stored counts against the developer’s app quota
- default permission for data is world readable
- permissions can be customized using iCloud Dashboard Roles


### Records ([`CKRecord`](https://developer.apple.com/documentation/cloudkit/ckrecord)):

- A record is a single “object” in the CloudKit database, essentially a list of key-value pairs
- Records have a Record Type (≈ table name)
- There is no defined up front schema, you can just save a record of any type with any keys and the schema will be updated based on that
  - note: this only works in development, the schema is fixed in production, see "Advanced CloudKit"
- Records can have metadata: who created it & modified it and when, also includes a “change tag” (version id) - for determining if two sides have the same version of a record
- Record values can be: strings, numbers, dates, [`NSData`](https://developer.apple.com/documentation/foundation/nsdata), [`CLLocation`](https://developer.apple.com/documentation/corelocation/cllocation), [`CKReference`](https://developer.apple.com/documentation/cloudkit/ckrecord/reference), [`CKAsset`](https://developer.apple.com/documentation/cloudkit/ckasset), arrays of any of these

```objc
- (instancetype)initWithRecordType:(NSString *)recordType;
- (id)objectForKey:(NSString*)key;
- (void)setObject:(id)object forKey:(NSString *)key;
- (NSArray *)allKeys;
```

Subscripts also work:

```objc
CKRecord *party = [[CKRecord alloc] initWithRecordType:@"Party"];
party[@"start"] = [NSDate date];
```

### Record zones ([`CKRecordZoneID`](https://developer.apple.com/documentation/cloudkit/ckrecordzone/id)):

- Records are grouped within a database inside “zones”
- The public database has one zone, the private database has one default zone, but it can have additional custom zones


### Record identifiers ([`CKRecordID`](https://developer.apple.com/documentation/cloudkit/ckrecord/id)):

- Record identifier is a tuple grouping: a “record name” + zone ID
- You can provide a recordID when creating a record instance
- If you don’t provide a recordID, a random UUID will be assigned

### References ([`CKReference`](https://developer.apple.com/documentation/cloudkit/ckrecord/reference)):

- A reference is a pointer from one record to another, as an id of the “parent” record contained in a child record’s field
- References allow you to do cascade deletes, deleting child records when parent is deleted
- You can create a reference from a CKRecord object, or from a CKRecordID if you know the ID but don’t have the object in memory

### Assets ([`CKAsset`](https://developer.apple.com/documentation/cloudkit/ckasset)):

- An asset is an unstructured piece of data, basically a binary file
- Assets are downloaded and uploaded from/to files on disk, not from memory
- An asset is always owned by a record, and is deleted when the record is deleted
- Transport of assets is optimized so that only the minimal amount of data is transferred


## APIs

There are two different APIs for managing CloudKit data: “operational API” and “convenience API”. The operational API has every possible operation you might need, the convenience API is more convenient.

Start with the convenience API, use operational API for tweaking and overriding options if needed.

CloudKit APIs for saving/fetching data are asynchronous - there is no SDK-managed local data, everything needs to go over the network unless you manually cache it. So it’s absolutely necessary to properly handle error cases - every network call can fail and your app needs to be prepared for this.

### Convenience API:

```objc
[publicDatabase saveRecord:obj completionHandler: ^(…) { … }];
[publicDatabase retchRecordWithID:recordID completionHandler: ^(…) { … }];
```

### Queries:

For any large database, or the shared public database, you shouldn’t try to keep a copy of the whole database on disk and sync all of it, but instead fetch what you need on demand - for this you can use queries.

A query ([`CKQuery`](https://developer.apple.com/documentation/cloudkit/ckquery)) allows you to fetch a list of records matching some conditions. Query can specify a [`RecordType`](https://developer.apple.com/documentation/cloudkit/ckrecord/recordtype), [`NSPredicate`](https://developer.apple.com/documentation/foundation/nspredicate) and optionally [NSSortDescriptors](https://developer.apple.com/documentation/foundation/nssortdescriptor).

A subset of the predicate language is supported, if something is not supported you’ll get an exception. Predicates such as “equal”, “greater than”, “distance to location”, string tokenizing, and OR / AND are supported.

```objc
[publicDatabase performQuery:query inZoneWithID:nil completionHandler: ^(…) { … }];
```

### Subscriptions:

If you repeatedly run the same query, polling for the same data, you can ask the server to run the query for you and notify you immediately when a new record is added. You do that by creating a subscription ([`CKSubscription`](https://developer.apple.com/documentation/cloudkit/cksubscription)).

A subscription includes: [`RecordType`](https://developer.apple.com/documentation/cloudkit/ckrecord/recordtype), [`NSPredicate`](https://developer.apple.com/documentation/foundation/nspredicate) and push configuration ([`CKNotificationInfo`](https://developer.apple.com/documentation/cloudkit/cksubscription/notificationinfo)). Your app is notified of changes through a push notification with some additional data.

```objc
CKSubscription *subscription =
  [[CKSubscription alloc] initWithRecordType:@"Party"
                                   predicate:predicate
                                     options:CKSubscriptionOptionsFiresOnRecordCreation];

[publicDatabase saveSubscription:subscription completionHandler: ^(…) { … }];
```

Pushes are handled through the usual push API:

```objc
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  …
}
```

Build a [`CKNotification`](https://developer.apple.com/documentation/cloudkit/cknotification) object from the user info:

```objc
CKNotification *notification = [CKNotification notificationFromRemoteNotificationDictionary:userInfo];

NSString *alertBody = notification.alertBody;

if (notification.notificationType == CKNotificationTypeQuery) {
    CKQueryNotification *queryNotification = notification;
    CKRecordID *recordID = [queryNotification recordID];
}
```

### Handling user accounts:

Your application does not get direct access to any user identifiers like iCloud email address. Instead, in each container each user is represented as a unique ID within that container that doesn’t change unless the user switches to another account. The same user will have a different ID in a different CloudKit container (the ID is an instance of [`CKRecordID`](https://developer.apple.com/documentation/cloudkit/ckrecord/id)).

```objc
[[CKContainer defaultContainer] fetchUserRecordIDWithCompletionHandler: ^(…) { … }];
```

Each user has a user record representing them (which is almost like any other record) with their user id and record type = [`CKRecordTypeUserRecord`](https://developer.apple.com/documentation/cloudkit/ckrecordtypeuserrecord) - one in the private database, and another with the same ID in the public database.

You can set and read any key-value data on this record like on other records; however, these records aren’t created by you and can’t be queried to get a list of users.

```objc
[publicDatabase fetchRecordWithID:userRecordID completionHandler: ^(…) { … }];
```

### User discovery:

You can ask the user to allow you to make them discoverable by other users (they get a request popup). If they agree, they can be looked up by user ID, specific email, or by fetching a list of all users matching your user’s contacts from the address book (this doesn’t give your app access to the address book itself, just a list of matching users). You get back record IDs, first & last names of users, but no emails.

```objc
[defaultContainer discoverAllContactUserInfosWithCompletionHandler: ^(…) { … }];
```

This returns an array of `CKDiscoveredUserInfo` objects with properties `userRecordID`, `firstName`, `lastName`.

[note: this API has been replaced since then with a new one that returns [`CKUserIdentity`](https://developer.apple.com/documentation/cloudkit/ckuseridentity) objects]


## When to use CloudKit vs. other APIs?

CloudKit doesn’t replace or deprecate any existing iCloud APIs [note: no longer true as of 2016 :)], it’s just an additional tool.

Key-value store:

- asynchronous, small amounts of data
- mostly for application preferences

iCloud Drive:

- works on files and folders
- on OS X it makes a full offline cache of the drive
- good for document-centric apps

iCloud Core Data:

- built on top of iCloud Drive
- good for keeping private, structured data (custom databases) in sync
- note: the whole data set is downloaded to each device

CloudKit:

- good for sharing public data between users, both structured data and large files
- good for large data sets where not every device needs to have a copy of the whole database
- for attaching some data to the user’s identity and sharing info between users that know each other
- more low-level, your app is in control of when any information is downloaded or uploaded to the iCloud servers, and has responsibility for handling sync
