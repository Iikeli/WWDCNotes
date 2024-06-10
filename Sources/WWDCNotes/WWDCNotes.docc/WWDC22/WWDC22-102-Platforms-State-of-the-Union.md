# Platforms State of the Union

Take a deeper dive into the latest tools, technologies, and advances across Apple platforms to help you create even better apps.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/102", purpose: link, label: "Watch Video (70 min)")

   @Contributors {
      @GitHubUser(zntfdr)
      @GitHubUser(fbernutz)
   }
}



![Sketchnote about WWDC22 Platforms State of the Union talk with news about Swift, SwiftUI, System Experiences and new APIs][sketchnote]

## Xcode Cloud

- available starting today

### Pricing

| 25 hours/month | 100 hours/month | 250 hours/month | 1000 hours/month |
| --- | --- | --- | --- |
| $14.99/mo. | $49.99/mo. | $99.99/mo. | $399.99/mo. |

- 25 hours/month free until end of 2023

## Vision for the Developer Platform

### Swift

- [`swift-async-algorithms`][swift-async-algorithms] - open-source package that brings concurrency to Swift's rich set of existing sequence algorithms
- new set of clock types for representing time units
- new set of async algorithms builds on these clock types to provide many time-based algorithms, e.g.:

```swift
fastEvents.throttle(for: .seconds (1)) // helps slow down updates from a sequence
```

- [distributed actors][swift-distributed-actors]
  - actors can communicate across multiple processes or devices. The `distributed` keyword marks these actors and methods that can be accessed remotely
  - just as actors help Swift protect your state data from race conditions, distributed actors help Swift make them available outside your process, using a pluggable transport mechanism

- regular expressions
  - new regular expression literal
  - built directly into the language, allowing the Swift compiler to check for correctness
  - unlock the power of Swift's type system when you're extracting information with a regular expression
  - Unicode support

```swift
let order = "Order from <Betsy's Bakery>, type: maple frosted, count in dozen: 3"
let regex = /Order from <(.*)>, type: (.*), count in dozen: ([0-9]+)/
order.firstMatch(of: regex)

let log = "0   FoodTruckKit    0x000000018a0e5d88 Donut.bake() + 144"
let logRegex = /\s+([a-zA-Z_.]+)\s+(0x[\da-fA-F]+)/
log.firstMatch(of: logRegex)
```

- Regex Builders
  - can directly convert a regular expression literal into a regex builder

```swift
let log = "0   FoodTruckKit    0x000000018a0e5d88 Donut.bake() + 144"
let logRegex = Regex {
  OneOrMore(.whitespace)
  Capture {
    OneOrMore {
      CharacterClass(
      	.anyOf("_."),
      	("a"..."z"),
      	("A"..."Z")
      )
    }
  }
  OneOrMore(.whitespace)
  Capture {
    "Ox"
    OneOrMore(.hexDigit)
  }
}

log.firstMatch(of: logRegex)
```

- Swift Generics improvements
- Package Plugins
  - instead of code in your app, they're code that helps build your app
  - can be invoked from the command line or within Xcode, either as part of your build phase or on-demand
  - run in a sandbox environment which prompts you for permission before reading or modifying your code
  - examples: linting, formatting, code generation
  - can run on Xcode Cloud

- build improvements
  - new parallelization efforts
  - (static) link time is up to twice as fast

- Swift concurrency runtime is now more tightly integrated with the OS
- launch time for apps written in Swift is dramatically faster on iOS 16 (up to 2x)
- wall of features:
  - Predictable ARC
  - Compile-time constants
  - `any` for all protocols
  - Flexible pointer comparisons
  - Implicit type parameters
  - Lightweight requirements
  - Type placeholders
  - SwiftPM plugins
  - Top-level async
  - Pointer comparisons
  - Pairwise result builders
  - Optimizations written in Swift
  - Distributed actors
  - Flexbile key coding
  - Regex literals
  - Better memory binding
  - Temporary buffers
  - Multi-statement closure type inference
  - Faster declaration checking
  - Protocol opening
  - Faster linking
  - Protocol caching on startup
  - @Sendable closures
  - Efficient Array metadata
  - Eager compilation
  - Async Algorithms
  - Unicode character classes
  - Concurrency back-deployment
  - `if let` shorthand
  - Clock types
  - Function back-deployment
  - Module disambiguation
  - Pointer conversion for C calls
  - Sendable
  - Native grapheme breaking
  - Faster `Set` operations

### SwiftUI

- new navigation API
  - programmatic control over view presentation
  - can easily save and restore selection 
  - can replace the full contents of a navigation stack

```swift
NavigationStack(path: $navPath) {
  MusicLibraryView()
    .navigationDestination(for: Album.self) { AlbumDetail($0) }
    .navigationDestination(for: Artist.self) { ArtistDetail($0) }
}
```

- `Grid`
  - new API that makes it easier to lay out a set of views aligned across multiple rows and columns
  - new custom layout API
    - gives you the flexibility to build any type of layout you want
    - e.g., flow layout, radial layout

