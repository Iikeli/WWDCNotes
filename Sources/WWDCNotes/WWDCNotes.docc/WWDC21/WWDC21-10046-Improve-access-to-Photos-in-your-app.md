# Improve access to Photos in your app

PHPicker is the simplest and most secure way to integrate the Photos library into your app — and it’s getting even better. Learn how to handle ordered selection of images in your app, as well as pre-selecting assets any time the picker is shown. And for apps that need to integrate more deeply with PhotoKit, discover how you can use PHCloudIdentifier to sync photo project content across devices, helping people easily transition their image work between iPhone, iPad, and Mac.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10046", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



## Improvements to the Photo Picker

### Privacy

When an app doesn't ask for a Photo Library access and only presents the Photo Picker - which runs out of process and automatically has access to the whole library, from which the user can pick some photos for the app - it was possible for people to misinterpret this workflow and assume that the app itself had a full access to the whole photo library, even if it didn't.

So now, in the Settings app, on the Photos Privacy screen, there is a new section for apps that only use the Photos Picker, titled "Apps with one-time photo selection".

### Ordered selection

In some apps, users may need to be able to select a few photos in a specific order (e.g. when adding them to a social media post). In iOS 15, your app can configure the picker to show selection order as badges ① ② ③ etc.

To do that, set the [`selection`](https://developer.apple.com/documentation/photokit/phpickerconfiguration/3752714-selection) property in the picker configuration to `.ordered`:

```swift
var configuration = PHPickerConfiguration()
configuration.selectionLimit = 0
configuration.selection = .ordered
```

### Selection adjustment

You can now preselect some photos in the picker to display user's previous selection that they can edit, adding additional photos or removing some previously selected ones.

To preselect photos, pass an array of asset identifiers to the picker configuration:

```swift
let assetIdentifiers: [String] = previousSelection

var configuration = PHPickerConfiguration(photoLibrary: .shared)
configuration.selectionLimit = 0
configuration.preselectedAssetIdentifiers = assetIdentifiers
```

⚠️ One caveat: when the picker returns updated results, all photos that have been previously selected (and included in the preselection config you provide) will not include the item providers - so you need to keep the original list of photos with their data until after the updated picker results are returned. If the preselected picker dialog is cancelled, it will return the preselected assets you've provided with all item providers empty.

In the delegate callback, you need to merge together the old results with the new results, taking the value from the old results for any photo that was selected previously (since there won't be an item provider in the new one):

```swift
func picker(_ picker: PHPickerViewController, didFinishPicking newResults: [PHPickerResult]) {
    dismiss(animated: true)

    let existingSelection: [String: PHPickerResult] = self.lastSelection
    var newSelection = [String: PHPickerResult]()

    for result in results {
        let identifier = result.assetIdentifier!
        newSelection[identifier] = existingSelection[identifier] ?? result
    }

    self.lastSelection = newSelection

    // do something with selected assets
}
```

### Reporting progress

Some assets that the user selects in the picker may not be immediately available and need to be downloaded from the cloud first, which may take a moment (especially for videos) - this can happen if they're selecting something from their iCloud Photos and the "Optimize Storage" option is turned on. In that case, previously your app could only show a spinner indicator, since it didn't have access to information about the download progress.

Now, the asset's item provider can give you a `Progress` object that you can use to track download progress and show a more appropriate loading UI:

```swift
let result: PHPickerResult = ...
let provider: NSItemProvider = result.itemProvider
let identifier: String = UTType.movie.identifier

if provider.hasItemConformingToTypeIdentifier(identifier) {
    let progress: Progress = provider.loadFileRepresentation(forTypeIdentifier: identifier) { url, error in
        // Do something with the video, or handle the error
    }

    // Show progress in the meantime
}
```


## New cloud identifier APIs

There are some categories of apps that require full or partial direct access to the user's photo library, e.g. photo editing apps, camera apps or apps whose purpose is to display the photo library in some specific way. Those apps use the PhotoKit APIs to access and modify the photo library.

When using those APIs, assets are returned with unique identifiers that your app can save for later and then use again to retrieve the same assets on the next launch. These identifiers are specific to each device, even if some of the assets are synced between devices using iCloud Photos.

If your app syncs some user-generated data that references user's photos between devices and you want to access the same photos on multiple devices, you can use the new cloud identifiers that identify the same photo globally. There is a mapping between local identifiers and cloud identifiers and you can use a cloud identifier to look up a local copy of the photo. Cloud identifiers work even if the device is not signed into iCloud Photos (or even never was). The new identifiers are represented by [`PHCloudIdentifier`](https://developer.apple.com/documentation/photokit/phcloudidentifier) objects.

### How to use cloud identifiers

1. Get some local photos from a device's library using CloudKit
2. Map local identifiers to cloud identifiers:

```swift
let cloudMappings = PHPhotoLibrary.shared()
        .cloudIdentifierMappings(forLocalIdentifiers: localIdentifiers)

for (localIdentifier, cloudMapping) in cloudMappings {
    if let cloudIdentifier = cloudMapping.cloudIdentifier {
        // save the cloudIdentifier for later
        resolved[localIdentifier] = cloudIdentifier
    } else {
        // handle the cloudMapping error
    }
}
```

3. Transfer the cloud identifiers to other devices using whatever communication method you want to use (iCloud / CloudKit etc.)
  - use [`stringValue`](https://developer.apple.com/documentation/photokit/phcloudidentifier/2909174-stringvalue) to encode the identifier to a string

4. Look up local identifiers on each device based on the cloud identifiers in the same way

```swift
let localMappings = PHPhotoLibrary.shared()
        .localIdentifierMappings(for: cloudIdentifiers)

for (cloudIdentifier, localMapping) in localMappings {
    if let localIdentifier = localMapping.localIdentifier {
        // add the localIdentifier to our resolved assets
        resolved[cloudIdentifier] = localIdentifier
    } else {
        // handle the localMapping error
    }
}
```

5. Use the local identifiers to fetch the assets and display them

### Error handling

The mapping in both directions may return an error instead of the identifier, so you need to be able to handle that case. There are two possible kinds of errors to take into account:

1) Identifier Not Found - if the app isn't able to find or access the relevant record:

```swift
let error = localMapping.error! as NSError

if error.code == PHPhotosError.identifierNotFound.rawValue {
    // couldn't find this photo, add it to the missing photos list
    missingPhotos.append(cloudIdentifier)
}
```

2) Multiple Identifiers Found - this can happen if the cloud state isn't completely in sync and the app tries to find the image using content match and finds multiple copies; in this case, the error info will contain the list of matching identifiers under [`PHLocalIdentifiersErrorKey`](https://developer.apple.com/documentation/photokit/phlocalidentifierserrorkey).

```swift
let error = localMapping.error! as NSError

if error.code == PHPhotosError.multipleIdentifiersFound.rawValue {
    // found multiple matches, prompt the user to pick one
    let matches = error.userInfo[PHLocalIdentifiersErrorKey] as! [String]
    multipleMatches[cloudIdentifier] = matches
}
```

Note: looking up cloud identifiers takes some work, so use local identifiers for normal app interactions and map them to cloud identifiers only for syncing with other devices.


## Updates to the limited library

The limited library is when you access the user's photo library using PhotoKit after the user has requested to only share selected photos with your app. This is designed to work transparently for your app - all PhotoKit APIs work fine, but they work as if the photo library only contained those selected photos and nothing else. (See last year's talk [Handle the Limited Photos Library in Your App](/notes/wwdc20/10641/) for more info.)

In iOS 15, apps can now create, fetch and update their own photo albums within the user's photo library when running in the limited library mode.

Also, when you call [`photoLibrary.presentLimitedLibraryPicker()`](https://developer.apple.com/documentation/photokit/phphotolibrary/3616113-presentlimitedlibrarypicker) to let the user adjust a previous selection of photos shared with your app, you can now provide a callback which will be given a list of photos that have just been added to the selection:

```swift
let library = PHPhotoLibrary.shared()
library.presentLimitedLibraryPicker(from: controller) { addedIdentifiers in
   // fetch the newly added photos and use them in your app 
}
```

If your app still uses the old [Assets Library framework](https://developer.apple.com/documentation/assetslibrary) that was deprecated in iOS 9 (`ALAssets*`) - please switch to PhotoKit and the Photos Picker, the old API will be removed in a future SDK.
