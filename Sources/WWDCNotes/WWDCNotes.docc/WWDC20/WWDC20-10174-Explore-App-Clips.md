# Explore App Clips

Help people experience the right parts of your app at the exact moment they need them. We’ll explain how to design and build an App Clip — a small part of your app that focuses on a specific task — and make it easily discoverable. Learn how to focus your App Clip on short and fast interactions and identify contextually-relevant situations where you can surface it, like a search in Maps or at a real-world location through QR codes, NFC, or App Clip codes. Find out a few key differences between apps and App Clips, and explore how App Clips interact with their corresponding apps.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10174", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



## Introduction to App Clips

1. App Clips are part of an app bundle
2. The App Clip experience starts from a URL, handled by the App Clip
3. App Clip is a separate binary shipped with the original app, an App Clip can be launched on its own

## App Clip Experiences

* Entry points to an App
* Invoked by URLs, registered in the App Store Connect
* Surfaced through user actions (QR codes, NFC tags, links in Safari and Messages, business details in Maps) and proactively as Siri suggestions

## App Clip

* It builds and runs as a separate application: it needs to handle the whole experience on its own
* It's a secondary target within the main application in Xcode
* It's submitted within the app bundle to App Store Review
* The main app and App Clip are downloaded separately and mutually exclusive on-device (users can only have one or the other)
* App clips must be less than 10MB after thinning
* If the app is launched through an App Clip Experience:
  - if the app is not installed, our App Clip will be installed and launched
  - otherwise, if the app is installed in the user device, the main app is launched

* App Clip should include only what's needed for the experience
* It should generally omit top level navigation elements (like tabbars), and should instead deeplink directly into the intended experience
* App Clips can do many things in different contexts and each with a different link, ideally each link should do only one thing at a time

## How to create an App Clip

- Open your app project and create a new `App Clip` target (found under `Applications`).
- App Clip can embed dependencies via frameworks and libraries (just bare in mind the 10MB limit!)
- You can share code and assets between the main app and App Clips via the file/assets catalog Target Membership (for example, the app icon should be shared)

## Technology Overview

* Can be built using `UIKit` or `SwiftUI`
* [`NSUserActivity`][nsUserActivityDoc] should be used to differentiate between different experiences based on the received App Clip url
* App Clips are allowed to use any API in the iOS SDK, unlike other kind of extensions
* Access to user data in App Clips is more limited than main apps
* There is a "Location Confirmation" API that can be used to confirm where do we think the user's location might be when the user doesn't share its location to an App Clip (more on that in the [`Streamline Your App Clip`][streamlineSessionLink] session)

## Things to Consider

* Access to personal information is limited (e.g. no access to health/fitness)
* Can only be launched by the user or in response to app clip URL
* Universal links, document types, and URL schemes aren't available
* App clips local storage is temporary and will be deleted in couple of days on system discretion, unless the user uses it again, in which case its lifetime is extended and it may never be cleaned up
* We can use App Clips shared data container to migrate data from the App Clip to our main app when the user installs it
* We can use [`SKOverlay`][skOverlayDoc] to offer the full app
* Use [`ASAuthorizationController`][authorizationControllerDoc] to signin or signup

[nsUserActivityDoc]: https://developer.apple.com/documentation/foundation/nsuseractivity
[streamlineSessionLink]: https://developer.apple.com/videos/play/wwdc2020/10120/
[skOverlayDoc]: https://developer.apple.com/documentation/storekit/skoverlay
[authorizationControllerDoc]: https://developer.apple.com/documentation/authenticationservices/asauthorizationcontroller?language=objc
