# Run your iPad and iPhone apps in the Shared Space

Discover how you can run your existing iPad and iPhone apps on Vision Pro. Learn how iPadOS and iOS apps operate on this platform, find out about the Designed for iPad experience, and explore the paths available for enhancing your app experience on visionOS.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10090", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(coughski)
   }
}



## Introduction

* Easily run your existing iPad and iPhone apps on Apple Vision Pro.
* The majority of apps will work great right out of the box.

## Built-in behaviors

#### Layout

- iPad and iPhone apps are displayed as windows with light mode style
- System prefers using iPad landscape orientation, otherwise uses iPhone aspect ratio with portrait orientation
- Rotation button automatically provided if other orientations supported
- Scaling controls provided and preserve aspect ratio

#### Input and interaction

Users may interact with content using any of the following methods:
- Look to select and tap with fingers
- Reach out and touch content directly
- Bluetooth trackpad or game controller

All methods send events already familiar to your app

System views like photo picker will match system appearance

`LocalAuthentication` handles Touch ID or Face ID requests through Optic ID instead

No way to rotate Apple Vision Pro, so your app may want to specify a preferred rotation for new scenes

Use `UIPreferredDefaultInterfaceOrientation` plist key for this (new key for visionOS)

`UISupportedInterfaceOrientations` used by system to provide window rotation controls

`UIRequiredDeviceCapabilities` App Store Connect uses to determine compatibility with Apple Vision Pro

All suitable apps are automatically made available on the App Store

## Functional differences

Gestures: Max of 2 simultaneous inputs supported

Existing `ARView`s and `ARSession`s need to be rebuilt, won't work on visionOS. See "Re-imagine your ARKit app for spatial experiences [_sic_]"
(No session with that name exists; probably referring to [Evolve your ARKit app for spatial experiences](https://developer.apple.com/videos/play/wwdc2023/10091), maybe [Meet ARKit for spatial computing](https://developer.apple.com/videos/play/wwdc2023/10082).)

Location approximated via Wi-Fi or shared from iPhone

Look to dictate interaction available on search bars

Disabled by default on iPhone and iPad apps. [Code sample 07:59](https://developer.apple.com/videos/play/wwdc2023-10090/?time=479).

Enable by adding `.searchDictationBehavior(.inline(activation: .onLook))` to SwiftUI

Or use `searchController.searchBar.isLookToDictateEnabled = true` with UIKit

Use availability checks and verify hardware configurations for presence and support

SpriteKit and Storyboards only supported in iPad apps

Just build and run your apps using the "visionOS Designed for iPad" target run destination

## Choose your experience

To use `ARKit`, `RealityKit`, `Volume`, `ImmersiveSpace`, visionOS style, or ornaments, redesign your app for visionOS
