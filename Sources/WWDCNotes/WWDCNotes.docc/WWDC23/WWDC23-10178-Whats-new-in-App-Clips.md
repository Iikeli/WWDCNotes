# What’s new in App Clips

Explore the latest updates to App Clips. We’ll show you how to build App Clips more easily using default App Clip links. Learn how you can take advantage of the increased App Clip size limit to build richer and more engaging experiences, and find out how you can launch App Clips directly from your app.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10178", purpose: link, label: "Watch Video (6 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Kevin Turner, App Clips Engineer

## New size limit

Starting with iOS 17, there is a new size limit for digital invocations of App Clips at 50 MB.

If you'd like to make use of physical invocations, such as those through NFC tags or App Clip codes, you'll have to keep to the 15 MB limitation introduced in iOS 16.

If you're targeting iOS 15 or earlier, the original 10 MB limitation still applies.

## Default App Clip links

> Supported starting from iOS 16.4.

If your App Clip only needs one App Clip experience, you can use the default App Clip experience.

Now you don't need to setup the required associated files on your own website to support the Universal link that opens the App Clip.

You can now use the following URL to launch App Clips:

`https://appclip.apple.com/id?p=<bundle_id>&key=value`

This URL is automatically generated in App Store Connect, when you create your App Clip experience.

The parameters at the end are custom parameters you specify.

Access this data in SwiftUI using `.onContinueUserActivity(NSUserActivityTypeBrowsingWeb, perform: { activity ... })` modifier and access `activity.webpageURL`.

*For more information on configuring your App Clip experiences, refer to "Configure and link your App Clips" from WWDC 2020.*

## Invoke from your app

### Show an App Clip link in app

You can use the link presentation API to generate a tappable rich preview of the App Clip that will allow it to be invoked. Once you've retrieved the metadata via a `LPMetadaProvider` request, you can pass that along to the `LPLinkView` to render a preview.

```swift
let provider = LPMetadataProvider()

provider.startFetchingMetadata(for: url) { (metadata, error) in
    guard let metadata = metadata else {
        return
    }

    DispatchQueue.main.async {
        lpView.metadata = metadata
    }
}
```

### Launch App Clip from SwiftUI

```swift
var body: some View {
    let appClipURL = URL(
        string: "https://appclip.apple.com/id?p=com.example.naturelab.backyardbirds.Clip"
    )!

    Link("Backyard Birds", destination: appClipURL)
}
```

### Launch App Clip from `UIApplication`

```swift
func launchAppClip() {
    let appClipURL = URL(
        string: "https://appclip.apple.com/id?p=com.example.naturelab.backyardbirds.Clip"
    )!

    UIApplication.shared.open(appClipURL)
}
```
