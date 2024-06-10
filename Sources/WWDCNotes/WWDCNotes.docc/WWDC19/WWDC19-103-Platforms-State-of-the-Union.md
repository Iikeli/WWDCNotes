# Platforms State of the Union

WWDC 2019 Platforms State of the Union

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/103", purpose: link, label: "Watch Video (117 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- App Store requirement to adapt to all screen sizes from Spring 2020
- tvOS gets Multi-User support plus support for Xbox and Playstation game controllers

## SwiftUI

- Less Code. Better Code. Everywhere. Declarative Syntax.
- Round images by `Image(name).clipShape(Circle()).shadow(radius: 3, y: 3)`
- CornerRadius by `Image(name).cornerRadius(3)`
- Creating a label by `Text("Done!").font(.headline).color(.gray)`
- Changing label to button by `Button("Done!", action: done).font(.headline).color(.gray)`
- Automatic `Spacing & Insets`, `Localizability`, `Right-To-Left`, `Dynamic Type`, `Dark Mode`, ...
- Animations are Repsonsive, Interruptable, Automatic with minimum amount of code
- Build powerful layouts through composition leveraging `VStack` and `HStack`
- Adopt `ObservableObject` for model objects that can change
- Live Development Experience
- Adopt `PreviewProvider` for views/view controllers (surrounded by `#if DEBUG`) to live-preview in canvas
- Multiple previews with different configurations displayable on one canvas
- Adopt SwiftUI gradually at your own pace

## Swift
- Introduction of `Module Stability` which ensures compatibility for binaries of current/future versions of Swift compiler
- GitHub adds Swift Package support to the GitHub Package Registry
- Swift Packages for apps on all platforms seamlessly supported by Xcode 

## Xcode 11
- Add additional editors whenever and whereever you want
- Use related content (Live View, Assistants, ...) in any editor of your workspace. They disappear automatically when there is no content.
- Minimap enables straigt-forward code navigation. Highlights MARK declarations, Issues, Test Failures, ...
- `Add Documentation` feature auto-adds missing function parameters
- Refactoring also changes documentation parameters
- Source-control history moves to the Inspector pane on the right side. Can now always be open.
- Editor displays inline code-change comparison
- TestPlans enable the developer to create many different test combinations (localizations, environment settings, sanitizers, devices). Works with Xcode Server!
- Enable device conditions in Xcode: `Network Link Conditioner`, `Thermal State`
- New app performance metrics for released apps: `Battery Usage`, `Launch Time`, `Hang Rate`, `Memory Use`, `Disk Access`
- TestFlight user feedback lets users share screenshots and comments

## macOS Catalina
- Bring your apps to the Mac by just enable one checkbox in Xcode
- Move kernel extensions out of the kernel into `user space` with `DriverKit` which improves macOS stability for all users
- System volume will become read only. User data and apps are on a read/write volume.
- New permissions for apps that want to do `Keystroke Recording`, `Screen Capture`, `Screen Recording`
- New permissions for apps that want to access directories like `Documents`, `Desktop`, `Downloads`, `iCloud Drive`, `Removable Media`, `Network Volumes`

## watchOS
- The watch becomes independent from the phone by running its own apps, by just checking a checkbox in Xcode
- Independent apps are highlighted on the watchOS App Store
- App Store has fully featured app descriptions. Apps are searchable by dictation and scribble. Direct download of the new, small bundles.
- Notifications, CloudKit, Complication Pushes, Text Fields, Sign In With Apple
- Streaming audio to the watch
- SwiftUI support

## iOS
- New semantic colors that automatically update for the brand new Dark Mode
- New set of adaptive materials and vibrant content filters with variable transparency which support ligh/dark content mode
- Modal view controllers are now displayed as cards by default
- Peek and pop redesigned to show previews with contextual actions
- SFSymbols are 1500+ free, vector assets, variable scales and weights
- Re-designed share sheet

## iPadOS
- Major enhancements to multi tasking by new UIWindowScene API
- New `SceneDelegate` to manage multiple windows at the same time from one app
- New state restauration system using `NSUserActivity` 
- PencilKit framework for customize drawing in your app
- Major simplification in text selection. `UITextInteraction` API to fix conflicting-gesture issues
- ScrollViewScrubbing: Grab and drag the any scrollview scroll indicator

## Accessibility Features
- Accessibility has been added to the iOS Quick Start guide
- Accessibility has been moved one hierarchy level up into the Settings main menu
- Accessibility sub-menus menu got icons for better discoverability
- Voice Over can display a grid for better selectability. You can show accessibility labels and much much more. It's an incredible feature!
- Showing accessibility labels via Voice Over completely changes how we can test them. They are now just displayed on screen!

## Privacy
- Good Practice: Process user data on device whenever you can & Ask for permission when processing user data & Encrypt user data
- Allow once is a new option when requesting location permission
- Apps location usage shown on a map by apple from time to time
- Sign in with Apple provides fast, easy sign in without tracking by tapping a `Sign in with Apple Button` and a quick FaceID
  - No more email address verification since apple already done this!
  - All AppleID-emails are protected by 2FA
  - Built in fraud detection
  - Apple provided email relays protect users real email address
  - Cross platform support: Works on Android and Windows devices (on the web)
  - No tracking by Apple

## Machine Learning
- Happens on-device
- Engine is capable of 5 trillion operations per second
  - Image Saliency provides image heatmap
    - Highlights important objects and where users are likely to focus
    - Helps auto-cropping photos

- 
  - Text Recognition
    - Search text from signs, posters and documents
    - Take advantage of the document camera capability from the Notes app

- 
  - On-Device speech API with support for 10 languages

- CoreML supports more than 100 layer-model types which brings e.g. breakthrough natural language processing to your apps
  - Core ML On-Device Personalization enables developers to update models from inside the app (in the background)
  - Create ML becomes a macOS app enabling users creating models without writing code
    - Choose from different model templates
    - Build multiple models with different data sets and define parameters of each
    - Real-time feedback on model training 
    - Supports transfer learning (e.g. for image classification) which speeds up training
    - Experiment and preview models, e.g. use microphone to test sound classification models

## Siri Shortcuts

- Make them discoverable by adding the **Add to Siri** button to your apps
- Apple made them conversational by adding **Parameters**
- Shortcuts app is build into iOS and iPadOS
- Added **Automation** which allows to set triggers for when to run specific shortcuts
- Configurable 3rd-party actions enable combination of multiple actions (multi-step shortcuts)
  - Synced via iCloud to all of your devices

## ARKit

- Integration of AR Quick Look with Apple Pay to try out clothes, glasses, and other products (and immediately by them)
- Worlds largest AR platform
- 3 technologies that make it easy to develop AR experiences: `ARKit`, `RealityKit`, `Reality Composer`
- `Reality Composer` lets you layout AR experiences in 3D on macOS, iOS and iPadOs
- `RealityKit` is a modern real-time 3D engine. Designed for AR. Easy integration with existing apps. 
  - Uses physically based rendering, Data driven rendergraph, Multi-threaded
  - Access through native Swift API framework

- `ARKit 3`
  - Simultaneous from and back camera
  - Motion capture by ML algorithm
  - People occlusion detection by ML algorithm

## Metal

- Modern high performance (100x OpenGL) GPU programming. Low overhead. Easy to use.
- Core layer under all of Apples frameworks: UIKit, AppKit, Core Animation, SpriteKit, RealityKit, ARKit, MapKit, WebKit, Core ML, Create ML, VisionKit, Core Image, Camera Processing
- Full support in iOS Simulator brings major performance improvements
  - Metal Compute provides building blocks for general purpose computation: `Shading Language`, `Compiler`, `Compute Encoding`, `Metal Performance Shaders`, 
  - Metal Tile Shading combines compute shaders and fragment part into one highly-efficient render pass
  - Metal Indirect Compute Command Encoding lets you build GPU compute commands right on the GPU

- Amazing Metal-based, animated Raytracing demo with computationally expensive multi-reflections and more
- "Built for Pros" 😅
