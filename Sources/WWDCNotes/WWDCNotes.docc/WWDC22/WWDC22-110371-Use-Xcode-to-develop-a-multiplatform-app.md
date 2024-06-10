# Use Xcode to develop a multiplatform app

Learn how you can build apps for multiple Apple platforms using Xcode 14. We'll show you how to streamline app targets, maintain a common codebase, and share settings by default. We'll also explore how you can customize your app for each platform through conditionalizing your settings and code.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110371", purpose: link, label: "Watch Video (16 min)")

   @Contributors {
      @GitHubUser(Jeehut)
      @GitHubUser(zntfdr)
      @GitHubUser(elkraneo)
   }
}



## Multiplatform app target

- support many destinations across multiple platforms
- common codebase and settings
- new ways to conditionalize individual settings and files

## Configure project

### Support many destinations across multiple platforms

- recommended especially for SwiftUI or Mac Catalyst projects
- if you're starting from scratch, use the improved <kbd>Multiplatform App</kbd> template 

![multiplatform-app-template](https://user-images.githubusercontent.com/72805/184593197-850fdf02-c859-4c7d-b7eb-828097ab5b84.png)

The multiplatform app template uses SwiftUI for its lifecycle and interface, which starts us out with a target configured by default to support iPhone, iPad, and Mac.

- it's also possible to upgrade an existing codebase:
	- in Xcode, open your project editor and select the app target
	- in the `General` tab, we can see a list of all the destinations our app support
	- there we can add as many destinations as we like

![upgrade-existing-codebase](https://user-images.githubusercontent.com/72805/184593738-a055442c-e20f-4c04-bedf-be01061ad84c.png)

### Choosing between Mac and Mac Catalyst

If your app makes heavy use of UIKit or Storyboards, Mac Catalyst would be a great way to convert an existing iPad app into a compatible Mac app. If your app uses SwiftUI, "Mac option" makes the best choice to craft our, well, Mac app.

### Custom configurations “Display Name” pane

Custom configuration in place editor 

![custom-configuration-in-place-editor](https://user-images.githubusercontent.com/72805/184594756-e50d2e6f-78a1-428a-886b-e4bd066a017f.png)

### Signing Certificate and Provisioning Profile

With Automatic Signing turned on, the necessary Signing Certificate and Provisioning Profile for the Mac is generated on my behalf. All destinations will use the same bundle identifier by default, that means when published to the App Store, they will be made available for Universal Purchase.

### Conditionalize individual settings and files

- Most settings in the target editor now come with a <kbd>Conditions</kbd> option
- <kbd>Conditions</kbd> lets us specify different values based on which SDK is being targeted (macOS, iOS, ..)

![conditionalize-individual-files](https://user-images.githubusercontent.com/72805/184594071-9fee2758-9e4e-4060-b815-855bd3790812.png)

- App capabilities that can be shared across different destinations will get combined into a single entitlements file

## Resolve build issues

### Framework availability

Quick way:

```swift
#if canImport(ARKit)
import ARKit
#endif
```

> This is useful if I don't want to manage a list of known platforms a framework is available for and simply say if it's not available, don't include it.

If, instead, we want to entirely exclude a file entirely when targeting a specific SDK:

- open your target editor and go to the <kbd>Build Phases</kbd> tab
- select your file and, in the <kbd>Filters</kbd> column, specify for which SDK(s) this file should be included

### API availability

Use the `#if os(iOS)` compiler macro

```swift
#if os(iOS)
@Environment(\.editMode) private var editMode
#endif
```

It can be used to hide properties, SwiftUI modifiers, etc.

## Platform experience

- Utilize each platform features
- Refine choices for new expectations
- Rely on SwiftUI for best practices
- Refer to [Human Interface Guidelines][hig]

## Publish app

- Just because we have a single target, it doesn't mean we only have a single product
- We'll need to archive for each platform and upload those individually
- If you're building and archiving locally, you'll need to select a destination that has the SDK you want to create an archive for
- Once I have a destination selected, I can choose "Product Archive" to create the archive
- Once my archives are complete, I can use the Organizer window in Xcode to upload them to App Store Connect

[hig]: https://developer.apple.com/design/human-interface-guidelines/
