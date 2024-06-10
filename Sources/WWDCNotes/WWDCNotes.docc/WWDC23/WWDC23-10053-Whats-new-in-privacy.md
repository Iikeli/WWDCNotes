# What’s new in privacy

At Apple, we believe that privacy is a fundamental human right. Learn about new technologies on Apple platforms that make it easier for you to implement essential privacy patterns that build customer trust in your app. Discover privacy improvements for Apple’s platforms, as well as a study of how privacy shaped the software architecture and design for the input model on visionOS.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10053", purpose: link, label: "Watch Video (32 min)")

   @Contributors {
      @GitHubUser(NinjaLikesCheez)
   }
}



Apple breaks privacy down into 4 pillars:

* Data Minimization
* On-Device Processing
* Transparency & Control
* Security Protections

## New Tools

New APIs have been added:

* Embedded Photos picker
* Screen Capture picker
* Write-only calendar access
* Oblivious HTTP
* Communication Safety

### Embedded Photos Picker

* This new API gives your app access to selected photos or videos without requiring permission to access the entire photo library
* Can be embedded entirely into your app (iOS 17 & macOS Sonoma)
  * These are rended by the system, and only shared with the app when selected
* This includes UI customization to allow you to style the photo picker
* No permissions required

For more, watch [Embed the Photos picker in your app - WWDC23](https://developer.apple.com/wwdc23/10107))

In addition, the photos permission dialog has been redesigned to remind users what they're giving access too, and periodically asks them if they'd like to change the permission level.

### Screen Capture picker

New API in macOS ScreenCaptureKit that enables people to share only the window or screen your app needs.

* Previously, a user had to allow full screen recording access to an app
  * Now, users can choose portions of their windows or entire screens to record
* Because this is an explicit user action, no permissions are required
* New screen sharing menu bar item alerts users that a screen recording session is happening
* This includes UI customization for the picker

For more details: [What's new in ScreenCaptureKit - WWDC23](https://developer.apple.com/wwdc23/10136)

### Write-only calendar access

* No permission required to create new events using the EventKitUI
  * These are rendered outside of your app
* New Write-only permission if you want to use a custom UI
* You can ask for a permission upgrade from write-only to full
  * This has a new UI which (like Photos picker) shows the user what information an app would have access to
* Importantly, the new default permission is write-only
  * If you previously had the Calendar permission, it will default to write-only on iOS 17/macOS Sonoma
* If linking to an older version of EventKit:
  * Prompt defaults to write-only
  * Implicit upgrade when app fetches events

For more details: [Discover Calendar and EventKit - WWDC23](https://developer.apple.com/wwdc23/10052)

### Oblivious HTTP

New API which helps you to hide client IP addresses from your server and usage patterns from network operators.

* Cellular and Wi-Fi operators can observe what server someone connects to
  * IP Address can be abused to track users
* Apple Platforms now support OHTTP which allows you to relay users (encrypted) traffic
  * This results in network operators only seeing the user connect to the relay provider
  * Relay knows the user IP address & destination server, but not the content
  * This means no party sees both the client IP & content

There are some additional considerations:

* Replace IP addresses with alternatives
* Private Access Tokens
* Encrypt DNS

For more details:

* [Replace CAPTCHAs with Private Access Tokens - WWDC22](https://developer.apple.com/wwdc22/10077)
* [Enable encrypted DNS - WWDC20](https://developer.apple.com/wwdc20/10047)

### Communication Safety

Communication Safety helps keep children safe by warning children and providing resources if they receive or attempt to share images that likely contain nudity.

This is expanded to include:

* Sharing via AirDrop
* Leaving a message on FaceTime
* Contact posters in the phone app
* Selecting photos in the Photos picker
* Older users can also enable these features to be shown warnings

Sensitive Content Analysis Framework allows your app to enable communication safety:

* Detect sensitive content
* On-device processing
* System provided ML models

To add it to your app:

```swift
// Analyzing photos
let analyzer = SCSensitivityAnalyzer ()
let policy = analyzer.analysisPolicy

let result = try await analyzer. analyzeImage(at: url)
let result = try await analyzer.analyzeImage (image.cgImage!)

if result.isSensitive {
	intervene(policy)
}
```

If the result is sensitive, you application should intervene:

* Obfuscate content
* Option to view the content
* Adjust intervention based on enabled feature

For more details: [Detecting nudity in media and providing intervention options - Documentation](https://developer.apple.com/documentation/sensitivecontentanalysis/detecting_nudity_in_media_and_providing_intervention_options)

## Platform Changes

* Mac app data protection
  * Sonoma provides additional control over what apps can access other apps data
  * Sandboxed apps get this protection automatically
  * Non-sandboxed apps should adopt the App Sandbox
  * Permission is asked when you attempt to access the data and is valid while the app is open
    * Permission is reset when the app is closed
  * Explicit permission via `NSOpenPanel` don't require a permission prompt
  * Apps with Full Disk Access won't require a prompt
  * You can access another apps data, if it's signed with the same Team ID
    * You can define a more restrictive policy in your Info.plist using the `NSDataAccessSecurityPolicy` key
* Advanced Data Protection
  * Added in 2022, it allowed users to enable E2E encryption for the vast majority of iCloud data
  * By adopting CloudKit you can E2E data in iCloud when someone enables Advanced Data Protection
    * This is seamless, and you don't need to handle key management or recovery situations
    * To enable:
      * Use encrypted data types
        * `CKAsset`
        * Encrypted variants: `String` -> `EncryptedString`
    * For details: [What's new in CloudKit  - WWDC21](https://developer.apple.com/wwdc21/10086)
* Safari Private Browsing
  * Adds Advanced Tracking and Fingerprinted Protection
    * Web Developers should test in Private Browsing mode
    * Web Inspector will show blocked connections to known trackers
    * When browsing or copying a link, Safari will remove tracking parameters
* Safari app extensions
  * New permission model for app extensions
    * Per-site permissions
    * User control whether extension can run in Private Browsing mode
    * For details: [What's new in Safari extensions - WWDC23](https://developer.apple.com/wwdc23/10119)

## Spatial Input Model

High level goals the interface of the model needs to achieve:

* Fast and natural interaction
* User know where they tap
* iOS and iPadOS apps work seamlessly
* Apps do not need new permissions

Privacy goals for the model:

* Very limited access to eye cameras
* Apps work without eye and hand data
* Apps do not know what users look at

Hand and eye data:

* Hand & eye data is processed by an isolated system process
  * This is handed to other systems to determine positions of hands and eyes
* Hover feedback is done by highlighting UI components
  * This is done as the app is rendered
  * App has no means of seeing what's been highlighted
* When a Pinch is detected, the system delivers a normal tap event to the UI element

Customizable hover effects:

* Type
* Shape
* Affected elements

For more details:

* [Meet SwiftUI for spatial computing - WWDC23](https://developer.apple.com/wwdc23/10109)
* [Elevate your windowed app for spatial computing - WWDC23](https://developer.apple.com/wwdc23/10110)
