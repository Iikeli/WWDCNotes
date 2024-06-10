# Embed the Photos Picker in your app

Discover how you can simply, safely, and securely access the Photos Library in your app. Learn how to get started with the embedded picker and explore the options menu and HDR still image support. We’ll also show you how to take advantage of UI customization options to help the picker blend into your existing interface.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10107", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(RamitSharma991)
   }
}



- The redesigned picker UI in iOS 14 has many great features like search and a zoomable grid. 
- Request any photo library access to use it.
- Replace custom picker with the system one whenever possible. 
- Easy and consice API to present the picker and receive user selected images.

```swift

import SwiftUI
import PhotosUI

struct ContentView: View {

	@Binding var selection: [PhotosPickerItem]
	var body: some View {
		PhotosPicker(
			selection: $selection,
			matching: .images
			) {
				Text(“Select Photos”)
    }
  }
}

```

## Embedded picker

### Privacy: Photos picker is out of process
- It’s really running in a separate process, rendered on top of your app. 
- Your app can't access the picker directly and not even take screenshots of the picker content.
- Only what the user actually selected is passed back to your app. 

### Disabling capabilities
- A new `.photosPickerDisabledCapabilities` modifier allows you to disable certain picker capabilities, to implement your own version of it. 

### Hiding accessories 
- A new `.photosPickerAccessoryVisibility` modifier allows you to hide accessory UI around the picker content, like the navigation bar and the toolbar.

### Setting frame
- specify the size and position of the picker using standard SwiftUI modifiers like `.frame` and `.padding`

### Changing selection
- You can now set selection behavior to `.continuous` to receive live selection updates.

### Embedding picker 
- Use the new `.photosPickerStyle(.inline)` modifier to embed the picker in your app, instead of having it presented as a separate sheet.
- though the picker is embedded, it is still rendered in a separate process. 
- As embedded picker is displayed, an onboarding UI will automatically appear explaining that your app can only access selected photos.  
- The Photos privacy badge indicates that the picker is private and out of process. 
- If your app already presents the full size picker, a dismissible banner will appear when users upgrade to iOS 17. 

### Explained Settings
* Help users understand what your app can access.
* The Privacy Settings UI is also updated with more detailed explanation.
* Changes to the library access permission prompt. 
* For more check [What’s new in privacy](https://developer.apple.com/wwdc23/10053)

### Embedded picker
* If you want to embed the out-of-process picker, use the `.photosPickerStyle` modifier. 
* To place your own UI around the picker, `.photosPickerAccessoryVisibility` is the one to use. 
* It also has an optional argument allowing you to control accessories around specified edges. 
* The default value is all edges.
* To implement the replacement for some picker features, use the `.photosPicker DisabledCapabilities` modifier. 
* To adjust or respond to selection updates in real time, make sure `selectionBehavior` is set to `.continuous` 

### Picker capabilities
* The search bar will be hidden if the search capability is disabled. 
* If collection navigation is disabled, the albums tab will be hidden. 
* On iPadOS and macOS, the sidebar will be hidden as well.
* If the staging area is disabled, the toolbar button will be replaced with the status label.
* If you disabled selection actions without continuous selection, only the `Cancel` button will be hidden and the `Add` button will still be visible. 
* Else, your app won't be able to receive any user selection. 
* Both buttons will be hidden if you also set the selection behavior to `.continuous`

### Picker styles 
* Supports the `.presentation` style ,the `.inline` style, and the `.compact` style. 
* The single row picker allows you to embed it in more places where the available vertical space is seriously constrained. 
* API is available on iOS, iPadOS, and macOS for SwiftUI, UIKit and AppKit apps. 

***Embedded configuration***

```swift

// Create a normal picker
var configuration = PHPickerConfiguration(photoLibrary: .shared()) 


// Specify selection behaviors
configuration.selectionLimit = 0
configuration.selection = .continuous  // or .continuousAndOrdered


// Specify the mode of the picker
continuous.mode = compact // or .default


// Set edges without content margins
continuous.edgesWithoutContentMargins = .all


// Disable certain picker capabilities 
continuous.disabledCapabilities = .selectionActions

```

***Embedding picker***

```swift

// create a picker using a configuration
let picker = PHPickerViewController(configuration: configuration)

// Add the picker as a child view controller
containerViewController.addChild(picker)

// Set the frame of the picker, or use Auto Layout constraints
picker.view.frame = _

// Add the picker’s view as a subview
containerViewController.view.addSubview(picker.view)

// Notify the picker that it has been added
picker.didMove(toParent: containerViewController)

```

***Updating picker***

```swift

//Update picker configurations
var update = PHPickerConfiguration.Update()
update.edgesWithoutContentMargins = []
picker.update(using: update)

// Deselect or reorder assets in the picker
picker.deselectAssets(withIdentifiers: [identifiers])
picker.moveAsset(withIdentifiers: identifiers, afterAssetWithIdentifier: afterIdentifier)

```

## Options menu
* The new Options menu gives users more control over what they can share with your app. 
* By default, all image metadata are included, but users can now choose to remove sensitive metadata, like location, from selected photos.
* With `PhotosPicker` and `Transferable` API, no need to do any adoption work to support the new Options menu.
* No adoption work is needed if you are using the `PHPickerViewController` and `NSItemProvider API`
* Options menu will not be supported for apps that use the legacy `UIImagePickerController` API or have full library access.

## HDR and Cinematic
* The picker may automatically transcode assets to compatible formats, like JPEG, by default. However, the transcoded asset may not contain all information included in the original asset. So if you want to receive HDR content, it's best to avoid automatic transcoding. 
* Avoid transcoding: 
    * Use `.current` encoding policy
    * Use `.image` or `.movie` content type
* Cinematic mode videos:
    * Picker gives the rendered version.
    * Request library access and use `PhotoKit` to get decision points.
