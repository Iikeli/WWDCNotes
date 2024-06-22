# Using Grouped Notifications

Grouping the notifications your app sends helps people get more information at a glance and manage multiple notifications at once. Learn how to implement Grouped Notifications in your app.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/711", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

- iOS 12 introduces grouping of notifications. The aim is to help organize content for users.
- Part of these changes include summaries displayed on the **Lead Notification** (top of the group stack). These can be customized to provide clarity for the user about the notifications grouped.
- Grouping can be customized through the ‘thread-identifier’ key in the push payload. 

## Strategies

- Basically this session is just a list of suggestions on how to use grouping in your app
- Apps can adopt different grouping strategies. Ex. Calendar groups informative updates from actionable events. Ex. mail allows user preferences to alter the grouping of content. Ex. Messages groups based on conversations.
  - Separate important, actionable notifications from informative updates.
  - Create groups for meaningful personal communications
  - Respect the users priorities and organization.

## Notification Group Summary

- Group summaries can be customized (more content to read / number in the group/ the people in the conversation).
- The basic version, is done  by providing a summary format in the push notification category:

```swift
let summaryFormat = "%u more messages"

return UNNotificationCategory(
	identifier: "category-identifier",
	actions: [],
	intentIdentifiers: [], 
	hiddenPreviewsBodyPlaceholder: nil,
	categorySummaryFormat: summaryFormat,
	options: []
) 
```

- Alternatively, we also have a `hiddenPreviewPlaceholder` (this is used when the phone is locked and the notifications are private):

```swift
let summaryFormat= "%u more messages"
let hiddenPreviewsPlaceholder = "%u messages" 

return UNNotificationCategory(
	identifier: "category-identifier", 
	actions: [], 
  intentIdentifiers: [], 
  hiddenPreviewsBodyPlaceholder: hiddenPreviewsPlaceholder,
  categorySummaryFormat: summaryFormat,
  options: []
) 
```

- If we want something more than just the number of notifications such as:
![][pizzaImage]
We need to pass the placeholders in the summary format:

```swift
let summaryFormat = "%u more messages from %@"

return UNNotificationCategory(
	identifier: "group-messages",
	actions: [], 
  intentIdentifiers: [], 
  hiddenPreviewsBodyPlaceholder: nil,
  categorySummaryFormat: summaryFormat,
  options: []
)
```

- Then we will pass the other parameters in the notification content:

```swift
// Notification Group Summary Argument 

let content = UNMutableNotificationContent()
content.body = "..."
content.threadIdentifier = "notifications-team" 
content.summaryArgument= "Kritarth" 
```

Note that iOS will collect multiple values of the summaryArgument and format them for us

- In the previous examples the %u was matching the number of notifications that we had in that group, but sometimes a notification is like “you have two new messages”, so how can I summarize this?
Easy, new property in the notification content:

```swift
let content = UNMutableNotificationContent() 
content.body = "..."
content.threadIdentifier = "..."
content.summaryArgument = "Song by Song" 
content.summaryArgumentCount = 3 
```

- Localization for summaries can be handled locally from the app via plist bundled with the app

[pizzaImage]: WWDC18-711-pizza