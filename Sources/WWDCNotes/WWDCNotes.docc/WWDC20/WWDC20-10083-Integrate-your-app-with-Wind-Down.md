# Integrate your app with Wind Down

Discover how you can help people get ready for a good night's sleep by surfacing your app's actions for Wind Down Shortcuts, part of the new Sleep experience. Learn more about how Wind Down works. Find out how you can build intents that expose features in your app like guided meditations, soothing audio stories, or many other categories. And explore how you can surface those features on someone's device before bedtime.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10083", purpose: link, label: "Watch Video (10 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

Wind Down helps users meet their sleep goals, during Wind Down apps can offer shortcuts that:

- focus on the things that will let the user relax or give them some space and mindfulness after a long day. 
- get a heads-up on what is most important for tomorrow before going to bed.

During Wind Down mode, the lock screen will show a shortcuts button to show the user favorite/suggested Wind Down shortcuts. 

To create such shortcuts, either: 

- go to the Health app's new sleep setup flow.
- add any existing shortcut (from the Shortcuts.app) by going to the shortcut details view and toggle `Show in Sleep Mode`.

All these shortcuts will be available in the Shortcuts.app under the `Sleep Mode` category (this category will show up only after some shortcuts have been added to it).

## Integrate Apps actions

### Shortcut Availability

Apps can feature shortcuts in the Wind Down setup flow so they can be run with just a few taps from the lock screen. 

The main way any app can expose its Wind Down actions is through intents and user activities:  
from iOS 14 there's a new [INIntent][INIntent] (and [`NSUserActivity`][NSUserActivity]) [`shortcutAvailability`][shortcutAvailability] property which you can use to tell the system which of your app's actions should appear in the Wind Down setup.

Here are all the possible [availability options][INShortcutAvailabilityOptions]:

- Journaling
- Mindfulness
- Music
- Podcasts
- Prepare for Tomorrow
- Reading
- Yoga and Stretching

### Suggest or donate apps actions

There are two things you can do to enable the system to suggest your app's actions during Wind Down setup:

- Suggest shortcuts to the system that you want to feature by calling [`setShortcutSuggestions`][setShortcutSuggestions] on [`INVoiceShortcutCenter`][INVoiceShortcutCenter].

```swift
// Suggest intent

import Intents

let playSoundIntent = INPlayMediaIntent()

playSoundIntent.shortcutAvailability = .sleepMusic
playSoundIntent.suggestedInvocationPhrase = "Play sleeping songs"

let shortcut = INIShortcut(intent: playSoundIntent)
INVoiceShortcutCenter.shared.setShortcutSuggestions([shortcut])
```

- Donate the intents/activities the user does in the app

```swift
// Donating an intent

import Intents 

let playSoundIntent = INPlayMediaIntent() 

playSoundIntent.shortcutAvailability = .sleepMusic playSoundIntent.suggestedInvocationPhrase= "Play Counting Sleepy Dinosaurs" 

let interaction = INInteraction (intent: playSoundIntent, response: nil) 
interaction. donate { error 
  // Handle the error
}
```

```swift
// Donating an user activity

import Intents 
import UIKit 

let userActivity = NSUserActivity(activityType: "your.app.domain.id.playSound" 

userActivity.isEligibleForSearch = true
userActivity.isEligibleForPrediction = true

userActivity.title = "Play Running Water"
userActivity.suggestedInvocationPhrase = "Play Running Water"
userActivity.shortcutAvailability = .sleepMusic 

viewController.userActivity = userActivity 
```

[INIntent]: https://developer.apple.com/documentation/sirikit/inintent
[shortcutAvailability]: https://developer.apple.com/documentation/sirikit/inintent/3552187-shortcutavailability
[NSUserActivity]: https://developer.apple.com/documentation/foundation/nsuseractivity
[INShortcutAvailabilityOptions]: https://developer.apple.com/documentation/sirikit/inshortcutavailabilityoptions
[setShortcutSuggestions]: https://developer.apple.com/documentation/sirikit/invoiceshortcutcenter/2994364-setshortcutsuggestions
[INVoiceShortcutCenter]: https://developer.apple.com/documentation/sirikit/invoiceshortcutcenter
