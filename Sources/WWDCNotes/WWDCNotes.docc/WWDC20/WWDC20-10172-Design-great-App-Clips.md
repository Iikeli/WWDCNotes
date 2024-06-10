# Design great App Clips

App Clips offer fast, convenient ways for people to perform everyday tasks without needing to download or navigate your full app. We'll show you how to identify key elements from your iOS app that make up a great App Clip, design a smooth flow, work with notifications, and provide messaging guidance when encouraging people to download your full app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10172", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What are App Clips?

- Lightweight versions of our main apps
- They only offer _some_ of our app functionality
- They offer said functionality only when and where the user needs it
- Fully native
- Instant
- Meant for people that have not downloaded our main app
- Apps don't live on the device, they will be removed for the user by the system when they're deemed as no longer needed

## Discovering App Clips

## Found in physical places tags

The new App Clip code comes with both an NFC tag, right in the middle, useful when the user can come up close to it, and a visual, QR-like, code on the outside around it, which can be useful if the user can't get too close, maybe because the code is on a billboard or similar.

![][clipCodeImage]

### Found also in built-in apps 

#### Maps

You can also attach app clips to locations. This way your app clip might show up in maps which lets people use the app clip before they actually get to the place itself. 

![][mapsClipImage]

If the app becomes frequently used in a specific location, it will even show up in Siri Suggestions:

![][siriSuggestionsImage]

#### Safari

Like previously with apps, we can now show a smart app banner with the App Clip instead of the main app:

![][safariClipImage]

#### Messages

If you have a webpage with a smart app banner, and a user share a link to that page in messages, people can go directly from messages right into the app clip:

![][messagesClipImage]

## Designing for App Clips

### Icon

The main App Clip icon is generated automatically for you:

![][iconDiffImage]

If your app creates App Clips for other physical businesses, you can have custom icons for each clip:

![][businessIconImage]

For more, refer to the [`Create app clips for other businesses`][20-10118] session

### Concepts

- App Clips are simple 
- App Clips they're built to let the user accomplish a task
- App Clips are fast

### Approach to App Clips

1. **Purpose**: think what your App Clip is for, define what is the single task that the App Clip accomplishes
2. **Minimum Flow**: think what is the minimum flow to accomplish that task, remove  everything else
3. **Offer the full app**: once the task is complete, you can offer the user to download the full app

## Tips

- Narrow down you full app into a single feature
- avoid onboardings, introductions, etc
- avoid using log-ins
- avoid tabs or other kind of global navigation
- no settings

## Notifications

- By default App Clips can send notifications up to 8 hours since their launch.
- If the App Clip request for notifications permission, this time is extended up to one week.
- Keep notifications on the specific App Clip task
- Do not send unsolicited notifications

For more information, see the [App Clips's Human Interface Guidelines][hig]


[20-10118]: ../10118
[hig]: https://developer.apple.com/design/human-interface-guidelines/app-clips/overview/

[iconDiffImage]: WWDC20-10172-iconDiff
[businessIconImage]: WWDC20-10172-businessIcon
[clipCodeImage]: WWDC20-10172-clipCode
[mapsClipImage]: WWDC20-10172-mapsClip
[siriSuggestionsImage]: WWDC20-10172-siriSuggestions
[safariClipImage]: WWDC20-10172-safariClip
[messagesClipImage]: WWDC20-10172-messagesClip