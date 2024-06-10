# Sync to iCloud with CKSyncEngine

Discover how CKSyncEngine can help you sync people’s CloudKit data to iCloud. Learn how you can reduce the amount of code in your app when you let the system handle scheduling for your sync operations. We’ll share how you can automatically benefit from enhanced performance as CloudKit evolves, explore testing for your sync implementation, and more.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10188", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speakers: Tim Mahoney and Aamer Husain, CloudKit Engineers

## The state of sync

- Sync is expected
- Sync is not magic
- Simpler is often better

### APIs

- `NSPersistentCloudKitContainer`: Full-stack solution that includes local persistence.
- **NEW:** `CKSyncEngine`: Bring your own local persistence.
- `CKDatabase` and `CKOperation`: More fine-grained control.

> ... if you want to sync with CloudKit, and if you're not using NSPersistentCloudKitContainer, you should use CKSyncEngine. Sync involves many moving parts, and using a higher level API like CKSyncEngine can help reduce complexity and improve your app's sync experience.

## Meet CKSyncEngine

> When you use CKSyncEngine, the amount of sync code you have to write becomes much smaller and more focused. You only have to handle the things that are specific to your app, and the sync engine handles the rest.

- Convenient but flexible
- Private and shared data
- Backward compatible
- Used by many Apple apps (including Freeform)
- `NSUbiquitousKeyValueStore` was rewritten on top of the sync engine

### Already have a custom CloudKit sync implementation?

- Consider switching
- Less maintenance
- Future enhancements
- Smaller API
- Submit feedback if you are missing features

### General flow of operation

On saving data to CloudKit:
> Generally, the sync engine acts as a conduit of data between your app and the CloudKit server. Your app communicates with the sync engine in terms of records and zones.
> 
> When there are changes to save, your app gives them to the sync engine. When it fetches these changes on another device, it gives them to your app. That said, when the sync engine has work to do, it doesn't always do it immediately.
> 
> If it needs to communicate with the server, it'll first consult with the system task scheduler. This is the same scheduler used across the OS for background task management, and it makes sure that the device is ready to sync.
> 
> Once the device is ready, the scheduler runs the task, and the sync engine talks with the server.
> 
> This is the basic flow of operation for the sync engine. More specifically, what does it look like when the sync engine sends changes to the server?
> 
> - First, someone makes a modification to the data. Maybe they typed something or they flipped a switch or deleted an object.
> - Then, your app tells the sync engine that there's a pending change to send to the server. This lets the sync engine know that it has work to do.
> - Next, the sync engine submits a task to the scheduler. Once the device is ready, the scheduler runs the task.
> - When the task runs, the sync engine starts the process of sending changes to the server. In order to do that, it asks your app for the next batch of changes to send.
>   - If someone made a single modification, you might only have one pending change. 
>   - However, if someone imports a huge database of new data, you might have hundreds or thousands of changes.
> - Since there's a limit on how much can be sent to the server in a single request, the sync engine asks for these changes in batches. 
>   - This also helps to reduce memory overhead by not bringing any records into memory until they're actually needed.
> - After you provide the next batch, the sync engine sends it to the server. 
> - The server responds with the result of the operation, including any information about the success or failure of these changes.
> - Once the request finishes, the sync engine calls back to your app with the result.
> - This is your opportunity to react to the success or failure of the operation.
> - If you have any more pending changes, the sync engine will continue to ask for batches until there's nothing left to send.

On receiving data from CloudKit:
> When the server receives a new change, it sends a push notification to the other devices that have access to that data.
> 
> CKSyncEngine automatically listens for these push notifications in your app. 
> 
> - When it receives a notification, it submits a task to the scheduler.
> - When the scheduler task runs, the sync engine fetches from the server.
> - When it fetches new changes, it gives them to your app. This is your chance to persist these changes locally and show them in the UI.
> 
> And that's the basic flow of operations when using the sync engine.

#### The scheduler

- Enables automatic sync
- Monitors system condition
- Ensures proper balance between user experience and device resources
- Often fast

**Trust the scheduler!**

- Rely on automatic sync
- Easier, more efficient
- Manual sync when necessary
- Testing with manual sync

> In general, we recommend that you rely on automatic sync scheduling. But, we understand that there are valid use cases for manual syncing, and the sync engine has API to do that when necessary.

