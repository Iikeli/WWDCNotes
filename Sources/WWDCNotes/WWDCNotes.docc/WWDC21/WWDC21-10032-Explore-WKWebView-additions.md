# Explore WKWebView additions

Explore the latest updates to WKWebView. We’ll show you how to use APIs to manipulate web content without JavaScript, explore delegates that can help with WebRTC and Downloads, and share how you can easily create a richer web experience within your app.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10032", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What's new in `SFSafariViewController`

- New API to bring one of your app extensions to a customized button on `SFSafariViewController`.
- Tapping that button will run your app extensions directly from `SFSafariViewController`'s toolbar, including running JavaScript on the page

```swift
import UIKit
import SafariServices

class ViewController: UIViewController {
  func showSafariViewController (pageURL: URL) {
    let configuration = SFSafariViewController.Configuration()
    configuration.activityButton = SFSafariViewController.ActivityButton(
      templateImage: UlImage(named: "example_image")!,
      extensionIdentifier: "com.example.extension"
    )
    
    let safariViewController = SFSafariViewController(url: pageURL, configuration: configuration)
    present(safariViewController, animated: true)
  }
}
```

> For anything more than this, you will need to use `WKWebView`.

## What's new in `WKWebView`

- new APIs to allow you to easily interact with the content in your web view without having to deal with injecting JavaScript:
  - access theme and related colors for a website
  - manage text interaction
  - control media playback

- new browser-level APIs, previously only available to Safari.app:
  - disable HTTPS upgrade
  - control media capture (getUserMedia)
  - manage downloads

### Access colors

Get `themeColor`:

```swift
class ViewController : UIViewController {
  var headerView: UIView
  var webView: WKWebView
  var observations: [NSKevValueObservation] = []

  override func viewDidLoad() {
    let observation = webView.observe(\.themeColor) { // 👈🏻
      [unowned self] webView, _ in
      self.headerView.backgroundColor=webView.themeColor
    }
    observations.append (observation)
  }
}
```

If `themeColor` isn't set, there's an alternate calculated background color exposed as `underPageBackgroundColor`:

```swift
class ViewController : UIViewController {
  var headerView: UIView
  var webView: WKWebView
  var observations: [NSKeyValueObservation] = []
  
  override func viewDidLoad() {
    let observation = webView.observe(\.underPageBackgroundColor) { // 👈🏻
      [unowned self] webView,_ in
      self.headerView.backgroundColor = webView.underPageBackgroundColor
    }
    observations.append (observation)
  }
}
```

### Text interaction

Disable/enable text interaction:

```swift
override func loadView(){
  let preferences = WKPreferences()
  preferences.textInteractionEnabled = false
  
  let webConfiguration = WKWebViewConfiguration()
  configuration.preferences = preferences

  webView = WKWebView(frame: .zero, configuration: preferences)

  ...
}
```

### Media playback

Pause all media:

```swift
await webView.pauseAllMediaPlayback()
```

Close all media windows:

```swift
await webView.closeAllMediaPresentations()
```

Request media state:

```swift
let playbackState = await webView.requestMediaPlaybackState()
```

Set Media Playback Suspended State

```swift
await webView.setAllMediaPlaybackSuspended(true)
```

### Disable HTTPS upgrade

```swift
override func loadView(){
  let webConfiguration = WKWebViewConfiguration()
  webConfiguration.upgradeknownHostsToHTTPS = false
  webView = WKWebView(frame: .zero, configuration: webConfiguration)
  ...
}
```

### Control media capture (getUserMedia)

- `WKWebView` supports `getUserMedia` from iOS 14.3; which allows WebRTC functions to work inside your app
- From iOS 15, you can load your web content from a custom scheme handler; the user request prompt will show your app as the origin of the request, rather than show a request from the website URL
- New api to prompt user for camera and microphone permissions when working with web content

```swift
// Prompt for microphone access only
class ViewController: UIViewController, WKUIDelegate{
  var webView: WKWebView
  
  override func viewDidLoad() { 
    webView.uiDelegate = self
    webView.load(URLRequest (url: URL(string: "https://recordMyMic.example.org")!))
  }

  func webView(
    _ webView: WKWebView,
    decideMediaCapturePermissionFor origin: WKSecurityOrigin,
    initiatedByFrame frame: WKFrameInfo,
    type: WKMediaCaptureType
  ) async -> WKPermissionDecision {
    return type == .microphone? .prompt: .deny
  }
}
```

### Manage downloads

Three ways to initiate a download from the web:

1. The Web Content can initiate a download via Javascript:

```js
let a = document.createElement('a'):
let b = new Blob([1,2,3]);
a.href = URL.createObjectURL (b);
a.download = "filename.dat":
a.click();
```

2. The server can initiate a download by responding like this, after calling a `loadRequest` on your web view

```js
HTTP/1.1 200 OK
Content-Disposition: attachment; filename="filename.dat"
Content-Length: 100
```

3. App via new API

```swift
class ViewController : UIViewController, WKDownloadDelegate {
  @IBOutlet var textField: UITextField!
  @IBOutlet var webView: WKWebView!
  
  @IBAction func downloadFile() {
    guard let url = URL(string: textField.text ??"') else { return }
    async {
      let download = await webView.startDownload(using: URLRequest (url: url))
      download.delegate = self
    }
  }
```

- Whatever method you use, when you get the `WKDownload` object, you need to set the `delegate` property on that object to be able to tell it where to write the bytes to disk. If you do not, the download will automatically be cancelled

- If a download fails, the data to resume the download will be handed to you if you implement the [`download(_:didFailWithError:resumeData:)`][download(_:didFailWithError:resumeData:)] method on the delegate

[download(_:didFailWithError:resumeData:)]:  https://developer.apple.com/documentation/webkit/wkdownloaddelegate/3727345-download
