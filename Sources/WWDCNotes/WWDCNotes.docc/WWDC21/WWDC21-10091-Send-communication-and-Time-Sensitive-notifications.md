# Send communication and Time Sensitive notifications

Learn more about the evolution of notifications on Apple platforms. We’ll explore how you can help people manage notifications within your app, including how you can craft meaningful moments with interruption levels and Time Sensitive notifications. And we’ll introduce you to communication notifications, providing a richer experience for calls and messages in your app through SiriKit.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10091", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Visual updates

- new look with greater focus on the content and the associated media along with the application icon
- you can now associate notification actions with icons
  - pass a [`UNNotificationActionIcon`][UNNotificationActionIcon] instance into the [`UNNotificationAction`][UNNotificationAction] icon parameter
  - accepts both system or template images

```swift
// Setting up notification actions with icons

let likeActionIcon = UNNotificationActionIcon(systemImageName: "hand.thumbsup") // 👍🏻
let likeAction = UNNotificationAction(identifier: "like-action",
                                           title: "Like",
                                         options: [],
                                            icon: likeActionIcon)
        
let commentActionIcon = UNNotificationActionIcon(templateImageName: "text.bubble") // 💬
let commentAction = UNTextInputNotificationAction(identifier: "comment-action",
                                                       title: "Comment",
                                                     options: [],
                                                        icon: commentActionIcon,
                                        textInputButtonTitle: "Post",
                                        textInputPlaceholder: "Type here…")

let category = UNNotificationCategory(identifier: "update-actions",
                                         actions: [likeAction, commentAction],
                               intentIdentifiers: [], options: [])
```

## Notification management

- Notification summary
  - notifications can now be delivered at scheduled times in the day as a summary
  - reduces the number of active interruptions from incoming notifications and presents them collectively at preset times
  - include media attachments with the notification content so there is a better chance for that notification to be featured at the top of the notification summary
  - adopt the new [`UNNotificationContent`][UNNotificationContent] [`relevanceScore`][relevanceScore] to provide a relevance score for the notification content so the right notifications from the application get featured in the notification summary as well

- Focus
  - in iOS 15, devices can be set in a particular Focus based on the activity or time of day such as Reading, Sleep, or Work
  - when active, the device will filter the presentation and interruption of notifications
  - Focus configuration allows selecting people and applications that can send interruptive notifications
  - it's possible to break through these management controls

## Interruptions

New API for interruption levels as part of the `UserNotifications` framework

Interruption levels ([`UNNotificationInterruptionLevel`][UNNotificationInterruptionLevel])
- Passive (NEW)
  - delivered silently
  - available when the user will view the notification list
  - to be used when the notification does not require immediate attention but should be seen eventually

- Active (default level)
  - do not interrupt if configured not to

- Time Sensitive (NEW)
  - allowed to breakthrough, if it has been allowed
  - to be used when it is relevant to actively interrupt, requiring immediate attention

- Critical
  - requires approved entitlement

|  | sound or vibration | screen light up | system breakthrough^ | bypass ringer switch |
| --- | --- | --- | --- | --- |
| Passive | ❌ | ❌ | ❌ | ❌ |
| Active | ✅ | ✅ | ❌ | ❌ |
| Time Sensitive | ✅ | ✅ | ✅ | ❌ |
| Critical | ✅ | ✅ | ✅ | ✅ | 

> ^bypass notification summary and Focus

Set the interruption level while configuring the content object for the notification request

```swift
// Interruption levels
// Local notification

import UserNotifications

let content = UNMutableNotificationContent()
content.title = "Passive"
content.body = "I’m a passive notification, so I won’t interrupt you."
content.interruptionLevel = .passive

let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)

let request = UNNotificationRequest(identifier: "passive-request-example",
                                       content: content,
                                       trigger: trigger)
```

For push notifications, provide a new key-value pair, with the key interruption level:

```json
{
  "aps" : {
    "alert" : {
      "title" : "Passive",
      "body" : "I’m a passive notification, so I won’t interrupt you."
    },
    "interruption-level" : "passive"
  }
}
```

### Announce