## Getting started

- Learn about the fundamental types in CloudKit: `CKRecord` and `CKRecordZone`.
- Enable the CloudKit capability in your Xcode project.
- Enable the Remote Push notifications capability in your Xcode project as the sync engine relies on push notifications.
- Initialize `CKSyncEngine` on app launch to start listening for push notifications and scheduler tasks.
  - Provide an object which conforms to the `CKSyncEngineDelegate`.
  - Provide the last known version fo the sync engine state.
  - The delegate should persist the `Event.stateUpdate` when received, for next app launch.

```swift
actor MySyncManager : CKSyncEngineDelegate {
    
    init(container: CKContainer, localPersistence: MyLocalPersistence) {
        let configuration = CKSyncEngine.Configuration(
            database: container.privateCloudDatabase,
            stateSerialization: localPersistence.lastKnownSyncEngineState,
            delegate: self
        )
        self.syncEngine = CKSyncEngine(configuration)
    }
    
    func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async {
        switch event {
        case .stateUpdate(let stateUpdate):
            self.localPersistence.lastKnownSyncEngineState = stateUpdate.stateSerialization
        }
    }
}
```

## Using CKSyncEngine

- Add changes in `CKSyncEngine.State`.
- Implement the delegate method `nextRecordZoneChangeBatch`.
- Handle change events: `sentDatabaseChanges` and `sentRecordZoneChanges`.

### Sending changes to the server

```swift
func userDidEditData(recordID: CKRecord.ID) {
    // Tell the sync engine we need to send this data to the server.
    self.syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
}

func nextRecordZoneChangeBatch(
    _ context: CKSyncEngine.SendChangesContext, 
    syncEngine: CKSyncEngine
) async -> CKSyncEngine.RecordZoneChangeBatch? {

    let changes = syncEngine.state.pendingRecordZoneChanges.filter { 
        context.options.zoneIDs.contains($0.recordID.zoneID) 
    }

    return await CKSyncEngine.RecordZoneChangeBatch(pendingChanges: changes) { recordID in
        self.recordToSave(for: recordID)
    }
}
```

### Fetching changes from the server

- `CKSyncEngine` automatically fetches
- Handle fetch events: `fetchedDatabaseChanges` and `fetchedRecordZoneChanges`.
- Handle optional events: `willFetchChanges` and `didFetchChanges`.
  - Handing these events may be useful if you want to perform any setup or cleanup tasks before or after fetching changes.

```swift
func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async {
    switch event {
        
    case .fetchedRecordZoneChanges(let recordZoneChanges):
        for modifications in recordZoneChanges.modifications {
            // Persist the fetched modification locally
        }

        for deletions in recordZoneChanges.deletions {
            // Remove the deleted data locally
        }

    case .fetchedDatabaseChanges(let databaseChanges):      
        for modifications in databaseChanges.modifications {
            // Persist the fetched modification locally
        }
      
        for deletions in databaseChanges.deletions { 
            // Remove the deleted data locally
        }

    // Perform any setup/cleanup necessary
    case .willFetchChanges, .didFetchChanges:
        break
      
    case .sentRecordZoneChanges(let sentChanges):

        for failedSave in sentChanges.failedRecordSaves {
            let recordID = failedSave.record.recordID

            switch failedSave.error.code {

            case .serverRecordChanged:
                if let serverRecord = failedSave.error.serverRecord {
                    // Merge server record into local data
                    syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
                }
            
            case .zoneNotFound: 
                // Tried to save a record, but the zone doesn't exist yet.
                syncEngine.state.add(pendingDatabaseChanges: [ .save(recordID.zoneID) ])
                syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
             
            // CKSyncEngine will automatically handle these errors
            case .networkFailure, .networkUnavailable, .serviceUnavailable, .requestRateLimited:
                break
              
            // An unknown error occurred
            default:
                break
            }
        }
      
    case .accountChange(let event):
        switch event.changeType {

        // Prepare for new user
        case .signIn:
            break
          
        // Delete local data
        case .signOut:
            break
          
        // Delete local data and prepare for new user
        case .switchAccounts: 
            break
        }
    }
}
```

### Error handling

- `CKSyncEngine` handles transient errors
- Automatic retry
- You handle application errors
- Re-schedule sync to retry

