# Design App Shortcuts

Learn how you can surface great features from your app directly in Siri, Spotlight, and the Shortcuts app. We'll introduce you to App Shortcuts, provide best practices to help you evaluate features in your app that would work well as App Shortcuts, and take you through the process of creating one of your own. Learn how to create clear and memorable names, design custom visuals, collect required information, and create discoverable shortcuts

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10169", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

- Having habitual tasks completed outside their respective apps, and making them available via Siri and Spotlight gives people flexibility in how they interact with their device and how they get things done
- Shortcuts are how you can offer people that same flexibility to accomplish key tasks from your app throughout the OS
- all shortcuts begin with a fundamental component called <kbd>actions</kbd>
  - represents an individual task that people can complete with your app
  - e.g., as creating a reminder or sending a message

- Actions can be used in two ways:
  - custom shortcuts - which people can create using one or more actions from apps
  - app shortcuts - shortcuts created by app developers containing one action
    - before iOS 16, users needed to find and tap the <kbd>Add to Siri</kbd> button to enable each new app shortcut
    - from iOS 16, the App Shortcuts you create for your app will be automatically available as soon as your app is installed. 

## Identify a feature

Focus on tasks that are:

- self-contained - which they can be completed outside your app
- straightforward - efficient to get through

- note that you can create maximum 10 shortcuts

## Pick a name 

- keep it simple/brief
- make it memorable
- it should clearly communicate its function
- (requirement) must include your app name
  - the app name in this phrase can be your official app name, or any of the alternative names you submitted to the App Store

- provide thoughtful natural language variations to ensure it works for whatever similar phrases people might say
  - ...for every language your app support

- you can use a dynamic parameter directly in the shortcut name (e.g., <kbd>Start X Meditation</kbd>)
  - up to one dynamic parameter
  - its value must be within a provided, finite list
  - this list can be updated any time the app is open

## Refine your visual

> snippets = notification-like interface where your app can show something to the user when using your shortcut

- [snippets][sni] use a semitranslucent material
- do not fill the visual with an opaque background
- use vibrant label colors on text

- Two ways to show results in iOS 16: 
  - Live Activities
  - Custom Snippets (now supported in Apple Watch)

- snippet key elements:
  - supporting dialog - this is what Siri speaks and is intended to accompany your custom visual
    - if the dialog is redundant, you can suppress it in code
  - custom visual - your snippet UI

- snippets will be used in other devices
  - for audio only devices (AirPods, etc) the full dialog will be read
  - Be sure to provide both types of dialog, so that people have access to all the information they need regardless of which device they choose to interact with your shortcut

## Ask for information

- <kbd>Parameter Confirmation</kbd> - when possible, try to make a meaningful assumption and present it as an option for people to confirm
- <kbd>Disambiguation</kbd> - you provide a short list (five or less, remember that these will be read on voice-only devices), helps people that are unfamiliar with the shortcut learn the possible values
- <kbd>Open-ended request</kbd> - when your app needs an open-ended value that isn't compatible with a short list, like a number, a location, or a string
  - Supported types:
    - Decimal
    - Person
    - Location
    - URL
    - Integer
    - File
    - Payment Method
    - Rich Text
    - Boolean
    - Measurement
    - Enumeration
    - String
    - Date
    - Duration
    - [App Entity][app-entities]
    - Currency

### Ask for confirmation - Intent confirmation

- if you have all the information you require, you may still want to do one final confirmation before proceeding with your shortcut
- only use this step for consequential actions (e.g., financial transaction, destructive actions like deleting content, or an action that may just feel high risk, like sending a calendar invite to a big group)
- these ask for confirmations will always have a pair of buttons in the snippet, offering to either proceed with or cancel the proposed action
  - the confirm button should contain a verb reiterating the action that is about to be taken and not ambiguous words like "confirm." (e.g., <kbd>order</kbd>, <kbd>buy</kbd>)
  - App Intents framework provides a set of default verbs with corresponding synonyms, you can also create a custom verb (and define its synonyms)

## Make it discoverable

- in place of the old <kbd>Add to Siri</kbd> button is a new [`SiriTipView`][SiriTipView] (UIkit: [`SiriTipUIView`][SiriTipUIView]) for you to feature in your app
- new [`ShortcutsLink`][ShortcutsLink] button that you can place in your app to link to the Shortcuts app directly to the landing page that lists all your app shortcuts

[sni]: https://developer.apple.com/documentation/sirikit/inuihostedviewcontext/sirisnippet
[app-entities]: https://developer.apple.com/documentation/appintents/app-entities
[tip]: https://developer.apple.com/documentation/appintents/siritipview
[SiriTipUIView]: https://developer.apple.com/documentation/appintents/siritipuiview
[ShortcutsLink]: https://developer.apple.com/documentation/appintents/shortcutslink