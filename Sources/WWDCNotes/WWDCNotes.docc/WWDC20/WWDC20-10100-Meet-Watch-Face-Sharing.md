# Meet Watch Face Sharing

Show off your watchOS app’s complications and create a watch face worth sharing. Learn how to share watch faces inside your watchOS and iOS apps or host them on the web for anyone to find and download. We’ll also explore best practices for using watch face preview images, and show you how to create a smooth installation experience.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10100", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- Face sharing is the ability to share a watch face with anyone (requires a Series 3 or later). 
- Watch faces include complication data.
- if you include a complication in the watch face that the user doesn't have, they will be prompted to install the missing apps when they add the face.

## Watch face distribution

Watch faces can be shared:

- directly from the watch by long-pressing on your face and tapping the new "share" button.
- from the Watch.app
- from apps and websites

## Watch face file

A `.watchface` file contains everything that there's in a watch face:

- Face type
- Color, styles
- Complications
- ...
 
It also includes default settings for complications (e.g. a selected city for a weather widget etc). When sharing a face you can choose which data is shared.

## Embedded face sharing implementation

1. Generate a watch face with your app complication (your app needs to be live in the store for this)
2. import watch face file and preview into the project (do not put them in an assets catalog)
3. prompt the user to add the face

### Check if a watch is paired

You should offer the user to add a watch face only if the user has a paired watch, you can check so with the following:

```swift
var isPaired: Bool {
    if (WCSession.isSupported()) {
        let session = WCSession.default
        session.delegate = self
        session.activate()
        return session.isPaired
    } else {
        return false
    }
}
```

### Prompt the user to add a face

Use [`addWatchFace(at:completionHandler:)`][addWatchFace(at:completionHandler:)] to prompt the user to add a watch face (api available on both iOS and watchOS):

```swift
private func addFaceWrapper(withName: String) {
    if let watchfaceURL = Bundle.main.url(forResource: withName, withExtension: "watchface") {
        CLKWatchFaceLibrary().addWatchFace(at: watchfaceURL, completionHandler: {
            (error: Error?) in
            if let nsError = error as NSError?, nsError.code == CLKWatchFaceLibrary.ErrorCode.faceNotAvailable.rawValue {
                print(nsError)
            }
            isLoading = false
        })
    }
}
```

## Website distribution

Host and link to the `.watchface` file with MIME type `application/vnd.apple.watchface`.

[addWatchFace(at:completionHandler:)]: https://developer.apple.com/documentation/clockkit/clkwatchfacelibrary/3601124-addwatchface