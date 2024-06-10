# Create quick interactions with Shortcuts on watchOS

Shortcuts are a natural fit on Apple Watch, allowing people to get things done with just a tap — even from a complication. Bring your app’s intents to the wrist: We’ll help you optimize your shortcuts performance, understand how intents can be routed from watchOS to iOS, explore the latest interaction and presentation interfaces, and examine how the Shortcuts app manages shortcuts and intents for Apple Watch.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10190", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What's new in Shortcuts on Apple Watch

- New Shortcuts.app
- on iPhone you can maintain a Apple Watch collection of all the shortcuts that should sync to the Watch
- New Shortcuts complications
  - to launch the Shortcuts.app
  - to trigger a specific shortcut

## Running Shortcuts on the Watch

For shortcuts based on `NSUserActivity` API, we need to open the app to handle the activity: 

- if the app is found on the watch, the app is launched
- if the app is not found, an error will be shown

For shortcuts based on `Intent` API, we need to call the Intent extension:

- if the extension is found on the watch, the extension will be asked to handle the intent
- if not, the watch will try to handle the intent _remotely_, aka in the phone:
  - if the intent can be handled without launching the app, it will do so
  - if the intent requires to be handled in the app, an error will be shown