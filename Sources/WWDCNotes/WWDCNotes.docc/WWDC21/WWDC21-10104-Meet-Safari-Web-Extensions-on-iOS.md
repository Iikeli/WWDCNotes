# Meet Safari Web Extensions on iOS

Safari Web Extensions use HTML, CSS, and JavaScript to offer people powerful browser customizations — and you can now create them for every device that supports Safari. Learn how to build a Safari Web Extension that works for all devices, and discover how you can convert an existing extension to Safari through Xcode and the Safari Web Extension Converter.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10104", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## (iOS) Safari Web Extensions

- built with the standard web technologies: HTML, CSS, and JavaScript. 
- uses the same WebExtension API available for all the major desktop browsers
- extensions can be create a new custom Start Page, shown when the user opens a new tab
- once enabled, extensions will show up in Safari actions menu
- extensions need users permissions (like in the desktop)

## Safari Web Extension structure

- `manifest.json`
  - describes the structure of the extension
  - includes the extension's name
  - lists which websites the extension wants to access to
  - lists what features it supports, such as a pop-up page or a New Tab page

- `background.js`
  - Safari runs this script in the background when your extension is enabled
  - allows your extension to listen for various events coming from the browser or other parts of your extension

- `content.js`
  - the browser automatically runs this script on web pages that the user visits
  - this script gives your extension the power to extend and customize pages by directly manipulating them
  - every extensions has one or more `content` scripts
  - the manifest file declares which script runs for which pages

## Creating extensions

- Safari web extensions are hosted in an app, which can be downloaded as any other app in the App Store

### Create a new Safari Web Extension 

- Create a new Xcode project with the `Safari Extension App` template

### Convert an existing Web Extension from another browser 

- use Xcode's [Safari Web Extension Converter][converter], which generates an Xcode project from that existing extension

```shell
xcrun safari-web-extension-converter [options] <path to extension> 
```

### Add iOS support to a macOS Safari Web Extension 

- rebuild the project with Xcode's [Safari Web Extension Converter][converter]:

```shell
xcrun safari-web-extension-converter [options] --rebuild-project <path to extension> 
```

## Debugging techniques

- while our extension is running on an iOS simulator, we can use macOS's Safari.app Web Inspector by going to `Develop > Simulator` and choose the tab we're interested in inspecting
- if you wanted to use macOS's Safari.app Web Inspector with a physical iOS device, enable Web Inspector support on that device in Safari's Advanced Settings
- view manifest errors in iOS's Safari's extension settings
  - these errors details will only appear in Settings for debug builds of your app from Xcode and not for copies from the App Store or TestFlight

## Best practices

- non-persistent background page
  - for `background.js` 
  - persistent is only supported for desktop (no iOS/iPadOS)
  - non-persistent meant that `background.js` is loaded when needed and unloaded when idle for some time
  - to make a background page non-persistent, add `"persistent": false` in your manifest

```json
"background": { "scripts": [ "background.js" ], "persistent": false }
```

- responsive design
  - use a responsive design that can accommodate various screen sizes
  - respect viewport safe area to avoid having content placed under Safari's Tab Bar or the device's home indicator
  - use CSS environment variables to calculate the safe area inset to make sure important elements are positioned within the safe area
  - by using `viewport-fit` parameter in your viewport, you can give your web content an edge-to-edge design that still keeps important content within the safe area
  - pop up pages are shown as popovers on iPadOS, as sheets in iOS
  - test your app with Dynamic Type sizes

- pointer events
  - `element.onmousedown` works only on macOS
  - use `element.onpointerdown` for cross platform support (mouse, touch, or Apple pencil events)

- window APIs
  - windows in iOS are called scenes
  - each Safari scene is represented with two windows: one for regular browsing and one for Private browsing, meaning on iOS `browser.windows.getAll()` gets two objects
  - on iPadOS, if we use two Safari tabs in split view, we will get four windows (and when we create a split view, we will get the `windows.onCreated` event twice (one for regular and one for Private browsing)
  - `windows.create()` `windows.remove()` `windows.update()` are not supported in iOS
  - `windows.onRemoved` is not fired
  - these restrictions are only for windows, Web Extensions still have full control of tabs creation/deletion etc

- feature detection
  - use feature detection to handle the case an API is not available on iOS:

```swift
if (browser.contextMenus) { 
  browser.contextMenus.create({ title: "Options...", ... });
}

if (browser.webRequest) { 
  browser.webRequest.onCompleted.addListener(function(details){ ... });
}
```

## Privacy considerations

- Web Extension permissions
  - Users opt in to your extension per website
  - Safari asks for user consent
  - Permissions is required for any privacy-sensitive API:
    - Tab URLs and titles
    - Cookies
    - Script and stylesheet injection
    - ...

- `activeTab` permission
  - granted when the user invokes your extension
  - Limited to current website in current tab
  - Does not require a prompt
  - add permission to your manifest:

```json
{
  "permissions": [ "activeTab" ]
  ...
}
```

[converter]: https://developer.apple.com/documentation/safariservices/safari_web_extensions/converting_a_web_extension_for_safari
