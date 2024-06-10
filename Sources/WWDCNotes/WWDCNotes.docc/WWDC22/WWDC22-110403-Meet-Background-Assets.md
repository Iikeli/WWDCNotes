# Meet Background Assets

Discover how you can use the Background Assets framework to download large files directly from your CDN and improve the initial launch experience of your apps and games. We’ll show you how to schedule background downloads during initial app install, app updates, and periodically as someone uses the app. We’ll also explore how you can manage scheduled downloads to make sure people have the content they want, when they want it.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110403", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Background Assets

- new framework called [Background Assets][ba]
- aims to:
  - reduce app download size from App Store
  - avoid having launch screen where apps need to download even more assets (make sure that assets are present before first app launch, or whenever the app is updated overnight)

- supports out-of-band content (able to push updated content to your apps without submission to the App Store)

- achieved via a new app extension for downloading content in the background
  - execution outside of an app's lifecycle
  - extension runtime is short-lived, schedule your downloads quickly
  - extension will execute:
    - after app installation (via AppStore or Test Flight)
    - after an app update
    - periodically in the background (based on app usage)

## Adopting the framework

### Download manager

- [`BADownloadManager`][BADownloadManager]
- primary way to communicate with the Background Assets system service (for managing assets)
- singleton object
- can be used throughout your app
- use it to: 
  - schedule downloads (in either foreground or background)
  - retrieve current download operations
  - cancel downloads
  - synchronize between app and app extension

#### Start downloading assets

```swift
// Getting started with Background Assets
import BackgroundAssets

let url = URL(string: "https://cdn.example.com/large-asset.bin")!

// 👇🏻 Use this app group container to manage your assets between app and the extension
let appGroupIdentifier = "group.WWDC.AssetContainer"

// 👇🏻 This is your download object, BackgroundAssets offers different objects.
//    This object tells the system both what we're downloading and where the 
//    resulting file will end up.
let download = BAURLDownload(
  identifier: "Large-Asset", // 👈🏻 Use this id to track the asset download across app/extensions launches
  request: URLRequest(url: url),
  applicationGroupIdentifier:
  appGroupIdentifier
)

// 👇🏻 Schedule the download via BADownloadManager
let manager = BADownloadManager.shared
manager.delegate = self // BADownloadManagerDelegate protocol
//       👆🏻 gets notified as soon as download objects are processed by the system

// Schedule download at an opportunistic time determined by the system
do {
  try manager.schedule(download)
} catch {
  print("Failed to schedule download. \(error)")
}

// or Schedule download in foreground
// This allows download to begin immediately (possible only in the app, not in the extension)
do {
  try manager.startForegroundDownload(download)
} catch {
  print("Failed to start foreground download. \(error)")
}

// or Promote downloads to foreground.
do {
  for download in try await manager.fetchCurrentDownloads) {
     try manager.startForegroundDownload(download)
  }
} catch {
  print("Failed to promote downloads to foreground \(error)")
}
```

