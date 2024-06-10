# What’s new in web apps

Discover web apps for Mac — a powerful way to experience your website from the Dock. Learn how you can customize your web app to give people the best experience when they add your site. We’ll also share how to take advantage of push notifications and badging for web apps for Mac and Home Screen web apps for iOS and iPadOS.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10120", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(rogerluan)
   }
}



- macOS 14 introduces support to web apps, similar to what was already possible with prior versions of iOS/iPadOS, but better.
  - Different from iOS/iPadOS, web apps on macOS look and feel like a standalone app, whereas on iOS/iPadOS they are by default merely a shortcut that opens up the default browser (e.g. Safari), functioning more as a homescreen bookmark than anything else.
  - With iOS/iPadOS 17, you can now customize the disply mode of your web app to make it look and feel more like a standalone app, and not just a shortcut to the default browser.

- When creating/saving a new web app, you can customize its app icon and name, and it gets saved in your dock.
- Safari copies the cookies so your session is saved. This means that if you're logged in to a website in Safari, you'll be logged in to the same website in the web app too.
  - Note that only cookies are copied, and not any other type of local storage.

- All websites support web apps out of the box without any specific implementation:
  - There's no Safari toolbar, or tabs, etc, so the view is cleaner.
  - Supports auto-filling passwords from iCloud Keychain or other 3rd party password managers.
  - Privacy-minded prompts for location, camera, microphone, etc. work the same way as with native macOS apps.
  - Supports web notifications, with app icon, sounds, badges, Focus mode.

- Developers can further customize the behavior of web apps to offer a better experience to users:
  - "Standalone Display Mode"
    - macOS: it hides the default navigation toolbar.
    - iOS/iPadOS: it launches as a real "app", and not in the default browser anymore, hiding all the browser-related UI, and with separate cookies and storage from the browser.
  - You can customize web links behavior within your web app by defining different link scopes. The default scope is your web app's host, so that your own links will open within your app, but other external links should open in the default browser.
    - If you need to bypass the scope for whatever reason, use `window.open`, which will always bypass the web app's link scope and open the link within the web app.

## App Manifest

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Browser Pets | All Pets Welcome‹/title>
    <link rel="manifest" href="manifest.json">
</head>

<body>
    <h1>Browser Pets</h1>
</body>
</html>
```

```js
{
    "name": "Browser Pets", // Optional, but recommended
    "display": "standalone", // Optional, but recommended
    "start_url": "https://example.com", // The domain of your web app
    "scope": "/browserpets/", // Optional, defines which links should open within your web app
    "id": "forums" // Used for Focus, optional. Defaults to start_url
}
```

## Web Notifications

- Web Push standards. If you already have a web push implementation, it should work out of the box.
- Sounds:
  - macOS: off by default.
  - iOS/iPadOS: on by default.
  - Override default values by passing `silent: true/false` in your notification payload.
- Managed through system preferences like any native macOS and iOS/iPadOS app.
- Supports badges.
- Supports Focus.

## Web Kit New APIs

- User Activation API
- Fullscreen API
- Screen orientation API
- See also: https://webkit.org/blog/13966/webkit-features-in-safari-16-4/

## Best Practices

- Use solely cookies for authentication, and not any other type of local storage, for a seamless experience. Only cookies are copied over to the web app from the browser when the web app is created.
- Ensure login flow stays within the app scope, so that the user doesn't get redirected to the browser.
- OAuth implementation should work within the web app too. If this doesn't work, please submit a bug report to Apple.
- Provide one-time passcodes for 2FA instead of "magic links". Magic links won't launch the web app, they will launch within the browser.

# Related Sessions

- [Meet passkeys - WWDC22](https://wwdcnotes.com/notes/wwdc22/10092/)
- [Meet Web Push for Safari - WWDC22](https://developer.apple.com/videos/play/wwdc2022/10098/)
- [Rediscover Safari developer features - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10262)
- [What's new in Web Inspector - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10118)
