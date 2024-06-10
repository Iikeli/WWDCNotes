# Meet UIKit for spatial computing

Learn how to bring your UIKit app to visionOS. We’ll show you how to build for a new destination, explore APIs and best practices for spatial computing, and take your content into the third dimension when you use SwiftUI with UIKit in visionOS.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/111215", purpose: link, label: "Watch Video (25 min)")

   @Contributors {
      @GitHubUser(chrisvasselli)
   }
}



## Getting Started
* General tab > Add run destination > visionOS
* Asset Catalog > New app icon
	* visionOS app icons have 3 layers that respond dynamically
	
## Platform Differences
* Cannot use APIs that were deprecated prior to iOS 14
* Cannot use APIs that don't translate well to visionOS. Examples (not an exhaustive list, check the docs):
	* UIDeviceOrientation
	* UIScreen
	* UITabBar leading and trailing accessory views
		* Tab bar is vertical
	* Apple Pencil
* Use `#if !os(visionOS)` to exclude code that can't run on visionOS. 

## Polishing Your App
#### Simulator
* In the simulator, clicking simulates looking at a spot and pinching your fingers.

#### Colors
* Semantic colors are especially important for this platform.
* Use semantic colors, fonts, and materials.
	* `UIColor.label`, `UIColor.secondaryLabel`, etc.
	* Labels using semantic colors get vibrancy by default
* In their case they wanted to update a text field to use `borderStyle = .roundedRect` to give a style that matched the visionOS search bar
* No distinction between dark and light mode on visionOS.


#### Vibrancy and Materials
* `UINavigationController` and `UISplitViewController` get the glass background by default
* Can override background with the new `preferredContainerBackgroundStyle` property on `UIViewController` (`automatic`, `glass` or `hidden`)

#### Hover
* Makes app feel responsive
* Hover effects indicate interactivity
* Exactly where someone is looking is never delivered to your process
* UIView
	* New property `hoverStyle`
	* `UIHoverStyle`
		* `highlight` or `lift`, or remove by setting to `nil`
	* Use `UIShape` to set shape of hover

#### Input
* Looking at something and pinching is like a tap
* Looking at something, pinching, moving your hand, then releasing is like a pan gesture
* If you’re close to the app you can also reach out and touch it
* You can also use a trackpad
* Accessibility has voiceover and switch control
* System gesture recognizers just work
* Maximum of two inputs on this platform (one for each hand). Gesture recognizers that look for more than 2 touches won't work.
* Can use `traitCollection.userInterfaceIdiom == .reality` to check whether we're running in a spatial computing environment, and then don’t use more than 2 touches

## Outside the bounds
### Presentations
* Sheets
	* Pushes the presenting view controller back and dims it
	* Won’t dismiss based on touches outside the bounds or other gestures, regardless of `isModalInPresentation`, unlike iPadOS
* Alerts
	* 2D representation of the app icon is placed at the top of an alert
	* Make sure to always present an alert from the view controller that should be pushed back
* Popovers
	* On iPad, popovers are constrained to the scene, but not in visionOS (similar to macOS)
	* May want to update `permittedArrowDirections` to `.any` on popovers to handle cases where it would now be more natural for the popover to go beyond the bounds of the window.

### Ornaments
* Views that are separate from but anchored to the scene.
* Requires SwiftUI
* Take advantage of the extra space the spatial platform provides
* Scene-relative placement
* Examples in the system
	* Tab View goes on leading side of scene
	* Safari puts the address bar above page content
	* Freeform puts a toolbar at the bottom
* Ornaments are placed forward from the main content, and outside the scene bounds
* Require SwiftUI
* Ornament Alignment
	* Scene alignment and content alignment (add screenshots)
* Create an array of `UIHostingOrnament` objects and set to `self.ornaments`.
* Build SwiftUI content inside initializer `UIHostingOrnament(sceneAlignment: contentAlignment:)` 
* Ornaments share controller lifecycle

### RealityKit
* There's a new SwiftUI view, `RealityView`, used to host RealityKit content (see talk "Build spatial experiences with RealityKit")
* Use `UIHostingController` to use `RealityView` (show example code)