```swift
func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async {
    switch event {
        
    case .fetchedRecordZoneChanges(let recordZoneChanges):
        for modifications in recordZoneChanges.modifications {
            // Persist the fetched modification locally
        }

        for deletions in recordZoneChanges.deletions {
            // Remove the deleted data locally
        }

    case .fetchedDatabaseChanges(let databaseChanges):      
        for modifications in databaseChanges.modifications {
            // Persist the fetched modification locally
        }
      
        for deletions in databaseChanges.deletions { 
            // Remove the deleted data locally
        }

    // Perform any setup/cleanup necessary
    case .willFetchChanges, .didFetchChanges:
        break
      
    case .sentRecordZoneChanges(let sentChanges):

        for failedSave in sentChanges.failedRecordSaves {
            let recordID = failedSave.record.recordID

            switch failedSave.error.code {

            case .serverRecordChanged:
                if let serverRecord = failedSave.error.serverRecord {
                    // Merge server record into local data
                    syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
                }
            
            case .zoneNotFound: 
                // Tried to save a record, but the zone doesn't exist yet.
                syncEngine.state.add(pendingDatabaseChanges: [ .save(recordID.zoneID) ])
                syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
             
            // CKSyncEngine will automatically handle these errors
            case .networkFailure, .networkUnavailable, .serviceUnavailable, .requestRateLimited:
                break
              
            // An unknown error occurred
            default:
                break
            }
        }
      
    case .accountChange(let event):
        switch event.changeType {

        // Prepare for new user
        case .signIn:
            break
          
        // Delete local data
        case .signOut:
            break
          
        // Delete local data and prepare for new user
        case .switchAccounts: 
            break
        }
    }
}
```

#### Transient errors

> [The following are] examples of transient errors that the sync engine will handle for you. You'll still receive these errors for your awareness but you do not need to take action in response to them. The sync engine will automatically retry for these errors when system conditions permit.

```swift
case .sentRecordZoneChanges(let sentChanges):
  case networkFailure, // The network was available but encountered an error
  case .networkUnavailable, // The network was not available
  case .serviceUnavailable, // CloudKit was not available
  case .requestRateLimited: // cloudKit has rate limited work and should be attempted later
```

#### Account changes

- `CKSyncEngine` monitors account status/changes
- You handle `.accountChange` events

```swift
case .accountChange(let event):
  switch event.changeType {
    case .signIn, // Prepare for new user
    case .signOut, // Delete local data
    case .switchAccounts: // Delete local data and prepare for new user
  }
```

> The sync engine will not begin syncing with iCloud until there is an account present on the device.

### Sharing data with other users

- Initialize a `CKSyncEngine` for each database
- Existing `CKShare` invitation/acceptance flow

```swift
let databases = [ container.privateCloudDatabase, container.sharedCloudDatabase ]

let syncEngines = databases.map {
    var configuration = CKSyncEngine.Configuration(
        database: $0,
        stateSerialization: lastKnownSyncEngineState($0.databaseScope),
        delegate: self
    )
    return CKSyncEngine(configuration)
}
```

*See more in the Tech Talk "Get the most out of CloudKit Sharing".*

## Testing and Debugging

- You can simulate device-to-device user flows using multiple `CKSyncEngine` instances.
- Simulate specific edge cases by setting `automaticallySync = false`

In this example we simulate two devices using `MySyncManager`. In this example, `MySyncManager` creates a local database and sync engine:
```swift
func testSyncConflict() async throws {
    // Create two local databases to simulate two devices.
    let deviceA = MySyncManager()
    let deviceB = MySyncManager()
    
    // Save a value from the first device to the server.
    deviceA.value = "A"
    try await deviceA.syncEngine.sendChanges()
    
    // Try to save the value from the second device before it fetches changes.
    // The record save should fail with a conflict that includes the current server record.
    // In this example, we expect the value from the server to win.
    deviceB.value = "B"
    XCTAssertThrows(try await deviceB.syncEngine.sendChanges())
    XCTAssertEqual(deviceB.value, "A")
}
```

### Tips for debugging

- Understand the sequence of events
- Logging record/zone IDs and timestamps
- Write tests for user flows
- Look at timestamps when piecing the puzzle together

## Sample code

There is sample code available at [https://github.com/apple/sample-cloudkit-sync-engine](https://github.com/apple/sample-cloudkit-sync-engine).
