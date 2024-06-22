# Why is my app getting killed?

Put on your detective’s hat: It’s time to track down those unruly app terminations. We’ll outline the six major reasons apps terminate in the background, and show you how you can use MetricKit to to help you identify key statistics to drive down the rate of terminations. Learn how to prevent problems and recover gracefully from inevitable jetsams, identify any underlying issues, and take actionable measures to fix them. And discover the importance of implementing state restoration to make terminations less jarring — especially where text entry or playback is concerned.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10078", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(abadikaka)
   }
}



Top causes of app termination:

* Crashes
* CPU Resources limit
* Watchdog
* Memory limit exceeded
* Memory pressure exit
* Background task timeout

## New MetricKit API

`MXBackgroundExitData` helps us understand why our app is getting killed by providing the exit count every time app is getting terminated:

```swift
- cumulativeBadAccessExitCount
- cumulativeIllegalInstructionExitCount
- cumulativeAbnormalExitCount
- cumulativeCPUResourceLimitExitCount
- cumulativeMemoryResourceLimitExitCount
- cumulativeMemoryPressureExitCount
- cumulativeSuspendedwithLockedFileExitCount
- cumulativeAppWatchdogExitCount
- cumulativeBackgroundTaskAssertionTimeoutExitCount
- cumulativeNormalExitCount 
```

### Crashes

Crash is the most straightforward type of terminations. Crash may happen because of these 3 reasons:

1. Segmentation fault
2. Illegal Instruction
3. Asserts and uncaught exceptions

These event will be generated on crashlog and the crash will report to us automatically. In addition of the Xcode organizer, MetricKit add more APIs for diagnostic on a per device basis which is `MXCrashDiagnostic`

`MXCrashDiagnostic` will provide:

* StackTrace
* Signal
* Exception Code
* Termination Reason

### Watchdog

Another termination happen because of Watchdog event that happen because of timeout during some key transitions.

* Timeout during key transitions: long hang during app key transition such as launch, going background or foreground again. It has time limit around 20 seconds during transition.
* Disabled in Simulator and in the debugger
* Fixing watchdog event will help to eliminate deadlocks, infinite loop, and unending synchronous work on the main thread
* Report available in `MXCrashDiagnostic`

### CPU Resource limit

CPU Resource limit is indicating the high sustained CPU load in background
Addition in Xcode 12 there will be solution:

* Energy Exception Report through Xcode organizer and `MXCPUExceptionDiagnostic`
* Call stack point out hotspots in code
* Consider moving work into `BGProcessingTask`

### Memory footprint exceeded

App using too much memory. Some solutions:

* Same limit for foreground and background
* Use instruments and memory debugger
* Keep in mind limitation for older devices

### Jetsam (Memory pressure exit)

> NOTE: This is not a bug in our app, and it is most common termination.

* System freeing up memory for active applications

How to reduce jetsam rate?

* Aim for less than 50 MB  in the background

Upon backgrounding:

* Flush state to disk
* Clear out image view
* Drop caches

There are also suggestion for recovering from jetsam:

* Save state upon entering background such as view controller stack, draft input in text fields, media playback position and many more depend on your use case
* Adopt UIKit State Restoration
* Remember users should not realize that the app was terminated

### Background task timeout

We can use `beginBackgroundTask` and `endBackgroundTask` for executing background task when going to background (Remember we have 30 seconds before app is suspended)

```swift
UIApplication.beginBackgroundTask(expirationHandler:)
UIApplication.endBackgroundTask(_:)
```

* The problem is when we don't call `endBackgroundTask`, it will lead to failure to end task explicitly result in termination. However it is preventable
* Counts exposed via `MXBackgroundExitData`

We should use the name  variant of UIKit API to prevent the termination happen

```swift
UIApplication.beginBackgroundTask(withName:expirationHandler:)
```

Why?

* The terminations do not occur in debugger
* After that we can do console message and do an audit of our call to matching background and end task process

Another solution is using `expirationHandler`

* Implement an `expirationHandler` as safety net and do not rely it exclusively
* Call `endBackgroundTask` inside handler
* Do not begin new work inside handler
* Add telemetry at the start and each of each expiration handler

```swift
let handle = MXMetricManager.makeLogHandle(category: "DatabaseExpHandler")
mxSignpost(.event, log: handle, name: "Entered")
cancelOperations()
closeDatabase()
mxSignpost(.event, log: handle, name: "Exited")

UIApplication.shared.endBackgroundTask(backgroundTaskIdentifier)
```

Now let us inspect `MXMetricPayload` to see the signpost count and check any imbalance of the signpost count

```xml
"signpostMetrics": [ 
  {
    "signpostCategory": "DatabaseExpirationHandler",
    "signpostName": "Entered",
    "totalSignpostCount": 2
  },
  {
    "signpostCategory": "DatabaseExpirationHandler", 
    "signpostName": "Exited",
    "totalSignpostCount": 1 
  }
]
```

Another solutions to improve debugging of app terminations is checking `backgroundTimeRemaining` before doing some background work
* Only start work if plenty of time remains
* Unsafe to begin task with < 5 seconds remaining

Example code:

```swift
let minimumTimeRemaining = min(5, estimateProcessingTime(inputData))

if UIApplication.shared.backgroundTimeRemaining > minimumTimeRemaining {
    // Enough time remains, call begin background task
    return UIApplication.shared.beginBackgroundTask { ... }
} else {
    // Not enough time remains, defer this work until later
    registerProcessingTask(inputData)
    return .invalid
}
```

The next thing we need to do also is avoiding leaking `UIBackgroundTaskIdentifier`. Use local variable instead of an instance variable to hold `UIBackgroundTaskIdentifier` and it will preventing the leak since it will allocated on different memory

Example code:

```swift
@IBAction func beginDataExport(sender: UIButton) {
    var taskId: UIBackgroundTaskIdentifier = .invalid
    taskId = UIApplication.shared.beginBackgroundTask {...}
    // End the background task after archiving, which takes several seconds
    ArchiveUtility.exportUserData(completion: ()->()) {
        UIApplication.shared.endBackgroundTask(taskId)
    }
}
```

Finally we learn about some solutions to reduce termination:

* Identify and fix termination
* Reduce memory usage
* Implement state restoration