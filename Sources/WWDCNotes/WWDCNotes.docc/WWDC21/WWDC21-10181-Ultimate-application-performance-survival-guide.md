# Ultimate application performance survival guide

Performance optimization can seem like a daunting task — with many metrics to track and tools to use. Fear not: Our survival guide to app performance is here to help you understand tooling, metrics, and paradigms that can help smooth your development process and contribute to a great experience for people using your app.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10181", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

Eight key metrics to track for application performance:

- Battery Usage
- Launch Time
- Hang Rate
- Memory
- Disk Writes
- Scrolling
- Terminations
- MXSignposts

Toolset:

- Xcode Organizer
- MetricKit
- Instruments
- XCTest
- App Store Connect API

## Metrics and resolutions

### Battery life

Subsystems to pay attention to improve battery life: 

- CPU
- Networking
- Location
- audio
- Bluetooth
- GPU

During development I can monitor battery impact via the energy gauge in the Debug navigator.

- `High CPU utilization` is when CPU use is greater than 20%
- `CPU Wake Overhead` is regions where the CPU wakes from an idle state, and there's an incurred energy cost

After shipping the app, we can monitor our app performance via MetricKit.

```swift
class AppMetrics: MXMetricManagerSubscriber {
  init() {
    let shared = MXMetricManager.shared
    shared.add(self)
  }

  deinit {
    let shared = MXMetricManager.shared
    shared.remove(self)
  }

  // Receive daily metrics
  func didReceive(_ payloads: [MXMetricPayload]) {
    // Process metrics
  }

  // Receive diagnostics
  func didReceive(_ payloads: [MXDiagnosticPayload]) {
    // Process metrics
  }
}
```

This data is also automatically collected (from devices that have given consent) and is available in the Xcode Organizer. 

New in Xcode 13, the Organizer has `Regression` pane:

- isolates all the metrics that have increased significantly in the most recent version of our app

Alternatively we can use the AppStore API.

## Hang Rate and Scrolling

- A hang is when the app is unresponsive to user input or actions for at least `250 milliseconds`
- Stuttering scrolls occur when new content isn't ready for the next screen refresh
- I can use Instruments to detect the cause of my hangs by using the `Thread State` or `System Call Traces`: 
  - the `Thread State Trace` instrument shows a timeline of the thread's state and when the OS has scheduled the thread to run
  - I can see how long a thread was blocked for in the details section
  - The `System Call Trace` shows a narrative that details the system calls entered and how long they took

- write performance tests with XCTest that launch and scrolls through the app

```swift
func testScrollingAnimationPerformance() throws {  
  app.launch()
  app.staticTexts["Meal Planner"].tap()
  let foodCollection = app.collectionViews.firstMatch

  let measureOptions = XCTMeasureOptions()
  measureOptions.invocationOptions = [.manuallyStop]
      
  measure(
    metrics: [XCTOSSignpostMetric.scrollDecelerationMetric],
    options: measureOptions
  ) {
    foodCollection.swipeUp(velocity: .fast)
    stopMeasuring()
    foodCollection.swipeDown(velocity: .fast)
  }
}
```

- New in iOS 15 and macOS 12, MetricKit will deliver all diagnostics, including hangs, in my app immediately after an issue occurs 
- In the case of scroll hitches, iOS 15 introduces a new API within MetricKit to tag custom animations using MXSignpost
- MXSignpost is a wrapper API shipped with MetricKit that allows to mark critical code sections for telemetry

```swift
func startAnimating() {
  // Mark the beginning of animations
  mxSignpostAnimationIntervalBegin(
    log: MXMetricManager.makeLogHandle(category: "animation_telemetry"), 
    name: "custom_animation”)
  }

  func animationDidComplete() {
  // Mark the end of the animation to receive the collected hitch rate telemetry
  mxSignpost(OSSignpostType.end, 
    log: MXMetricManager.makeLogHandle(category: "animation_telemetry"), 
    name: "custom_animation")
}
```

## Disk Writes

- profile your app using the File Activity template in Instruments
- this records file system use in the form of system calls, to easily identify places in the app's code where you're accessing the file system. 

Best practices:

- batching your write operations
- use Core Data for frequently-changing data
- avoiding rapid file creation and deletion
- write performance tests with XCTest to measure the disk usage

```swift
// Example performance XCTest

/// The test measures the amount of data written to disk by the code in the block and shows the result within Xcode itself
/// You can set a baseline of the amount of data expected to be written to disk so that the test fails if the code in the block exceeds that. 
func testSaveMeal() {
  let app = XCUIApplication()
  let options = XCTMeasureOptions()
  options.invocationOptions = [.manuallyStart]

  measure(metrics: [XCTStorageMetric(application: app)], options: options) {
    app.launch()
    startMeasuring()

    let firstCell = app.cells.firstMatch
    firstCell.buttons["Save meal"].firstMatch.tap()

    let savedButton = firstCell.buttons["Saved"].firstMatch
    XCTAssertTrue(savedButton.waitForExistence(timeout: 2))
  }
}
```

- We can look for the sources of these writes by taking a look at Xcode Organizer `Disk Writes` Reports
- these are collection of reports that are generated when your app writes more than 1 GB in a 24-hour period

## Launch time and Termination

- Launch time is the amount of time between when the user taps your app icon and when the first frame gets rendered in your app
- Process exits can happen for many different reasons, like hitting and exceeding the system memory limit or timing out on launch
- we can profile app launch time by using the `App Launch template` in Instruments
- this template runs the app for five seconds, during which it gathers a time profile and Thread State Trace of what was going on while the app was launching
- we can also measure launch times in a performance XCTest by using the `XCTApplicationsLaunchMetric` in a measure block. 

## Memory

- profile the memory use by using the Leaks, Allocations, and VM Tracker templates in Instruments
  - Leaks will examine my process's heap and check for leaked memory
  - Allocations will analyze the memory life cycle of my app
  - VM Tracker will show the virtual memory space of the app over time
