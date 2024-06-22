# Taking iPad Apps for Mac to the Next Level

macOS Catalina provides an easy way to bring your iPad app to the Mac while maintaining your single code-base. Hear about ways in which you can take your app beyond the default behaviors to optimize its interface for the Mac. Get an overview of APIs you can use and macOS design guidelines that need to be considered. Learn how the iPad app lifecycle comes across on the Mac, and get distribution details for your application.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/235", purpose: link, label: "Watch Video (54 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Best practices/Must-have

**Make it scale**

Your app now runs from a tiny 4” iPhone 5S, all the way to a full screen on a 27-inch iMac, and that's with a 77% skill factor. In fact, the 27-inch display is more like a **35-inch iPad**.

Therefore: support dynamic type and use Auto Layout, and lastly, remember that on the Mac app resize fast: resizes should run at 60 frames per second.

**Implement Great Keyboard support**

use `UIKeyCommand`

**Menu Bar**

Support menu bars (you can even use storyboards for this). 

Alternatively, override the `buildCommands(with:)` app delegate method.

**Use `UIHoverGestureRecognizer`**

..where it makes sense.

**Support the Touchbar**

**Author a Help Book**

**Sidebars**

Make sidebars look like a macOS sidebars (with the translucency as background): this should be done when in split mode and the master view acts as a toolbar.

```swift
#if targetEnvironment(UIKitForMac) 
primaryBackgroundStyle = .sidebar 
#else 
```

**Toolbars**

Add `NSToolbar`, used in macOS for most common actions:

Reconsider tabbars and make them a segmented controller on the top instead (on macOS).

**Lifecycle Considerations**

UIKitForMac apps will basically permanently stay in a foreground + active state, rarely they’ll move to a background state etc. 

The app will move to other states only when macOS determines that the app is not active/in use, however there’s no clear way to predict this. Do not rely on those statuses for critical logic.

**Distribution**

Unified certificate for both iOS and macOS distribution.

In iOS, we archive the app with “Generic iOS Device” as target in order to export our app for the store and upload it as an IPA. 

In MacOS, we archive the app with “My Mac” as target in order to export our app for the store and upload it as an Mac package. 

**Capabilities**

Most are automatically enabled by looking at the `info.plist` file, however some needs to be manually activated: in particular keychain and push notification capabilities.

[targetImage]: ../../../images/notes/wwdc19/235/target.png
[segmentImage]: ../../../images/notes/wwdc19/235/segment.png