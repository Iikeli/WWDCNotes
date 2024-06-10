# What's new in App Clips

Discover how App Clips can elevate quick and focused experiences for specific tasks, the moment your customer needs them. We’ll take you through some of the latest improvements to App Clips, including launching an experience directly from an app, testing your App Clip locally, and creating App Clip Codes to make it easy to access your experience in the real world. We’ll also share some great examples of App Clips from our developer community that provide innovative ways to interact with people and beautiful designs.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10012", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- new in iOS 15: Businesses app clips will show up in Spotlight

## App Clip card in Safari

![][appClipSafari]

- in iOS 14, we could use the `apple-itunes-app` meta tag to display the smart app banner in Safari or Safari View Controller
- new in iOS 15 we can configure the meta tag to show a full-sized App Clip Card
- if the user chooses `View in Safari`, Safari will remember the choice and not show the card next time the page is loaded again
- add `app-clip-display=card` in your meta tag:

```html
<head>
<meta name="apple-itunes-app"
      content="app-clip-bundle-id-com.example.fruta.Clip, app-id-123456789, app-clip-display=card"> 
</head> 
```

- works from another app's Safari View Controller too

## Test with local experiences

- you can test the whole app clip experience
- you can launch it with most of regular App Clip invocation methods.
- to create one local experience, go to `Developer Settings > Local Experiences`
  - Local experiences support QR, NFC, App Clip Code, Safari, and Messages
  - They don't show up in Maps, location suggestions, and Spotlight search
  - you can only specify local experiences for Xcode-installed App Clips or App Clips in Beta testing

## App Clip Code

- Apple-designed visual code
- available from iOS 14.3
- an App Clip code always leads to an App Clip
- each code encodes an unique URL
- two types: NFC-integrated or Scan-only
- can be used as an AR anchor
- you can create a code via App Store Connect or via the [App Clip Code Generator][resources] cli tool
  - See all available templates at `/Library/Developer/AppClipCodeGenerator/SampleTemplates`
  - generator command example: 

```shell
$ AppClipCodeGenerator generate --type nfc --url https://fruta.example --index 4 --output fruta.svg`
```

### Best practices

- keep the code on a flat surface
- the code should be placed at upright position (no sideways etc)
- make the code at least one-inch wide
- ensure good visibility: the code shouldn't be blocked, damaged, or placed too close to other codes

[appClipSafari]: WWDC21-10012-appClipSafari
[resources]: https://developer.apple.com/app-clips/resources/
