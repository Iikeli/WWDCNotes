# There and back again: Data transfer on Apple Watch

Advances in Apple Watch give you more ways to communicate to and from your app, and new audiences to consider. Learn what strategies are available for data communication and how to choose the right tool for the job. Compare and contrast the benefits of using technologies such as iCloud Keychain, Watch Connectivity, Core Data, and more.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10003", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Data communication tools

- iCloud 
  - allows us to share data with all our devices
  - gives us server storage
  - Keychain with iCloud Synchronization
  - CoreData with CloudKit

- Watch Connectivity
  - allows to transfer data between paired devices

- `URLSession` and/or sockets
  - to communicate directly with servers

### How to choose among the communication options

Ask yourself:

- Type of data - What kind of data is it?
- Data source and destination - Where is the data now, and where it needs to be?
- Reliance on companion iOS app - Is the interaction reliant on a companion iOS app?
- Support Family Setup - Do I want to support Family Setup?
- Timing - when does the data need to be at its destination? Can it wait to let the system optimize performance and battery usage for my customer? How frequently is the data going to change?

### Keychain with iCloud Synchronization (WatchOS 6.2+)

- Keychain provides secure storage for passwords, keys, and other sensitive credentials
- Keychain can also store other small bits of shared data, such as a user preference, as long as the information isn't changing frequently
- Keychain items can be synchronized to all of a person's devices
- The items are synchronized when possible based on network availability, battery, and other system conditions
- ⚠️ customers can disable iCloud Keychain synchronization
- ⚠️ Keychain + iClous is not available in all regions
- There are two ways you can benefit from iCloud Synchronization in your app, by using:
  1. Password autofill (with Associated Domains)
  2. Shared Keychain items

#### How to enable Password autofill

1. Add the Associated Domains capability to your target

![][ps1]

2. For your Watch app, add the capability to the WatchKit Extension Target

3. Add a `webcredentials` entry with your domain name (the <kbd>apple-app-site-association</kbd> file to your web server):

```xml
{
  "webcredentials": {
    "apps": [
      "A123456789.com.example.MyiOSApp",
      "A123456789.com.example.MyWatchApp.watchkitapp.watchkitextension"
    ]
  }
}
```

4. Add [`textContentType`][textcontenttype] to your TextFields:

```swift
struct LoginView: View {
  @State private var username = ""
  @State private var password = ""
  
  var body: some View {
    Form {
      TextField("User:", text: $username)
        .textContentType(.username) // 👈🏻
      
      SecureField("Password", text: $password) 
        .textContentType(.password) // 👈🏻
      ...
    }
  }
}
```

#### How to share Keychain items among apps

1. Add Keychain Sharing App Groups Capability to Targets (in all the apps where we want to share these Keychain items):

![][ks1]

This is required to share the items, and helps ensure the security and privacy of your customers' information by preventing access by other apps.

2. For your Watch app, add the capability to your Watch Extension target
3. Add apps to common Keychain Group or App Group - all apps that are going to share the Keychain items need to also share this group

Example on how to add/update a Keychain item:

```swift
func storeToken(_ token: OAuth2Token, for server: String, account: String) throws {
  let query: [String: Any] = [
    kSecClass as String: kSecClassInternetPassword,
    kSecAttrServer as String: server,
    kSecAttrAccount as String: account,
    //                                 👇🏻 this indicates that we want to sync this item in all user devices
    kSecAttrSynchronizable as String: true,
  ]
  
  let tokenData = try encodeToken(token)
  let attributes: [String: Any] = [kSecValueData as String: tokenData]
  
  //                👇🏻 update the item if it exists in the keychain
  let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
  
  guard status != errSecItemNotFound else {
    
    //     👇🏻 add the item if it doesn't exist in the keychain yet
    try addTokenData(tokenData, for: server, account: account)
    return
  }
  
  guard status == errSecSuccess else {
    throw OAuthKeychainError.updateError(status)
  }
}

func addTokenData(_ tokenData: Data, for server: String, account: String) throws {
  let attributes: [String: Any] = [
    kSecClass as String: kSecClassInternetPassword,
    kSecAttrServer as String: server,
    kSecAttrAccount as String: account,
    kSecAttrSynchronizable as String: true,
    kSecValueData as String: tokenData,
  ]
  
  let status = SecItemAdd(attributes as CFDictionary, nil)
  
  guard status == errSecSuccess else {
    throw OAuthKeychainError.addError(status)
  }
}
```

Example on how to retrieve a Keychain item:

