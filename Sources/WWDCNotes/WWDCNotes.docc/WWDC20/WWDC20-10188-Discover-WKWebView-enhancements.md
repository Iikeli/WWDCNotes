# Discover WKWebView enhancements

WKWebView is the best way to present rich, interactive web content right within your app. Explore new APIs that help you convert apps using WebViews or UIWebViews while adding entirely new capabilities. Learn about better ways to handle JavaScript, fine tune the rendering process, export web content, and more.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10188", purpose: link, label: "Watch Video (30 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## iOS in-app browser choices

In iOS we have mainly two ways: [`SFSafariViewController`][SFSafariViewController] and [`WKWebView`][WKWebView].

### SFSafariViewController

- If we don't need deep customization, [`SFSafariViewController`][SFSafariViewController] is the best choice
- Built on top of [`WKWebView`][WKWebView]
- It handles everything for you
- Provides many Safari features such as:
  - reader
  - content blockers
  - autofill
  - and more

### WKWebView

- More customizable
- Protects your application's code and data from the complexities of the web platform
- The web content runs in a separate process

## What's new in WKWebView

### Isolating your app from web content

- To disable javascript, use the new [`WKWebPagePreferences.allowsContentJavaScript`][allowsContentJavaScript] instead of the deprecated `WKPreferences.javaScriptEnabled`
- [`WKContentWorld`][WKContentWorld] makes sure that your javascript doesn't interfere with the website's javascript (e.g. by declaring the same function) and vice versa:
  -  use [`evaluateJavaScript(_:in:in:completionHandler:)`][evaluateJavaScript(_:in:in:completionHandler:)] and pass either `.defaultClient` (your own default world), `.page` (the webpage's), or `.world(name:)` (for a custom world)
  - You can also inject [`WKScriptMessageHandler`][WKScriptMessageHandler] in different words in order to isolate those as well

### Communicating with JavaScript

- [`callAsyncJavaScript`][callAsyncJavaScript] makes reusing the same script with different values much more like native functions:

```swift
let styleJavaScript = """
  var element = document.getElementById(elementIDToStylize);
  if (!element) 
    return false; 
  for (const theStyle in stylesToApply)
    element.style.theStyle = stylesToApply[theStyle]; 
  return true; 
"""

webView.callAsyncJavaScript(
	styleJavaScript, 
	arguments: [ 
    "elementIDToStylize)": "postContainer", 
    "stylesToApply": [
      "margin": 0, 
      "padding-left": "5px"
    ]
  ], 
  in: .defaultClient, 
  completion: { _ in
   	// check the return value if desired 
  }
)
```

- With `callAsyncJavaScript` serialization and de-serialization of argument types happens automatically
- The `callAsyncJavaScript` completion block is called only after the script code says so: it the script returns a promise, it will call the completion block only after the promise has been fulfilled.

### More flexible rendering

- Use [`WKWebView.pageZoom`][WKWebView.pageZoom] instead of changing the CSS zoom with JavaScript. This will make sure that the zoom is set before the page is rendered.
- Use [`WKWebView.mediaType`][WKWebView.mediaType] to set a custom media type (used in css queries)

### Working with web content

- Use [`find(_:configuration:completionHandler:)`][findString] to offer a native way to find a string in the content similarly to how Safari does
- Use [`createPDF(configuration:completionHandler:)`][createPdf] to share the whole web view similar to how sharing in Safari works
- WKWebView has learned to create Web Archives with [createWebArchiveData(completionHandler:)][createWebArchiveData(completionHandler:)]

### Respecting privacy

- Intelligent Tracking Prevention (ITP) is enabled by default on all `WKWebView`s on iOS 14 and macOS 11


### App-bound domains

- you specify which domains are the core part of the implementation of your app
- deep interaction with the web content not core to your app is disabled, making it impossible for user data to be accidentally compromised by other domains 

Define your domains in the app `info.plist`:

```xml
<plist version="1.0">
<dict> 
<key>WKAppBoundDomains</key>
<array>
	<string>webkittens.internal.apple.com</string>
	<string>pupsonsafari.internal.apple.com</string>
</array>
</dict>
</plist> 
```

[SFSafariViewController]: https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller
[WKWebView]: https://developer.apple.com/documentation/webkit/wkwebview
[allowsContentJavaScript]: https://developer.apple.com/documentation/webkit/wkwebpagepreferences/3552422-allowscontentjavascript
[WKContentWorld]: https://developer.apple.com/documentation/webkit/wkcontentworld
[evaluateJavaScript(_:in:in:completionHandler:)]: https://developer.apple.com/documentation/webkit/wkwebview/3656442-evaluatejavascript
[WKScriptMessageHandler]: https://developer.apple.com/documentation/webkit/wkscriptmessagehandler
[callAsyncJavaScript]: https://developer.apple.com/documentation/webkit/wkwebview/3656441-callasyncjavascript
[WKWebView.pageZoom]: https://developer.apple.com/documentation/webkit/wkwebview/3516411-pagezoom
[WKWebView.mediaType]: https://developer.apple.com/documentation/webkit/wkwebview/3516410-mediatype
[findString]: https://developer.apple.com/documentation/webkit/wkwebview/3650493-find
[createPdf]: https://developer.apple.com/documentation/webkit/wkwebview/3650490-createpdf
[createWebArchiveData(completionHandler:)]: https://developer.apple.com/documentation/webkit/wkwebview/3650491-createwebarchivedata