- [`BADownloadManagerDelegate`][BADownloadManagerDelegate]
  - receives callbacks for all downloads that have been scheduled by either the extension or your app
  - if your app does not handle one of the delegate methods or your delegate is not set, then your extension will wake to process the callback
  - the extension (if the app doesn't handle such methods) is woken only for callbacks that share common interfaces between `BADownloadManagerDelegate` and the [`BADownloaderExtension`][BADownloaderExtension] protocol

```swift
public protocol BADownloadManagerDelegate : NSObjectProtocol {

  /// triggered whenever a download starts
  optional func downloadDidBegin(_ download: BADownload)

  /// triggered whenever a download pauses
  optional func downloadDidPause(_ download: BADownload)

  /// lets you monitor active progress of your download as it is being downloaded in the foreground
  optional func download(
    _ download: BADownload, 
    bytesWritten: Int64, 
    totalBytesWritten: Int64, 
    totalExpectedBytes: Int64
  )

  /// lets you answer a challenge request. Useful for validating the authenticity of a connection 
  /// or for providing credentials to authorize a connection.
  optional func download(
    _ download: BADownload, 
    didReceive challenge: URLAuthenticationChallenge
  ) async -> (URLSession.AuthChallengeDisposition, URLCredential?)

  /// triggered whenever a download fails
  ///
  /// You might want to re-schedule the download, or figure out the failure cause
  optional func download(_ download: BADownload, failedWithError error: Error)

  /// triggered whenever a download completes
  ///
  /// Apple recommends to leave the file at the given URL, so that it can be managed by the 
  /// system (a.k.a. the system will delete the file "for you" if the device is low on space)
  optional func download(_ download: BADownload, finishedWithFileURL fileURL: URL)
}
```

## Extension overview

### Requirements

Required app <kbd>Info.plist</kbd> configurations (note: only the app <kbd>Info.plist</kbd> needs this, not the extension):

1. set a [<kbd>BAInitialDownloadRestrictions</kbd>][bainitialdownloadrestrictions] dictionary, which declares two restrictions for your extension:
  - [<kbd>BADownloadAllowance</kbd>][BADownloadAllowance] (Integer) - declare the combined size (in Bytes) of the initial set of asset downloads
  - [<kbd>BADownloadDomainAllowList</kbd>][BADownloadDomainAllowList] (Array of strings) - list of web domains the extension can use when scheduling the initial set of asset downloads (prefix wildcards are permitted, e.g., `*.cdn.example.com`)

2. set a [<kbd>BAMaxInstallSize</kbd>][BAMaxInstallSize] Number, representing the maximum size that your app will require in additional storage for these assets (when uncompressed)

- <kbd>BAInitialDownloadRestrictions</kbd> are reviewed by App Review, try to be as accurate as possible
- <kbd>BAInitialDownloadRestrictions</kbd> are only enforced after first app install, whenever your app is launched, these restrictions are no longer enforced
- <kbd>BAMaxInstallSize</kbd> is shown in the App Store

### Extension entry points

- the functions that you define from the [`BADownloaderExtension`][BADownloaderExtension] protocol will be called by the system and not by your app
- keep work at a minimum: the extension is for quick scheduling only, do not do anything like assets decompression or similar
- the extension has access to [`BADownloadManager`][BADownloadManager]
- use an app group container to manage your assets between app and the extension

```swift
public protocol BADownloaderExtension: NSObjectProtocol {
  // Triggered when the app is first installed - the app has not launched yet, but your extension has.
  // Use this callback to download required assets not present in your app binary
  // (remember: this is where info.plist download restrictions above are applied)
  optional func applicationDidInstall(metadata: BAApplicationExtensionInfo)

  // Triggered when the app gets updated (a new version has been published on the store
  // and it has been downloaded into the device).
  // As long as the user hasn't quit your app in the app switcher, this will trigger.
  optional func applicationDidUpdate(metadata: BAApplicationExtensionInfo)

  /// Periodically awoken by the system, use this to check for any updates that need to be background downloaded
  optional func checkForUpdates(metadata: BAApplicationExtensionInfo)

  /// Like BADownloadManagerDelegate
  optional func download(_ download: BADownload, didReceive challenge: URLAuthenticationChallenge) async -> (URLSession.AuthChallengeDisposition, URLCredential?)

  /// Like BADownloadManagerDelegate
  optional func backgroundDownloadDidFail(failedDownload: BADownload)

  /// Like BADownloadManagerDelegate, the error is stored within the BADownload object
  optional func backgroundDownloadDidFinish(finishedDownload: BADownload, fileURL: URL)
  
  optional func extensionWillTerminate()
}
```

### Synchronizing app and extension

- If both app and extensions are running, and both have set their own `BADownloadManager.shared.delegate`, both will received events regarding download finished and similar callbacks
- to avoid data races (e.g., where both app and extension would try to react on a download completed event), use `BADownloadManager`'s [`withExclusiveControl(_:)`][withExclusiveControl(_:)] or [`withExclusiveControl(beforeDate:_:)`][withExclusiveControl(beforeDate:_:)]
  - these methods accept a closure which guarantees to be mutually exclusive with other calls that require exclusive control
  - note that acquiring exclusive control can fail, check for the `error` parameter passed to your closure

```swift
// Synchronizing between app and extension
func download(_ download: BADownload, finishedWithFileURL fileURL: URL) {
  let manager = BADownloadManager.shared

  manager.withExclusiveControl { error in
    guard error == nil else {
      print("Unable to acquire exclusive control \(String(describing: error))")
      return
    }
    // Exclusive control acquired
    // All code in this scope ensures mutual exclusion between extension and app
    do {
      let data = try Data(contentsOf: fileURL, options: .mappedIfSafe)
      // Do something with memory mapped data
      try FileManager.default.removeItem(at: fileURL)
    } catch {
      print("Unable to read/cleanup file data. \(error)")
    }
  }
}
```

[ba]: https://developer.apple.com/documentation/backgroundassets
[BADownloadManager]: https://developer.apple.com/documentation/backgroundassets/badownloadmanager
[BADownloadManagerDelegate]: https://developer.apple.com/documentation/backgroundassets/badownloadmanagerdelegate
[BADownloaderExtension]: https://developer.apple.com/documentation/backgroundassets/badownloaderextension
[bainitialdownloadrestrictions]: https://developer.apple.com/documentation/bundleresources/information_property_list/bainitialdownloadrestrictions
[BADownloadAllowance]: https://developer.apple.com/documentation/bundleresources/information_property_list/bainitialdownloadrestrictions/badownloadallowance
[BADownloadDomainAllowList]: https://developer.apple.com/documentation/bundleresources/information_property_list/bainitialdownloadrestrictions/badownloaddomainallowlist
[BAMaxInstallSize]: https://developer.apple.com/documentation/bundleresources/information_property_list/bamaxinstallsize

[withExclusiveControl(_:)]: https://developer.apple.com/documentation/backgroundassets/badownloadmanager/4027481-withexclusivecontrol
[withExclusiveControl(beforeDate:_:)]: https://developer.apple.com/documentation/backgroundassets/badownloadmanager/4027482-withexclusivecontrol