```swift
func retrieveToken(for server: String, account: String) throws -> OAuth2Token? {
  let query: [String: Any] = [
    kSecClass as String: kSecClassInternetPassword,
    kSecAttrServer as String: server,
    kSecAttrAccount as String: account,
    kSecAttrSynchronizable as String: true,
    // 👇🏻 tells whether we want the item attributes returned
    kSecReturnAttributes as String: false,
    // 👇🏻 tells whether we want the item data returned
    kSecReturnData as String: true,
  ]
      
  var item: CFTypeRef?
  let status = SecItemCopyMatching(query as CFDictionary, &item)
      
  guard status != errSecItemNotFound else {
    // No token stored for this server account combination.
    return nil
  }
  
  guard status == errSecSuccess else {
    throw OAuthKeychainError.retrievalError(status)
  }
  
  guard let existingItem = item as? [String: Any] else {
    throw OAuthKeychainError.invalidKeychainItemFormat
  }
  
  guard let tokenData = existingItem[kSecValueData as String] as? Data else {
    throw OAuthKeychainError.missingTokenDataFromKeychainItem
  }
  
  do {
    return try JSONDecoder().decode(OAuth2Token.self, from: tokenData)
  } catch {
    throw OAuthKeychainError.tokenDecodingError(error.localizedDescription)
  }
}
```

Example on how to delete a Keychain item:

```swift
func removeToken(for server: String, account: String) throws {
  let query: [String: Any] = [
    kSecClass as String: kSecClassInternetPassword,
    kSecAttrServer as String: server,
    kSecAttrAccount as String: account,
    kSecAttrSynchronizable as String: true,
  ]

  let status = SecItemDelete(query as CFDictionary)
  
  guard status == errSecSuccess || status == errSecItemNotFound else {
    throw OAuthKeychainError.deleteError(status)
  }
}
```

### CoreData with CloudKit

- synchronizes your local database to all of your customer's other devices that share your app's CloudKit container
- CoreData integration with SwiftUI simplifies accessing and displaying data from your database in your Watch application.
- Synchronization of Core Data changes happens based on network availability and system conditions. Don't expect it to be instantaneous, but CloudKit will handle optimizing performance of this synchronization for your app

### Watch Connectivity

- allows you to send data between your Watch app and its companion iPhone app when both devices are within Bluetooth range or on the same Wi-Fi network
- best for:
  - optimizing your customer's experience when they have both your phone and Watch apps installed
  - sharing data only available on either Watch or iPhone

#### Tips

- Activate `WCSession` as early as possible in your app life-cycle - this makes your app available to receive information from its counterpart app as soon as possible
- Understand reachability - None of the background communication requires your counterpart app to be reachable when you send data. But interactive messaging does have reachability requirements
- All `WCSession` delegate functions are called on a non-main serial queue

#### Watch Connectivity features

