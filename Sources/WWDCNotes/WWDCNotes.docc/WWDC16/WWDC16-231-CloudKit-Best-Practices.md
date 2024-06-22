# CloudKit Best Practices

CloudKit keeps app data updated across iOS, macOS, watchOS, tvOS, and the web so you can focus on building your app. Hear best practices from the CloudKit engineering team about how to take advantage of the APIs and push notifications in order to provide your users with the best experience. Learn about the ways Apple apps use CloudKit and how you can apply the same approaches in your app.

@Metadata {
   @TitleHeading("WWDC16")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc16/231", purpose: link, label: "Watch Video (42 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



## Short CloudKit overview

Apple uses CloudKit in their applications, so you can be confident that it scales, because for Apple it scales to hundreds of millions of users.

CloudKit lets you focus on building your applications and not worry about building backend services for them. It provides your users automatic authentication - if the user is logged in to iCloud on their device, they don’t need to log in separately in your app.

A CloudKit container now includes 3 databases:

- public database - for data visible to everyone
- private database - for a given user’s private data
- new this year: shared database - for user data that they decided to share with others

Zones:

- public database has 1 default zone
- private database has a default zone and it can have one or more custom zones
- shared database includes some number of shared zones

A record always exists in a specific zone.


## Building an app with a sync feature

A common use case (e.g. Notes app):

- user creates some data/records/documents on one of their devices
- later, they open another device and they expect to see these documents there and be able to read/edit them

The way this is implemented is that CloudKit needs to be the source of truth, and the devices should maintain a local cache of all the app data and synchronize it using CloudKit.

The recommended workflow:

1. On app launch, fetch changes from the server
2. Subscribe to any future changes
3. Fetch changes when you receive a push


### Subscriptions:

Subscriptions let you ask the server to notify you whenever a change happens in the specified set of data. Previously you could subscribe to a specific query to a record type or to all changes in a zone.

New in iOS 10 - [`CKDatabaseSubscription`](https://developer.apple.com/documentation/cloudkit/ckdatabasesubscription) - lets you subscribe to all changes in the whole database (private or shared).

Types of subscription notifications:

1. Silent push:

```swift
let notificationInfo = CKNotificationInfo()

// we only set this, but none of the UI related keys
notificationInfo.shouldSendContentAvailable = true

// do this once. no need to ask the user for push notifications permission,
// since we won't show any visible notifications
application.registerForRemoteNotifications(…)
```

2. Visual notification:

```swift
let notificationInfo = CKNotificationInfo()

// set any of these
notificationInfo.shouldBadge = true
notificationInfo.alertBody = "alertBody"
notificationInfo.soundName = "default"

// we need to prompt the user for push notification access:
application.registerUserNotificationSettings(…)
application.registerForRemoteNotifications(…)
```

Remember that push notifications can be coalesced, so you may only get one out of a series. Push notifications tell you that *something* has changed, but not necessarily every single thing that has changed.


### Creating a subscription:

This only needs to be done the first time you launch an app - so we set a flag when we create a subscription and the next time we skip this part.

```swift
if subscriptionIsLocallyCached { return }

let subscription = CKDatabaseSubscription(subscriptionID: "shared-changes")

let notificationInfo = CKNotificationInfo()
notificationInfo.shouldSendContentAvailable = true
subscription.notificationInfo = notificationInfo

let operation = CKModifySubscriptionsOperation(
    subscriptionsToSave: [subscription],
    subscriptionIDsToDelete: []
)

operation.modifySubscriptionsCompletionBlock = { …
    if error != nil {
        …
    } else {
        self.subscriptionIsLocallyCached = true
    }
}

operation.qualityOfService = .utility
self.sharedDB.add(operation)
```

### Listening for pushes:

- turn on “Remote notifications” and “Background fetch” capabilities

```swift
func application(_ application: UIApplication,
    didReceiveRemoteNotification userInfo: [NSObject: AnyObject],
    fetchCompletionHandler completionHandler: (UIBackgroundFetchResult) -> Void) {

    let dict = userInfo as! [String: NSObject]
    let notification = CKNotification(fromRemoteNotificationDictionary: dict)

    if notification.subscriptionID == "shared-changes" {
        fetchSharedChanges {
              completionHandler(.newData)
        }
    }
}
```

### Fetching the changes:

Steps:

- ask in which zones something was changed (in shared db - because there may be new zones added when a new user shares some content)
- ask which records have changed in each relevant zone

The server will not send you pushes about the changes you’re doing on this device, but you may receive those changes you’ve done on the list when fetching a delta download.

[`fetchAllChanges`](https://developer.apple.com/documentation/cloudkit/ckfetchdatabasechangesoperation/1640473-fetchallchanges): previously, in some operations you had to manually check for a flag that says there are more results waiting for you that you need to manually request (i.e. another page). Now, CloudKit does the paging automatically for you if `fetchAllChanges = true` (which is the default).

```swift
func fetchSharedChanges(_ callback: () -> Void) {
    let changesOperation = CKFetchDatabaseChangesOperation(
        previousServerChangeToken: sharedDBChangeToken  // cached between runs
    )

    // this gives you IDs of changed zones
    changesOperation.recordZoneWithIDChangedBlock = { … }

    // this gives you IDs of deleted zones
    changesOperation.recordZoneWithIDWasDeletedBlock = { … }

    // this gives you the current change token which you need to save
    // may be called multiple times if the operation fetches multiple pages of content
    // save the token each time, so in case of an error you don’t repeat all work
    changesOperation.changeTokenUpdatedBlock = { … }

    changesOperation.fetchDatabaseChangesCompletionBlock = {
        (newToken: CKServerChangeToken?, more: Bool, error: NSError?) -> Void in

        self.sharedDBChangeToken = newToken
        self.fetchZoneChanges(callback)
    }

    self.sharedDB.add(operation)
}
```

`fetchZoneChanges()` looks very similar, but fetches changes for a specific zone using [`CKFetchRecordZoneChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordzonechangesoperation) (you pass it a list of zones).


## CloudKit best practices

### Automatic authentication:

CloudKit allows you to authenticate users (if they’re logged in to iCloud) without requiring any private information.

You use the CloudKit user record for authentication. The user record is unique per container and never changes for that user.

```swift
container.fetchUserRecordID(completionHandler: (CKRecordID?, NSError?) -> Void)
```

### CKOperation API:

The convenience API works on single items and it’s simpler to use. Every convenience API call has a [`CKOperation`](https://developer.apple.com/documentation/cloudkit/ckoperation) counterpart that lets you perform an operation on a batch of records.

The [`CKOperation`](https://developer.apple.com/documentation/cloudkit/ckoperation) also has other advantages - for example, it lets you:

- set up dependencies between operations
- specify quality of service and queue priorities
- cancel operations that have started executing
- specify if you want the operation to work over cellular network
- limit the number of records or set of fetched keys
- report progress
- … and everything that [`NSOperation`](https://developer.apple.com/documentation/foundation/operation) provides

Watch the "[Advanced NSOperations](https://developer.apple.com/videos/play/wwdc2015/226/)" talk from 2015 to learn more about [`NSOperation`](https://developer.apple.com/documentation/foundation/operation).

### Quality of service:

QoS: select a quality of service (`.userInteractive` / `.userInitiated` / `.utility` / `.background`) depending on the task priority.

- default is `.utility`
- `.utility` and below enable discretionary networking

Discretionary networking means that:

- the system decides when is the best moment to run your request, so it may take longer than you expect
- however, all network failures will be automatically retried for you
- the request gets a timeout period of 7 days by default

### Long lived operations:

If you have some operations that you want to continue/retry if they don’t manage to complete by the time your app is terminated, iOS 9.3 adds “CloudKit long lived operations”. Once you run such operation, the system will finish it even if the app is killed by the system or the user. The request is executed even if your app isn’t running, the result is cached and is returned to you once the app restarts. Results are kept by the OS for at least 24 hours.

To use this API:

- set [`isLongLived`](https://developer.apple.com/documentation/cloudkit/ckoperation/1452374-islonglived) = `true` on [`CKOperation`](https://developer.apple.com/documentation/cloudkit/ckoperation)
- save the operation’s [`operationID`](https://developer.apple.com/documentation/cloudkit/ckoperation/3003367-operationid)
- use [`CKContainer.fetchLongLivedOperation(withId:)`](https://developer.apple.com/documentation/cloudkit/ckcontainer/3003356-fetchlonglivedoperation) to get the operation object back
- set completion blocks and run it again just like a new one

```swift
CKContainer.default().fetchLongLivedOperation(withID: myOpID) {
    (operation: CKOperation?, error: NSError?) in

    let fetchRecords = operation as! CKFetchRecordsOperation
    fetchRecords.fetchRecordsCompletionBlock = { … }

    CKContainer.default().privateCloudDatabase.add(fetchRecords)
}
```

### Parent references:

A new type of reference added this year to help you better model data, especially with sharing in mind. If your app supports sharing, it’s recommended that you set the parent reference to create a hierarchy between records.

Example: Album ⭢ list of photos

```swift
let photoRecord = CKRecord(recordType: "photo")
photoRecord.setParent(albumRecordID)
```

What this gives you: when the user shares the album record, the whole record hierarchy under this album (photos and other data) will also be shared.

### Types of errors:

#### 1) Fatal error (bad request)

Error codes like:

- [`.internalError`](https://developer.apple.com/documentation/cloudkit/ckerror/2325203-internalerror)
- [`.serverRejectedRequest`](https://developer.apple.com/documentation/cloudkit/ckerror/2325219-serverrejectedrequest)
- [`.invalidArguments`](https://developer.apple.com/documentation/cloudkit/ckerror/2325210-invalidarguments)
- [`.permissionFailure`](https://developer.apple.com/documentation/cloudkit/ckerror/2325225-permissionfailure)

In this case, you should show an alert to the user and tell them this can’t be executed.

#### 2) Connection/server error

Error codes like:

- [`.zoneBusy`](https://developer.apple.com/documentation/cloudkit/ckerror/2325220-zonebusy)
- [`.serviceUnavailable`](https://developer.apple.com/documentation/cloudkit/ckerror/2325227-serviceunavailable)
- [`.requestRateLimited`](https://developer.apple.com/documentation/cloudkit/ckerror/2325202-requestratelimited)

In this case, check for [`CKErrorRetryAfterKey`](https://developer.apple.com/documentation/cloudkit/ckerrorretryafterkey) and retry after specified time.

#### 3) Errors that are returned before connection is even made

[`.networkUnavailable`](https://developer.apple.com/documentation/cloudkit/ckerror/2325209-networkunavailable)

- you should monitor network reachability ([`SCNetworkReachability`](https://developer.apple.com/documentation/systemconfiguration/scnetworkreachability-g7d)) and retry when the device is connected again

[`.notAuthenticated`](https://developer.apple.com/documentation/cloudkit/ckerror/2325221-notauthenticated)

- when the user is not logged in and can’t access their private database
- you should register at startup for [`CKAccountChangedNotification`](https://developer.apple.com/documentation/foundation/nsnotification/name/1399172-ckaccountchanged), and when it fires, recheck account status and update the UI
