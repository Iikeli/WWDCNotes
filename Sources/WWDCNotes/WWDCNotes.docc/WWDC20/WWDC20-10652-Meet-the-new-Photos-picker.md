# Meet the new Photos picker

Let people select photos and videos to use in your app without requiring full Photo Library access. Discover how the PHPicker API for iOS and Mac Catalyst ensures privacy while providing your app the features you need.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10652", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



PHPicker: a new system-provided picker screen that gives you access to photos and videos from the user’s photo library.

It’s recommended that you use this picker instead of building your own custom photo selection UI.

New version includes:

- a new design and new easy to use API
- an integrated search
- easy multi-select
- zoom gesture

PHPicker is private by default:

- the picker screen runs out of process and talks to the app via XPC
- your app has no direct access to the photos library
- it doesn’t need to get photo library permission (don’t ask for it unless you *really* need it)
- you only get selected photos and videos in response

PHPicker is not a name of a single class, but a set of classes that work together.


## Elements of the API

[`PHPickerConfiguration`](https://developer.apple.com/documentation/photokit/phpickerconfiguration) – lets you specify limits and filters:

- `selectionLimit` – number of items that can be selected (1 by default, 0 = unlimited)
- `filter` – e.g. `.images` or `.any(of: [.videos, .livePhotos])`

[`PHPickerViewController`](https://developer.apple.com/documentation/photokit/phpickerviewcontroller) – the main view controller handling the picker:

- the picker doesn’t dismiss itself automatically, call `picker.dismiss(animated:)` when you get the response

[`PHPickerViewControllerDelegate`](https://developer.apple.com/documentation/photokit/phpickerviewcontrollerdelegate) – delegate for the picker:

- [`picker(_: didFinishPicking results:)`](https://developer.apple.com/documentation/photokit/phpickerviewcontrollerdelegate/3606609-picker)

[`PHPickerResult`](https://developer.apple.com/documentation/photokit/phpickerresult) – an array of these objects is passed to the app in response

- get `itemProvider` from the result
- check `itemProvider.canLoadObject(ofClass: UIImage.self)`
- get the image via `itemProvider.loadObject(ofClass: UIImage.self) { … }`

You can normally extract picked photos from `PHPickerResult` item providers without touching the `PHPhotoLibrary` at all, but if you do need to access the photo library anyway, then pass it to [`PHPickerConfiguration.init`](https://developer.apple.com/documentation/photokit/phpickerconfiguration/3616114-init) and get `assetIdentifier` references from the picker results.

If you use PHPicker with photo library access and you only got limited access to a subset of photos, then:

- PHPicker will still let the user choose photos from their whole library
- but the selection you have direct access to will not be extended by what they choose in the picker

The photo library APIs from [`UIImagePickerController`](https://developer.apple.com/documentation/uikit/uiimagepickercontroller) are deprecated.

> Full blog post here: [https://mackuba.eu/2020/07/07/photo-library-changes-ios-14/](https://mackuba.eu/2020/07/07/photo-library-changes-ios-14/)
