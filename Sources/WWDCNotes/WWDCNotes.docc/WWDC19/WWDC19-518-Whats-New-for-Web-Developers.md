# What's New for Web Developers

WebKit provides a rich set of classes designed to load, display, and manage web content in your app. Discover how to integrate your web content into powerful platform features including Dark Mode, new presentation features in Share Sheet, JavaScript payment APIs for Apple Pay, and more.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/518", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



## Dark Mode

  - Add `:root { color-scheme: light dark; ... }` to tell WebKit that the entire page supports dark mode
  - Adapting custom CSS styles with `@media (prefers-color-scheme: dark) { ... }`
  - Adapting images using `<picture><source srcset="dark.jpg" media="(prefers-color-scheme: dark)"> <img src="light.jpg"></picture>`
  - Adapting dynamic content via the JS **match media API**

## WebShare

  - Native share sheet from within web content
  - Uses all the improvements made in iOS 13 to the native share sheet
  - Add `await navigator.share({title: "", text: "", url: ""})` to e.g. a buttons event listener

## New Media Features

  - Checking the device's capabilities using the new **Media Capability API**
  - Support for **Alpha Channel Video** (ACV) in iOS 13 and macOS Catalina
    - Make sure that ACV is available on underlying device by adding an alpha channel key to capability object: `alphaChannel: true`
    - Check via `if(capabilities.supported && capabilities.supportedConfiguration.video.alphaChannel) {} else {}`

## WebRTC Screen Capture

  - Prompt the user for permission via `navigator.mediaDevices.getDisplayMedia({ video: true }).then((stream) => {...}`
  - `stream` contains capture of the space containing the Safari window
  - Can be used to share your screen

## Tools for Improving Performance

  - Web Inspector adds a new **CPU Timeline** packed with actionable information
  - There is a post on the WebKit blog and a WWDC session video called [Understanding CPU Usage with Web Inspector][wwdc19513]
  - WebDriver on iOS for automation and regression testing

## Storing Structured Data

  - Added IndexedDB and completely removed WebSQL in Safari 13
  - All modern browsers support `IndexedDB`
  - Transition as asap

## Apple Pay

- Fully supported using the **Payment Request API**
- Use Apple Pay in `WKWebView` on iOS 13
  - Denied if script has ever been injected
  - Future script injections denied if Apple Pay ever requested
  - Restrictions are reset during navigation

- `canMakePayments` must be called before showing an Apple Pay button or doing anything using Apple Pay
- Prefer WebKit APIs instead of using JS when possible

[wwdc19513]: https://developer.apple.com/wwdc19/513