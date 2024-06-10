# What's new in SiriKit and Shortcuts

Get a quick overview of everything new in Siri and Shortcuts to help people get more out of your app: We’ll demonstrate how you can design visually rich conversations, feel at home with the operating system by designing for the new compact Siri UI, and provide an overview of all the ways we’ve made it even easier for people to organize and set up actions from your apps.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10068", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Design

- in iOS 14 Siri comes with a notification-like, compact design
- The new design focuses only on the most essential information, minimizing disruption

- The same design also comes to Shortcuts.app
- in iOS 13 you may have needed to jump into the Shortcuts.app to complete a task: with iOS 14 shortcuts will run seamlessly in the background, and will only prompt you if the shortcut needs your input

## Shortcuts: intents principles

### Disambiguation

a.k.a. when you're asking follow-up questions to clarify the user's intent

- With conversational shortcuts, you have the opportunity to ask follow-up questions, with iOS 14 there are new APIs for customizing how the list is presented:
![][listImage]

### Intents UI

a.k.a. how your app UI shows in Siri and Shortcuts.

- Your Intents UI now appears in this new compact UI, whether you're running from Siri or Shortcuts:
![][intentsImage]

- Intents UIs now use a consistent background material whether they're run from Siri or from Shortcuts:
 - take advantage of the new material by keeping the background color of your Intents UI view transparent.

- focus on the essentials:
  - Because the compact UI appears on top of whatever the user is doing, the less vertical space your Intents UI takes up, the more lightweight the experience will feel.

## Shortcuts Folders

- users can now create folders for their shortcuts
- there are smart folders to identify which shortcuts appear in the share sheet or on the watch.

## Shortcuts on Apple Watch

![][watchImage]

- shortctus can now run on the watch with the new shortcuts.app on watchOS.
- shortcuts can be set as complications and run them right from the watch face

## Shortcuts.app automation

- New Automation suggestion in the Shortcuts.app gallery

### New triggers

- when receiving an email
- when receiving an message
- when closing an app
- when the battery hits a certain level
- when the phone starts charging

### More automatic triggers (without asking first)

- time of day
- alarm
- sleep
- workout
- CarPlay
- Airplane Mode
- NFC
- DND
- Low Power Mode
- Open app
- Close app
- Battery level

### Wind Down integration

If your app has sleep-friendly shortcuts, use the new [`INShortcutAvailabilityOptions`][INShortcutAvailabilityOptions] API to.

[INShortcutAvailabilityOptions]: https://developer.apple.com/documentation/sirikit/inshortcutavailabilityoptions

[listImage]: WWDC20-10068-list
[intentsImage]: WWDC20-10068-intents
[foldersImage]: WWDC20-10068-folders
[watchImage]: WWDC20-10068-watch