# Meet Safari Web Extensions

When you create a Safari Web Extension, you can help people get common online tasks done more quickly and efficiently. We’ll show you how to build a new Safari Web Extension and host it on the App Store, as well as how to use the safari-web-extension-converter tool to migrate existing extensions from other web browsers like Chrome, Firefox, or Edge with very little effort.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10665", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



Existing extension ecosystem:

- content blockers (iOS & macOS)
- share extensions - can run JS on the currently opened web page and return data to the extension
- Safari app extensions on macOS

If you’re a web developer and don’t want to learn Swift to build an extension, or you have an existing extension for Chrome/Firefox/etc., you can now use the new Safari Web Extensions API.

Safari Web Extensions:

- extensions built primarily using HTML, JS and CSS, like legacy Safari extensions
- API compatible with other browsers (the [WebExtensions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions) standard)
- improved user privacy controls
- extensions are sold through the App Store
- some WebExtensions APIs are missing, so provide feedback if you want something added

Like other extensions, Web Extensions must be packaged inside a native Mac app. Xcode 12 is required to build them.

A command-line tool is provided which wraps an exising web extension (e.g. for Chrome/Firefox) into a new app:

```
xcrun safari-web-extension-converter […] /path
```

- lets you know if any features are not available
- the largest icon in the manifest is used as the app icon (it’s recommended to include 512×512 and 1024×1024 icons)

To create a new extension from scratch, create a “Safari Extension App” project or add a “Safari Extension” target, and choose Type = Safari Web Extension.

## Extension privacy

- If your extension needs access to specific sites, the user will be asked for permission to run it on that site for one day or always
- Optional permissions: you can include the URL pattern under `optional_permissions` key and then ask for access using `browser.permissions.request(…)` at the moment when you require access
- The Safari preferences window page of your extension shows information about what kind of access was granted to the extension
- It’s best to use the `activeTab` permission, which grants access to the currently open page when the user interacts with your extension in some way
- The hostname of the extension changes every time Safari is launched in order to prevent fingerprinting
  - use `browser.runtime.getURL("/path/to/resource")` to create URLs to assets

## Debugging

- Access the background page through the Develop menu
- Content scripts are visible in the Sources tab for the page
  - to run JS in the console in the context of a content script, choose the script from the pulldown menu in the corner

- Don’t rely on code being executed when the page loads, since the extension may not have permission to run yet at this point

## Communicating between components

Content script ⭢ background page:

```js
browser.runtime.sendMessage()
browser.runtime.onMessage.addListener()
```

Background page ⭢ extension:

```js
browser.runtime.sendNativeMessage()
```

The message is handled by the [`SafariWebExtensionHandler.beginRequest(with context:)`](https://developer.apple.com/documentation/foundation/nsextensionrequesthandling/1413395-beginrequest) delegate method (requires the `nativeMessaging` permission).

Extension ⭢ background page: 

- Use completion handler from [`NSExtensionContext`](https://developer.apple.com/documentation/foundation/nsextensioncontext) object in [`beginRequest(with context:)`](https://developer.apple.com/documentation/foundation/nsextensionrequesthandling/1413395-beginrequest) to send back a response

App ⭢ background page:

- [`SFSafariApplication.dispatchMessage(...)`](https://developer.apple.com/documentation/safariservices/sfsafariapplication/2823941-dispatchmessage) (check that the extension is turned on first)

App ⭤ extension:

Shared [`NSUserDefaults`](https://developer.apple.com/documentation/foundation/nsuserdefaults) from an app group, or [`NSXPCConnection`](https://developer.apple.com/documentation/foundation/nsxpcconnection)
