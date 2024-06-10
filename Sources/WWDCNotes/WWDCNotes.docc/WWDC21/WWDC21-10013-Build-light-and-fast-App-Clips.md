# Build light and fast App Clips

App Clips give people the power to discover and download a small part of your app at a moment’s notice to complete tasks and transactions. Explore tips and best practices to help you create compact App Clips that emphasize modern features and elegant design. Learn how you can build reliable and secure App Clips to ensure that people can always access your experience when scanning a physical App Clip Code or viewing it through your website. And we’ll take you through specific strategies for testing an App Clip before releasing it to the world.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10013", purpose: link, label: "Watch Video (29 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## App Clip size

### How to generate a size report

1. Select the main app scheme
2. Product > Archive
3. Select newly created archive and click `Distribute App`
4. Choose `Development` as method of distribution
5. Select app clip
6. Select `All compatible device variants` in the `App Thinning` menu, make sure that `Rebuild from bitcode` is checked
7. when the archive is done preparing, click `Export`
8. Once exported, open the exported package and open the `App Thinning Size Report.txt`

This report has a different section for each device variant, with the uncompressed size for each variant (should be less 10MB)

You can unpack each `.ipa` and inspect  what's inside to understand why the size might exceed the limit. 

### Basic optimizations

- make sure Xcode will build using the smallest, fastest optimization setting when archiving
- use asset catalog, move assets not used in the App Clip in another catalog
- remove unused code from AppClip target
- remove unused strings from your localization, if needed, make an ad-hoc localization `.strings` file just for AppClip

### Advanced optimizations

- make sure you link only dependencies that are needed by App Clip, use system framework instead of 3rd parties
- optimize media:
  - use lossy compression images, PNG8 for non-photographic material
  - use HEVC for videos
  - compress audio using AAC or MP3 codecs, and experiment with a reduced bit rate
  - use SVG or SF Symbols for glyphs

- image variations: instead of including several variations of an image in your asset catalog, include one base image and build the variations needed at runtime
- lazy loading: fetch images from the web instead of pre-packaging them

## App Clip invocation

Two types of registration in Apple Store Connect

- default - for website and iMessage
- advanced - QR/NFC

Registration takes time to propagate to devices, so changes made in App Store Connect won’t be instantly available.

Prior to displaying UI that surfaces your App Clip, the OS checks to make sure that the invocation domain is verifiably associated with your App Clip:

- the URL viewed in Safari or encoded in a QR code must have a secure association via entitlements and an AASA file with your app and App Clip.
- if this is not configured properly, your clip will not be surfaced.

Make sure the website metdata is similar to:

```html
<meta name="apple-itunes-app" content="app-id=myAppStoreID, app-clip-bundle-id=appClipBundleID, app-clip-display=card">
```

- note that https://example.com is different than https://www.example.com 
- for QR urls, you need to provide a AASA file for the exact domain of the QR code

## Testing

- When building and running in Xcode, set the `_XCAppClipURL` environment variable to an invocation URL of your choice. When you run your clip, the URL will be passed through to your clip just as it would be on a customer’s device.

-  To see your in-development clip surfaced by the OS similar to production devices:  
`Settings.app` > `Developer settings` > `Local experiences` > create a new one
