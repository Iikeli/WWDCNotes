# CloudKit Tips and Tricks

CloudKit makes it easy to store and retrieve any kind of data from iCloud. Dive into the API with the CloudKit framework team as they explore some of its lesser-known features, explore best practices around subscriptions and queries, and reveal its hidden gems.

@Metadata {
   @TitleHeading("WWDC15")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc15/715", purpose: link, label: "Watch Video (54 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



## Error Handling

### Accounts:

To check the account status of the current user:

```swift
container.accountStatusWithCompletionHandler { status, error in … }
```

All APIs that fail because they require an authenticated user return [`CKErrorNotAuthenticated`](https://developer.apple.com/documentation/cloudkit/ckerror/code/notauthenticated).

You can now subscribe for [`CKAccountChangedNotification`](https://developer.apple.com/documentation/foundation/nsnotification/name/1399172-ckaccountchanged) to be notified when account status changes.

You should avoid showing alerts to the user about a missing account - simply disable parts of the UI that require an account, and reenable them when you get a notification that an account is now available.

### Network errors:

Network connection errors that you may sometimes get: [`CKErrorNetworkFailure`](https://developer.apple.com/documentation/cloudkit/ckerror/code/networkfailure), [`CKErrorServiceUnavailable`](https://developer.apple.com/documentation/cloudkit/ckerror/code/serviceunavailable), [`CKErrorZoneBusy`](https://developer.apple.com/documentation/cloudkit/ckerror/code/zonebusy), [`CKErrorRequestRateLimited`](https://developer.apple.com/documentation/cloudkit/ckerror/code/requestratelimited). These errors include a key [`CKErrorRetryAfterKey`](https://developer.apple.com/documentation/cloudkit/ckerrorretryafterkey) in their user info dictionary that tells you how long you should wait before retrying.

### Handling conflicts:

If you try to save a record that has been modified in the meantime on the server (meaning: the record change tag you’re sending with the save request is outdated), you will receive the error [`CKErrorServerRecordChanged`](https://developer.apple.com/documentation/cloudkit/ckerror/code/serverrecordchanged).

There is no magic happening behind the scenes in such case, CloudKit doesn’t make assumptions about how you want to resolve conflicts, you need to handle this yourself. However, the SDK provides you the necessary information in the `userInfo`:

- [`CKRecordChangedErrorClientRecordKey`](https://developer.apple.com/documentation/cloudkit/ckrecordchangederrorclientrecordkey) - what you tried to save
- [`CKRecordChangedErrorAncestorRecordKey`](https://developer.apple.com/documentation/cloudkit/ckrecordchangederrorserverrecordkey) - the original version
- [`CKRecordChangedErrorServerRecordKey`](https://developer.apple.com/documentation/cloudkit/ckrecordchangederrorancestorrecordkey) - what is currently on the server 

Usually you will want to resolve the conflict by applying the same changes that you did on the original record to the current server version of the record.


## CloudKit Operations

### Batch operations:

If you create and save a lot of records in one go, each of them will create a separate network request, and making a lot of requests in a short period means you’re likely to hit some kind of rate limit and they will be queued up. To avoid making multiple similar requests, you can use the [`CKOperation`](https://developer.apple.com/documentation/cloudkit/ckoperation) API. Almost every convenience API method that works on one record at a time has a `CKOperation` counterpart that allows you to work on a batch of records.

For saving multiple records, use [`CKModifyRecordsOperation`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation):

```swift
class CKModifyRecordsOperation: CKDatabaseOperation {
  convenience init(recordsToSave: [CKRecord]?, recordIDsToDelete: [CKRecordID]?)
}
```

Note: there are certain limits on how large batch operations you can make (number of items in the request and total request size). This doesn’t include the size of saved binary assets, just the record field data.

If you hit this limit, you will get the [`CKErrorLimitExceeded`](https://developer.apple.com/documentation/cloudkit/ckerror/code/limitexceeded) error. In that case, the best solution is usually to try to divide the batch in half and make two requests.

If one or more records in the batch can’t be saved, you will get [`CKErrorPartialFailure`](https://developer.apple.com/documentation/cloudkit/ckerror/code/partialfailure). From this error’s `userInfo` you can get a dictionary with specific record errors under [`CKPartialErrorsByItemIDKey`](https://developer.apple.com/documentation/cloudkit/ckpartialerrorsbyitemidkey).

In a standard zone, in such scenario some records will be saved and those with errors won’t. In a custom zone you can make an atomic update - in this case, in case of a problem with some of the records, no records will actually be saved, but instead you will get an error [`CKErrorBatchRequestFailed`](https://developer.apple.com/documentation/cloudkit/ckerror/code/batchrequestfailed) for those that could have been saved but weren’t.


### Queries:

If you expect a query to return a large number of records, but you only need a small number of them at a time, you can use the [`CKQueryOperation.resultsLimit`](https://developer.apple.com/documentation/cloudkit/ckqueryoperation/1515078-resultslimit) property (also available on [`CKFetchRecordChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordchangesoperation), [`CKFetchNotificationChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchnotificationchangesoperation)).

When limiting the number of records, you will usually also want to set the query's [`sortDescriptors`](https://developer.apple.com/documentation/cloudkit/ckquery/1413121-sortdescriptors) to e.g. sort records by oldest or newest first. You can use the [`creationDate`](https://developer.apple.com/documentation/cloudkit/ckrecord/1462223-creationdate) key which is automatically added to all saved records regardless of type.

To implement pagination and get further pages beyond the first one, use the [`CKQueryCursor`](https://developer.apple.com/documentation/cloudkit/ckqueryoperation/cursor) object that you get in response to the [`queryCompletionBlock`](https://developer.apple.com/documentation/cloudkit/ckqueryoperation/1515067-querycompletionblock) callback. Then, initialize the next `CKOperation` passing it the cursor object in the argument to the initializer.

If you don’t need all information contained in a record immediately, you can also use the [`desiredKeys`](https://developer.apple.com/documentation/cloudkit/ckqueryoperation/3003370-desiredkeys) property to only download the keys you want; e.g. download the record’s thumbnail image, but not a full-size photo (also available on [`CKFetchRecordsOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordsoperation), [`CKFetchRecordChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordchangesoperation)).


### Maintaining a local cache:

Let's say you want to keep some subset of all data completely cached on all local devices for quicker access (e.g. user’s personal notes).

You have two options:

- use [`CKQueryOperation`](https://developer.apple.com/documentation/cloudkit/ckqueryoperation) to fetch all records and synchronize them manually
- make a custom zone and use delta downloads using [`CKFetchRecordChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordchangesoperation)

When saving fetched records to a local database, you should save CKRecord’s system fields like the change tag together with your own data. To do that, you can use [`encodeSystemFieldsWithCoder`](https://developer.apple.com/documentation/cloudkit/ckrecord/1462200-encodesystemfields):

```swift
let data = NSMutableData()
let archiver = NSKeyedArchiver(forWritingWithMutableData: data)
archiver.requiresSecureCoding = true
record.encodeSystemFieldsWithCoder(archiver)
archiver.finishEncoding()
```

When restoring a record from the local storage, you don’t have to set all its data fields - it’s fine to only set those you want to change.

To synchronize any changes from the server, create a subscription subscribing to a given record type using silent notifications, and use [`CKFetchRecordChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordchangesoperation) to fetch all recent changes when notified.

Subscriptions ([`CKSubscription`](https://developer.apple.com/documentation/cloudkit/cksubscription)) are persistent queries on the server that send remote notifications about a relevant change - either in a specific record set (query subscription) or in the whole zone (zone subscription).

To get CloudKit subscription notifications, you need to follow the usual setup for push notifications:

- have push notification capability enabled for your app
- call [`registerForRemoteNotifications()`](https://developer.apple.com/documentation/uikit/uiapplication/1623078-registerforremotenotifications)
- call [`registerUserNotificationSettings(…)`](https://developer.apple.com/documentation/uikit/uiapplication/1622932-registerusernotificationsettings) if you want to show notifications to the user

To ask for silent subscription notifications, configure the [`CKNotificationInfo`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo) object appropriately:

- set the [`shouldSendContentAvailable`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo/1515110-shouldsendcontentavailable) key
- do not set any of the UI-related keys: [`alertBody`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo/1515270-alertbody), [`shouldBadge`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo/1514996-shouldbadge), [`soundName`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo/1514987-soundname)

Notification priorities: a notification is high priority if it has any UI keys set, otherwise it’s medium priority.

For silent notifications, the [`UIApplicationDelegate`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate) will receive the following callback:

```swift
func application(application: UIApplication,
    didReceiveRemoteNotification: [NSObject: AnyObject],
    fetchCompletionHandler: (UIBackgroundFetchResult) -> Void) { … }
```

Remember that push notification delivery in general is “best effort” - pushes can be dropped if many are received in a short period of time or because of network issues. Silent notifications may also be additionally delayed if the system is waiting for better conditions.

When you receive a notification, use [`CKFetchNotificationChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchnotificationchangesoperation) to check the server’s notification collection for any notifications you might have missed.

You may want to use a [`UIApplication`](https://developer.apple.com/documentation/uikit/uiapplication) background task ([`beginBackgroundTaskWithName(…)`](https://developer.apple.com/documentation/uikit/uiapplication/1623031-beginbackgroundtask)) for syncing tasks.

Interactive notifications: you can now make CloudKit notifications interactive (e.g. show action buttons) by setting the [`category`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo/1515082-category) key on [`CKNotificationInfo`](https://developer.apple.com/documentation/cloudkit/cknotificationinfo).


## Other performance tips

CloudKit is a highly asynchronous API, most operations require a network call and take some time to execute. You will often want to make a series of operations that have some dependencies between them.

Things to keep in mind:

- however you implement task handling, remember to always handle all errors
- never block the main thread with an operation in progress
- don’t nest calls to the convenience API methods, creating a “callback hell”
- don’t use locks/semaphores to wait for an API call to finish
- instead, use the [`addDependency()`](https://developer.apple.com/documentation/foundation/operation/1412859-adddependency) API in `CKOperation` to add dependencies between operations:

```swift
let firstFetch = CKFetchRecordsOperation(…)
let secondFetch = CKFetchRecordsOperation(…)
secondFetch.addDependency(firstFetch)

let queue = NSOperationQueue()
queue.addOperations([firstFetch, secondFetch], waitUntilFinished: false)
```

- use the [`qualityOfService`](https://developer.apple.com/documentation/foundation/operation/1413553-qualityofservice) property on `NSOperation` to indicate which operations are something you need in the UI and which are low priority background operations
- there used to be a `usesBackgroundSession` property on `CKOperation` too, but it’s deprecated now ⭢ use quality of service for this
- QoS = `.utility` and `.background` use discretionary networking, use `.userInteractive` and `.userInitiated` for high priority tasks
  - note: `.utility` QoS is the default if you don’t change it! [the video says `.background` but a later one says `.utility`]
