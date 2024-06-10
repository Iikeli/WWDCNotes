# What's new in Xcode

Discover the latest productivity and performance advancements in Xcode 14. We’ll introduce you to the fully redesigned SwiftUI canvas experience, explore enhancements to code completion and navigation, and take you through performance improvements we’ve made throughout the entire development process. We’ll also show you how you can now read and respond to feedback on your TestFlight builds without ever leaving Xcode.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110427", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



- Xcode 14 is 30% smaller, you must manually download additional platforms/simulators

## Preview canvas

- Interactive Previews (no need to click “Play” button anymore)
- Easy to see multiple dynamic type sizes using the 3rd icon on the bottom left
- First button in bottom left brings back to single view

## New features

- Xcode library now includes all of the SF Symbols
- Parameters with default value are now italic in auto-completion
- Auto-completion of just the arguments you need via fuzzy search
- You can now see all call-sites of a method via command-click
- Compiler checks Regex literals and shows errors at compile-time
- build errors are dimmed to gray when Xcode is reevaluating the diagnostics
  - for example after we change the code where the previous build had an error
  - this helps especially on long builds, where we can now easily tell which problems are from the latest build and which are from a previous build
- The new multiplatform target creates a single SwiftUI interface for use across iOS, iPadOS, macOS, and tvOS
- While you scroll, code structure (like function declarations) stays visible so you always know where you are
- When browsing a long file definition, the current type and function are sticky at the top of the window
- Interface Builder speeds-up:
  - 50% faster loading docs
  - 30% faster switching devices

## Build improvements

- Improved parallelism by eagerly producing Swift modules
- Linking 2x faster (via increased parallelism)
- Overall, Xcode is 25% faster in building
- Xcode 14 has a new build timeline to visualize build times:

![](https://user-images.githubusercontent.com/6942160/172454750-1f418a2f-a443-41b2-b966-b1c177e59d6e.png)

## Parallel testing improvements

- Xcode 14 eliminates scheduling dependencies between targets and test classes to increase parallelism
- up to 30% faster Tests for large test projects

## Ease of use improvements

- 4x faster notarization
- Xcode 14 supports a single target with multiple target destinations
- Xcode 14 expands memory object graph capabilities so that you can see all reference paths in and out of an object
- You can extend Xcode with Swift Package plugins

![](https://user-images.githubusercontent.com/6942160/172454812-c713a03f-ab3e-4c12-a48a-bd372e0f2bf7.png)

- Export localization catalog for packages now possible

![](https://user-images.githubusercontent.com/6942160/172454833-b6a9901c-d059-4f48-9d5f-bec62aa1998b.png)

- Run destination chooser prioritizes recent choices + search (same also in Scheme chooser)

![](https://user-images.githubusercontent.com/6942160/172454847-edd21fe9-75d7-497f-9b56-7f07d3fbe753.png)

- Two new reports in Organizer:

1. Feedback
  - can reply to the feedback via built-in email button 

![](https://user-images.githubusercontent.com/6942160/172454864-3de3c1ba-6448-445a-a71e-0a71b234accf.png)

2. Hangs
  - shows the highest-impact hangs from App Store users so that you know which code to restructure to have the biggest impact

![](https://user-images.githubusercontent.com/6942160/172454886-eb6fd007-beda-48b6-94cd-9bfe4b20128e.png)

- Can use a single 1024x1024 px App Icon asset for the app (make sure to select <kbd>Single Size</kbd>)

![](https://user-images.githubusercontent.com/6942160/172454948-15c42ff8-5201-4f61-9285-2f5d7d0c644a.png)

- Run different types of device features at the same time in SwiftUI Preview
![](https://user-images.githubusercontent.com/74823287/194710788-4e5f1883-38a6-4336-b2cd-421a4d9a077a.gif)


Bonus Xcode 14 Feature! Auto indent 
![](https://user-images.githubusercontent.com/74823287/194443167-e2682e61-071a-42c4-8ac0-5c6c407cd12c.gif)