- half sheet API (UIKit's Detents API)
- share sheet support
- new `UICollectionViewCell` [`contentConfiguration`][contentConfiguration] of type `UIHostingConfiguration` for hosting SwiftUI views in collection view cells:

```swift
cell.contentConfiguration = UIHostingConfiguration {
	// your SwiftUI view here
}
```

- [Swift Charts][charts]
  - highly customizable charting framework
  - makes it easy to create visualizations
  - uses the same declarative syntax as SwiftUI

```swift
Chart(data) { element in
  BarMark(
  	x: .value("Product", element.product),
  	y: .value("Sales", element.sales)
  )
}
```

- New [`ViewThatFits`][ViewThatFits] view
  - A view that adapts to the available space by providing the first child view that fits

```swift
ViewThatFits {
  HStack {
    // content
  }
  VStack {
  	// content
  }
}
```

- (macOS) new `MenuBarExtra` API
  - menu bar extras are those are the icons on the upper right corner of your screen, like Wi-Fi and Spotlight
  - add it to the body of your app

```swift
@main
struct MyApp: App {
  var body: some Scene {
    WindowGroup {
    	// ..
    }
    #if os(macOS)
    MenuBarExtra {
      ScrollView {
        VStack(spacing: 0) {
          BrandHeader(animated: false, size: .reduced)
          Text("Donut stuff!")
        }
      }
    } label: {
      Label("Food Truck", systemImage: "box.truck")
    }
    .menuBarExtraStyle(.window)
    #endif
  }
}
```

## System Experience

### Lock Screen widgets

- essentially watchOS complications for iOS
- Circular widget
- Rectangular
- Inline - tiny amount of text/SF Symbols above the clock on iPhone
- from WatchOS 9 complications also work with WidgetKit

### Live Activities

- also backed by WidgetKit and SwiftUI
- can animate your updates from one state to the next

### Messages Collaboration API

- new way to enhance collaborative experiences
- can start a collaboration via the or drag & drop
- uses existing API

### [App Intents Framework][AppIntents]

- Automatic generation of [App Shortcuts][app-shortcuts], without the need for the user to manually create or add to Siri
- no separate intent definition files or code generation, your code is the source of truth
- If you adopted Intents to integrate with Widgets or domains like media or messaging, you should keep using the [SiriKit Intents framework][sirikit]
- For developers who build custom intents for Siri and Shortcuts, you should go ahead and upgrade to App Intents
  - Xcode can convert your custom intents to App Intents

## New APIs

- [DriverKit][driverkit] is now available on iOS 16 and iPadOS 16
- [CallKit][CallKit] is now available in watchOS 9 and macOS 13
- ScanKit and RoomPlan have new features that use AR and LiDAR scanning
- [Focus filters][focus] let you adjust the content of your app based on the user's current focus
- [MapKit][mapkit] 
  - new 3D City Experience available, camera controls
  - Look Around
  - Apple Maps Server APIs
    - Geocode - turns a lat/long into an address
    - Reverse Geocode - turns an address into GPS coordinates
    - Search
    - Estimated Times of Arrival

- New [WeatherKit][weatherkit] (available in all platforms), pricing:

| 500K API calls/month | 1M API calls/month | 2M API calls/month | 5M API calls/month | 10M API calls/month | 20M API calls/month |
| --- | --- | --- | --- | --- | --- |
| $0/mo. | $49.99/mo. | $99.99/mo. | $249.99/mo. | $499.99/mo. | $999.99/mo. | 

- [VisionKit][visionkit]
  - [live text API][enabling_live_text_interactions_with_images]
  - [data scanner API][datascannerviewcontroller] 
  - new automatic language detection for Japanese and Korean

[swift-async-algorithms]: https://github.com/apple/swift-async-algorithms
[swift-distributed-actors]: https://github.com/apple/swift-distributed-actors
[contentConfiguration]: https://developer.apple.com/documentation/uikit/uicollectionviewcell/3600949-contentconfiguration
[charts]: https://developer.apple.com/documentation/charts
[ViewThatFits]: https://developer.apple.com/documentation/swiftui/viewthatfits
[AppIntents]: https://developer.apple.com/documentation/AppIntents
[app-shortcuts]: https://developer.apple.com/documentation/appintents/app-shortcuts
[sirikit]: https://developer.apple.com/documentation/sirikit
[driverkit]: https://developer.apple.com/documentation/driverkit
[CallKit]: https://developer.apple.com/documentation/callkit
[focus]: https://developer.apple.com/documentation/appintents/focus
[mapkit]: https://developer.apple.com/documentation/mapkit?changes=latest_minor
[weatherkit]: https://developer.apple.com/documentation/weatherkit
[visionkit]: https://developer.apple.com/documentation/visionkit
[enabling_live_text_interactions_with_images]: https://developer.apple.com/documentation/visionkit/enabling_live_text_interactions_with_images
[datascannerviewcontroller]: https://developer.apple.com/documentation/visionkit/datascannerviewcontroller
[sketchnote]: https://fbernutz.github.io/images/sketchnotes/wwdc22-state-of-the-union.jpg
