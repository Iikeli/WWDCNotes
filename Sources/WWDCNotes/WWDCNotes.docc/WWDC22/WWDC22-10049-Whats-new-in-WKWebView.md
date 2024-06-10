# What's new in WKWebView

Explore the latest updates to WKWebView, our framework for incorporating web content into your app’s interface. We’ll show you how to use the JavaScript fullscreen API, explore CSS viewport units, and learn more about find interactions. We’ll also take you through refinements to content blocking controls, embedding encrypted media, and using the Web Inspector.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10049", purpose: link, label: "Watch Video (8 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Web content interaction

Three new ways to interact with the web content in iOS 16:

1. Fullscreen API
2. CSS viewport units
3. Find Interactions

### Fullscreen API

- JavaScript has been able to make HTML elements, like videos or canvas games, full screen in browsers, now they can do that in your apps too

```swift
webView.configuration.preferences.isElementFullscreenEnabled = true

webView.loadHTMLString("""
<script>
  button.addEventListener('click', () => {
    canvas.webkitRequestFullscreen()
  }, false);
</script>
…
""", baseURL:nil)

// 👇🏻 use this if you want to customize the full screen transactions in your app
let observation = webView.observe(\.fullscreenState, options: [.new]) { object, change in
  print("fullscreenState: \(object.fullscreenState)")
}
```

### CSS viewport units

New viewport units support:

- `svw`, `lvw`, `dvw` - smallest/largest/dynamic view port width
- `svh`, `lvh`, `dvh` - smallest/largest/dynamic view port height
- `svb`, `lvb`, `dvb` - block
- `svi`, `lvi`, `dvi` - inline
- `svmin`, `lvmin`, `dvmin` 
- `svmax`, `lvmax`, `dvmax`

```swift
let minimum = UIEdgeInsets(top: 0, left: 0, bottom: 30, right: 0)
let maximum = UIEdgeInsets(top: 0, left: 0, bottom: 200, right: 0)
webView.setMinimumViewportInset(minimum, maximumViewportInset: maximum)
```

### Find interaction

- Find/replace support 

Using `UIFindInteraction` with `WKWebView`:

```swift
webView.findInteractionEnabled = true

if let interaction = webView.findInteraction {
  interaction.presentFindNavigator(showingReplace: false)
}
```

## Content blocking

- new capability to `WKContentRuleList`, the API used to implement content blockers in Safari
- you can run regular expressions on the URL of the current frame

Example: rule to block images only from frames containing Wikipedia:

```swift
let json = """
[{
  "action":{"type":"block"},
  "trigger":{
    "resource-type":["image"],
    "url-filter":".*",
    "if-frame-url":["https?://([^/]*\\\\.)wikipedia.org/"]
  }
}]
"""

WKContentRuleListStore.default().compileContentRuleList(forIdentifier: "example_blocker",
  encodedContentRuleList: json) { list, error in
  guard let list = list else { return }
  let configuration = WKWebViewConfiguration()
  configuration.userContentController.add(list)
}
```

## Encrypted media

- supported on iPadOS 16

## Remote Web Inspector

- now supported