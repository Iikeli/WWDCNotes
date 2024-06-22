# SwiftUI On All Devices

Once you’ve learned the basics of SwiftUI, you’ve learned what you need to know to use SwiftUI anywhere. You can use the same SwiftUI skills for making an iOS app as you would for making an app on watchOS, tvOS or macOS. We'll cover the basics, and then dig into more detail about how  SwiftUI can help you make changes to your app on every Apple device. Hear about design principles for each platform and learn about how much code you can share across platforms. See how to incorporate device-specific features and how to make changes in SwiftUI by following along with a starter project, available for download.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/240", purpose: link, label: "Watch Video (45 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- SwiftUI is a common toolkit to learn once and use on all platforms (macOS, iOS / iPadOS, tvOS, watchOS)
- Cross-platform common elements like the `Toggle Cnotrol`, `Picker Control` and the whole `Layout System`

- **One size doesn't fit all devices**
  - But: "Learn once, apply anywhere" - base principle of SwiftUI
  - Use appropriate design patterns for each screen size (e.g. `List` on iPhone `UICollectionView` on iPad)
  - We still have to build 4 apps - for each platform

- **SwiftUI on Apple TV**
  - Carefully cosider what's appropriate to show on a big screen in the living room
  - Entire interface must support the tvOS feature `focus` which is supported by many SwiftUI elements, e.g. List
  - Elements have commands like `.focusable { isFocused in /* focus changed */ }`, `.onPlayPauseCommand { /* play/pause button pressed */ }`, `.onExitCommand { /* menu button pressed */ }`
  - iOS > tvOS adoption Demo at [15:20](https://developer.apple.com/wwdc19/240/?time=920)

- **SwiftUI on Mac**
  - Great device to provide more information (e.g. text)
  - SwiftUI auto-adapts spacing and padding
  - Access small- and mini-size controls using `.controlSize()` modifier
  - Consider proper support of multiple windows if users of your app may want to:
    - Compare content across windows side-by-side
    - Focus on a single item in its own window
    - Spatially organize windows around Desktop and Spaces

- 
  - Support keyboard shortcuts

- 
  - **Touch Bar**
    - SwiftUI makes it easier than ever to support this element
    - Just call the `.touchbar()` modifier of a view with a `TouchBar {}` parameter and fill the closure with one or multiple `Button` controls

- **SwiftUI on Apple Watch**
  - The watch is all about showing the right information at the right time
  - The most critical action of your app should be available within 2-3 taps
  - `.digitalCrownRotation` API to control crown rotation and haptics
  - Use `ScrollView`, `List`, `HStack`, `VStack` to group and layout your app
  - Use `.listStyle(.carousel)` when you have a few cell count or cells with interactive controls