- Siri can announce notifications if there are compatible devices (AirPods) connected
- in iOS 14 it was available via [`UNNotificationCategoryOptions.allowAnnouncement`][UNNotificationCategoryOptions.allowAnnouncement]
- in iOS 15, any notification is automatically supported
- communication and time sensitive notifications will be announced by default
- communication notifications will be announced by default on CarPlay

## Time Sensitive

How to configure:

- enable the associated capability via Xcode for your application
- set the Time Sensitive interruption level on the notification request being posted

Time Sensitive local notification example:

```swift
let content = UNMutableNotificationContent()
content.title = "Urgent"
content.body = "Your account requires attention."
content.interruptionLevel = .timeSensitive

let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 0, repeats: false)

let request = UNNotificationRequest(identifier: "time-sensitive—example",
                                       content: content,
                                       trigger: trigger)
```

Push notification payload example:

```json
{
  "aps" : {
    "alert" : {
      "title" : "Urgent",
      "body" : "Your account requires attention."
    },
    "interruption-level" : "time-sensitive"
  }
}
```

## Communication

- new API that allows your applications to signal what notifications are communications, and the people associated with those communications
- Siri will announce the contents of communication notifications on supported devices including HomePod, AirPods, and CarPlay
- Siri will provide suggestions to help prioritize these communication notifications
- People in your apps will get associated with people in Contacts.app
  - These associations are shown as suggestions on notifications.
  - Once a user confirms a suggestion, Siri Shortcuts are available for tasks with those people in your app
  - Siri will suggest relevant people to break through in the Focus configuration -- including those people associated with communications in your app

- needs to integrate with Siri intents ([`INStartCallIntent`][INStartCallIntent], [`INSendMessageIntent`][INSendMessageIntent])

- you need to update a NotificationContent object with a SiriKit intent in your app's NotificationServiceExtension
  - SiriKit intents are local to the device, and thus must be serviced locally
  - you can also do this in your main app process for local notifications

Example for message-related notification:

```swift 
// Create a messaging notification
// In UNNotificationServiceExtension subclass

func didReceive(_ request: UNNotificationRequest,
                withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    let incomingMessageIntent: INSendMessageIntent = // ...
    let interaction = INInteraction(intent: incomingMessageIntent, response: nil)
    interaction.direction = .incoming
    interaction.donate(completion: nil)
    do {
        let messageContent = try request.content.updating(from: incomingMessageIntent)
        contentHandler(messageContent)
    } catch {
       // Handle error
    }
}
```

Example for call-related notification:

```swift
// Create a call notification
// In UNNotificationServiceExtension subclass

func didReceive(_ request: UNNotificationRequest,
                withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    let incomingCallIntent: INStartCallIntent = // ...
    let interaction = INInteraction(intent: incomingCallIntent, response: nil)
    interaction.direction = .incoming
    interaction.donate(completion: nil)
    do {
        let callContent = try request.content.updating(from: incomingCallIntent)
        contentHandler(callContent)
    } catch {
       // Handle error
    }
}
```

- always set interaction direction to `.incoming` for notifications
- send intents also for outgoing interactions (a.k.a when the user sends messages or make calls)
  - Siri only learns from outgoing intent donations, so Contacts are not cluttered by people not engaged with

How to configure:

- enable the associated capability via Xcode for your application
- add intent types to `NSUserActivityTypes` in `Info.plist`

[INStartCallIntent]: https://developer.apple.com/documentation/sirikit/instartcallintent
[INSendMessageIntent]: https://developer.apple.com/documentation/sirikit/insendmessageintent

[UNNotificationInterruptionLevel]: https://developer.apple.com/documentation/usernotifications/unnotificationinterruptionlevel
[UNNotificationActionIcon]: https://developer.apple.com/documentation/usernotifications/unnotificationactionicon
[UNNotificationAction]: https://developer.apple.com/documentation/usernotifications/unnotificationaction
[UNNotificationContent]: https://developer.apple.com/documentation/usernotifications/unnotificationcontent
[relevanceScore]: https://developer.apple.com/documentation/usernotifications/unnotificationcontent/3821031-relevancescore
[UNNotificationCategoryOptions.allowAnnouncement]: https://developer.apple.com/documentation/usernotifications/unnotificationcategoryoptions/3240647-allowannouncement
