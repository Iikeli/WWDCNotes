# Meet Focus filters

Discover how you can customize app behaviors based on someone's currently enabled Focus. We'll show you how to use App Intents to define your app's Focus filters, act on changes from the system, and present your app's views in different ways. We'll also explore how you can filter notifications and update badge counts.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10121", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Focus filter

- new way for users to customize app behavior based on the currently enabled Focus

Examples:

- Calendar.app can show different calendars based if it's weekend (Weekend focus) or if it's work time (Work focus)
- Mail.app can show different mailboxes and notifications based on the active focus

When to adopt focus filters

- when your app manages multiple accounts (e.g., for leisure and work)
- when your app has lots of content that can be filtered
- when your app should be less distracting (e.g., by turning of badges) for certain focus modes

How Focus filter work:

1. Your app defines what can be customized via AppIntents
2. The system exposes to the user what can be configured as a Focus filter
3. The user will configure Focus filter settings in your app

## Defining a Focus filter in your app

1. implement [`SetFocusFilterIntent`][setfocusfilterintent] - this tells the system that your app is interested in having custom settings per Focus

```swift
import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
  /// Title and description help users discover what your Focus is about.
  ///
  /// Title and description are static and are ready by the system when 
  /// your app is installed.
  static var title: LocalizedStringResource = "Set account, status & look"
  static var description: LocalizedStringResource? = "..."
}
```

2. define your app parameters - these will represent what can be configured within your app by the user
  - each parameter must have a name and a data type (`Bool`, `String`, etc.)
  - custom data types are supported via App Intents entities ([`AppEntity`][appentity])
  - parameter can be marked optionals, meaning that they do not have to be configured
  - non-optional parameters should provide default values

```swift
import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
  // ...

  @Parameter(title: "Use Dark Mode", default: false)
  var alwaysUseDarkMode: Bool

  @Parameter(title: "Status Message")
  var status: String?

  @Parameter(title: "Selected Account")
  var account: AccountEntity?

  // ...
}
```

3. set display representation - making your Focus filter appears in system settings with the correct content

```swift
struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
  // ...

  /// This should return a dynamic representation of the current user configuration to 
  /// be displayed in the Settings.app.
  var displayRepresentation: DisplayRepresentation {
    ...

    return DisplayRepresentation(title: title, subtitle: subtitle)
  }
  
  // ...
}
```

## Acting on a Focus filter

The system will deliver relevant focus updates to your app in two ways:

1. If the app is running, you will receive a call to the perform method in your `FocusFilterIntent` (if you've implemented it)
2. If the app is not running, you can implement an extension that will be triggered when needed
  - at the end, [`perform()`][p] in your FocusFilterIntent will get called (in your extension)
  - an AppIntent extension is not necessary if you only need to act upon the changes in your main app
  - however, if you need to control widgets, notifications and/or badges, you should implement the extension

```swift
struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
  // ...

  func perform() async throws -> some IntentResult {
    let myData = AppData(
      alwaysUseDarkMode: self.alwaysUseDarkMode, // 👈🏻 you access to the user configuration
      status: self.status,                       // by accessing the @Parameter properties
      account: self.account                      // values
    )
    myModel.shared.updateAppWithData(myData)
    return .result()
  }
}
```

[setfocusfilterintent]: https://developer.apple.com/documentation/appintents/setfocusfilterintent
[appentity]: https://developer.apple.com/documentation/appintents/appentity
[p]: https://developer.apple.com/documentation/appintents/openentityintent/perform()
