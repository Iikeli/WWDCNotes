# Testing Tips & Tricks

Testing is an essential tool to consistently verify your code works correctly, but often your code has dependencies that are out of your control. Discover techniques for making hard-to-test code testable on Apple platforms using XCTest. Learn a variety of tips for writing higher-quality tests that run fast and require less maintenance.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/417", purpose: link, label: "Watch Video (37 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- The usual reminder about the “Pyramid of Tests”, three kinds of tests (from small to big, fast to slow):
  - Unit: test specific functions/methods
  - Integration: test classes/methods interaction
  - End-To-End (System Tests, UI): test the user final experience

- `NotificationCenter` can use different environments (kind of like `UserDefaults` have different groups): you should avoid using the default notification center in your tests as this may affect other parts of the code (the test is therefore NOT isolated). To make things testable you should inject the `notificationCenter` in all your methods, going from this: 

```swift
class Points0fInterestTableViewController { 
  var observer: AnyObject? 

  init() { 
    let name = CurrentLocationProvider.authChangedNotification
    observer = NotificationCenter.default.addObserver(
      forName: name, 
      object: nil, 
      queue: .main
    ) { [weak self] _ in 
      self?.handleAuthChanged()
    }
  }

  var didHandleNotification = false 

  func handleAuthChanged() { 
    didHandleNotification = true 
  }
}
```

To this:

```swift
class Points0fInterestTableViewController { 
  let notificationCenter: NotificationCenter
  var observer: AnyObject? 
  
  init(notificationCenter: NotificationCenter) {
    self.notificationCenter = notificationCenter
    let name = CurrentLocationProvider.authChangedNotification
    observer = notificationCenter.addObserver(
      forName:name, 
      object: nil, 
      queue: .main
    ) { [weak self] _ in 
      self?.handleAuthChanged() 
    }
  }

  var didHandleNotification = false 

  func handleAuthChanged() { 
    didHandleNotification = true 
  }
}
```

- You can also add a default value so you need to change the parameter only in the tests:

```swift
class PointsofInterestTableViewController {
  let notificationCenter: NotificationCenter 
  var observer: AnyObject? 

  init(notificationCenter: NotificationCenter = .default) { 
    self.notificationCenter = notificationCenter
    let name = CurrentLocationProvider.authChangedNotification
    observer = notificationCenter.addObserver(
      forName: name,
      object: nil,
      queue: .main
    ) { [weak self] _ in
      self?.handleAuthChanged() 
    }
  }

  var didHandleNotification = false 

  func handleAuthChanged() { 
    didHandleNotification = true 
  }
}
```

- When creating mocks, beside relying as much as possible on protocols, another neat thing your mock classes/struct should have is the possibility to overwrite some of their methods, so every test can change the behaviour of the mock accordingly on what you’re trying to test

- To make tests run even faster, you can try to skip some setups in your app launch by adding an environment variable or a launch argument in your schema:

![][setup1Image]

And then use it in your app:

```swift
func application (_ application: UIApplication, didFinishLaunchingWithOptions opts: ...) -> Bool {
  let isUnitTesting = ProcessInfo.processInfo.environment["IS_UNIT_TESTING"] == "1"

  if isUnitTesting == false { 
    // Do UI-related setup, which can be skipped when testing 
  }

  return true
}
```

[setup1Image]: WWDC18-417-setup1