- Application Context
  - the application context is a single property list dictionary
  - sent to the counterpart app in the background, with the goal of being available when the app wakes up
  - if you update the application context before the previous dictionary is sent (via [`updateApplicationContext(_:)`][updateApplicationContext(_:)], it is replaced by the new value
  - useful for keeping content up to date on the counterpart app when you have new data
  - data that may update frequently

- User Info transfer
  - similar to application context
  - the user info is a single property list dictionary
  - sent sequentially in the background - instead of being a single dictionary that is replaced each time you update it, each user info dictionary transfer is queued and delivered in the order that you enqueued it
  - we can access the queue (via [`outstandingUserInfoTransfers`][outstandingUserInfoTransfers]) and cancel an update

- File transfer
  - similar to User Info, but for files
  - delivered in background (similar to the features above)
  - can access the queue (via [`outstandingFileTransfers`][outstandingFileTransfers]) and cancel transfers
  - received files are stored in the Inbox/Documents directory
  - received files are deleted once `WCSessionDelegate`'s [`session(_:didReceive:)`][session(_:didReceive:)] is called - make sure to move/process the file before returning from this method

- Transfer current complication user info
  - Send Complication-related data to Watch
  - Update up to 50 times/day with an active Complication
  - Sent as soon as possible when budget and connectivity allows
  - Check budget with [`remainingComplicationUserInfoTransfers`][remainingComplicationUserInfoTransfers]
  - Sent as normal user info transfer when no budget remains

- Send message ([`sendMessage(_:replyHandler:errorHandler:)`][sendMessage(_:replyHandler:errorHandler:)])
  - Interactive messages to the counterpart app
  - Send a property list dictionary or data and get a reply
  - Counterpart must be reachable
  - Keep messages small
  - (in the counter app) implement delegate [`session(_:didReceiveMessage:replyHandler:)`][session(_:didReceiveMessage:replyHandler:)] and/or [`session(_:didReceiveMessageData:replyHandler:)`][session(_:didReceiveMessageData:replyHandler:)]

#### Reachability

- Both of your apps need to be reachable to send messages
- Check the `WCSession.isReachable` property to determine reachability
- Both devices/app must be in Bluetooth or Wi-Fi range (to be reachable)
- The WatchKit Extension must be running in the foreground or in the background (high priority only)
- The iOS counterpart does not have such requirements - If you send a message from your Watch app to your iOS app, and your iOS app is not in the foreground, your iOS app will be activated in the background to receive the message

### URLSession

- background and foreground configuration sessions available:
  - use background sessions as much as possible
  - foreground sessions need to complete while your app is in the foreground or front-most

#### Background URL Session

Use background URL session:

- any time communication can be delayed
- for all large data transfers
- when updates are initiated by a server (via push notifications)

Example on how to send data to a server in the background:

```swift
class BackgroundURLSession: NSObject, ObservableObject, Identifiable {
  private let sessionIDPrefix = "com.example.backgroundURLSessionID."
  
  enum Status {
    case notStarted
    case queued
    case inProgress(Double)
    case completed
    case failed(Error)
  }
  
  private var url: URL

  /// Data to send with the URL request.
  ///
  /// If this is set, the HTTP method for the request will be POST
  var body: Data?
  
  /// Optional content type for the URL request
  var contentType: String?
  
  private(set) var id = UUID()
  
  /// The current status of the session
  @Published var status = Status.notStarted
  
  /// The downloaded data (populated when status == .completed)
  @Published var downloadedURL: URL?
  
  private var backgroundTasks = [WKURLSessionRefreshBackgroundTask]()
  
  private lazy var urlSession: URLSession = {
    let config = URLSessionConfiguration.background(withIdentifier: sessionID)
    // Set isDiscretionary = true if you are sending or receiving large 
    // amounts of data. Let Watch users know that their transfers might 
    // not start until they are connected to Wi-Fi and power.
    config.isDiscretionary = false
    config.sessionSendsLaunchEvents = true
    return URLSession(configuration: config, delegate: self, delegateQueue: nil)
  }()
  
  private var sessionID: String {
    "\(sessionIDPrefix)\(id.uuidString)"
  }
  
  /// Initialize the session
  /// - Parameter url: The URL for the Background URL Request
  init(url: URL) {
    self.url = url
    super.init()
  }

  // Enqueue the URLRequest to send in the background. 
  func enqueueTransfer() {
    var request = URLRequest(url: url)
    request.httpBody = body
    if body != nil {
      request.httpMethod = "POST"
    }
    if let contentType = contentType {
      request.setValue(contentType, forHTTPHeaderField: "Content-type")
    }
    let task = urlSession.downloadTask(with: request)

    // Note that the system will determine the actual time our task starts 
    // based on background budget, network, and system conditions. Your 
    // app can receive up to four background refresh tasks per hour, if 
    // you have a complication on the active watch face, so schedule your 
    // tasks at least 15 minutes apart to prevent them from being delayed 
    // by the system.
    task.earliestBeginDate = nextTaskStartDate

    BackgroundURLSessions.sharedInstance().sessions[sessionID] = self

    task.resume()
    status = .queued
  }

  // Add the Background Refresh Task to the list so it can be set to 
  // completed when the URL task is done.
  func addBackgroundRefreshTask(_ task: WKURLSessionRefreshBackgroundTask) {
    backgroundTasks.append(task)
  }
}

extension BackgroundURLSession: URLSessionDownloadDelegate {
  private func saveDownloadedData(_ downloadedURL: URL) {
    // Move or quickly process this file before you return from this function.
    // The file is in a temporary location and will be deleted.
  }
  
  func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL) {
    saveDownloadedData(location)
  
    // We don't need more updates on this session, so let it go.
    BackgroundURLSessions.sharedInstance().sessions[sessionID] = nil
  
    DispatchQueue.main.async {
      self.status = .completed
    }
    
    for task in backgroundTasks {
      task.setTaskCompletedWithSnapshot(false)
    }
  }
}
```

The system will notify our app when our background request has been processed using a background task sent to our Extension Delegate:

```swift
class ExtensionDelegate: NSObject, WKExtensionDelegate {
  
  func applicationDidFinishLaunching() {
    // For Watch Connectivity, activate your WCSession as early as possible
    WatchConnectivityModel.shared.activateSession()
  }
  
  func applicationDidBecomeActive() {
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
  }
  
  func applicationWillResignActive() {
    // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
    // Use this method to pause ongoing tasks, disable timers, etc.
  }
  
  func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
    // Sent when the system needs to launch the application in the background to process tasks. Tasks arrive in a set, so loop through and process each one.
    for task in backgroundTasks {
      // Use a switch statement to check the task type
      switch task {
        case let backgroundTask as WKApplicationRefreshBackgroundTask:
          // Be sure to complete the background task once you’re done.
          backgroundTask.setTaskCompletedWithSnapshot(false)
        case let snapshotTask as WKSnapshotRefreshBackgroundTask:
          // Snapshot tasks have a unique completion call, make sure to set your expiration date
          snapshotTask.setTaskCompleted(restoredDefaultState: true, estimatedSnapshotExpiration: Date.distantFuture, userInfo: nil)
        case let connectivityTask as WKWatchConnectivityRefreshBackgroundTask:
          // Be sure to complete the connectivity task once you’re done.
          connectivityTask.setTaskCompletedWithSnapshot(false)
        case let urlSessionTask as WKURLSessionRefreshBackgroundTask:
          if let session = BackgroundURLSessions.sharedInstance()
                  .sessions[urlSessionTask.sessionIdentifier] {
              session.addBackgroundRefreshTask(urlSessionTask)
          } else {
              // There is no model for this session, just set it complete
              urlSessionTask.setTaskCompletedWithSnapshot(false)
          }
        case let relevantShortcutTask as WKRelevantShortcutRefreshBackgroundTask:
          // Be sure to complete the relevant-shortcut task once you're done.
          relevantShortcutTask.setTaskCompletedWithSnapshot(false)
        case let intentDidRunTask as WKIntentDidRunRefreshBackgroundTask:
          // Be sure to complete the intent-did-run task once you're done.
          intentDidRunTask.setTaskCompletedWithSnapshot(false)
        default:
          // make sure to complete unhandled task types
          task.setTaskCompletedWithSnapshot(false)
      }
    }
  }
}
```

Here's how we connect this delegate class in a watch app:

```swift
@main
struct MyWatchApp: App {
  @WKExtensionDelegateAdaptor(ExtensionDelegate.self) var extensionDelegate // 👈🏻
  
  @SceneBuilder var body: some Scene {
    ...
  }
}
```

#### Foreground URL Session

Use foreground URL session:

- for quick server communication
- when you require immediate data during app interaction

Foreground URL sessions have a <kbd>2.5</kbd> minute timeout, but you should limit foreground sessions to interactions that are much quicker than that.

### Sockets

- If you're building a streaming audio app, sockets are another option to communicate directly with servers
- You can use:
  - HTTP Live Streaming (HLS)
  - Web Sockets

## Data communication tools Cheat sheet 

|  | Source, Destination | Relies on companion iPhone | Supports Family Setup | Best For |
| --- | --- | --- | --- | --- |
| iCloud Keychain synchronization | All devices | No | Yes | Infrequently changing data |
| Core Data with CloudKit | All devices and iCloud | No | Yes | Structured data |
| Watch Connectivity | Paired iPhone and Watch | Yes | No | Optimization |
| URL Sessions | Server | No | Yes | Most server communication |
| Sockets | Server | No | Yes | Streaming audio |

[ps1]: ps1.png
[ks1]: ks1.png
[textcontenttype]: https://developer.apple.com/documentation/swiftui/geometryreader/textcontenttype(_:)-ki0h
[updateApplicationContext(_:)]: https://developer.apple.com/documentation/watchconnectivity/wcsession/1615621-updateapplicationcontext
[outstandingUserInfoTransfers]: https://developer.apple.com/documentation/watchconnectivity/wcsession/1615627-outstandinguserinfotransfers
[outstandingFileTransfers]: https://developer.apple.com/documentation/watchconnectivity/wcsession/1615641-outstandingfiletransfers
[session(_:didReceive:)]: https://developer.apple.com/documentation/watchconnectivity/wcsessiondelegate/1615651-session
[remainingComplicationUserInfoTransfers]: https://developer.apple.com/documentation/watchconnectivity/wcsession/1771700-remainingcomplicationuserinfotra
[sendMessage(_:replyHandler:errorHandler:)]: https://developer.apple.com/documentation/watchconnectivity/wcsession/1615687-sendmessage
[session(_:didReceiveMessage:replyHandler:)]: https://developer.apple.com/documentation/watchconnectivity/wcsessiondelegate/1615677-session
[session(_:didReceiveMessageData:replyHandler:)]: https://developer.apple.com/documentation/watchconnectivity/wcsessiondelegate/1615653-session