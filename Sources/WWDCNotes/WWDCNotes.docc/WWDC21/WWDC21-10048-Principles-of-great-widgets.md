# Principles of great widgets

Explore the foundations of great widgets by keeping them relevant and customizable. Learn how to keep widgets up to date with timeline entries and TimelineReloadPolicies. Discover how to adapt your widget to different presentation environments and physical location. And lastly, find out how to create customizable widgets that someone can personalize to their liking.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10048", purpose: link, label: "Watch Video (26 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- provide as many timeline future entries as possible (when it makes sense for your app)
- support as many sizes as you possible, so that users have choice when placing their widgets

## Widgets relevance

- Time
- Presentation
- Location

## Widgets reloads

- Individual background reload budgets
- Budget updates throughout the day
- Influenced by viewing habits
- Variable update cadence per widget
- Updates are withheld until new budget is available
- A frequently viewed widget can be expected to receive 40-70 background updates per day
- Reloads are on the order of minutes/hours, not seconds

### Ways the widget will refresh

|    | Budgeted | Free |
| --- | --- | --- |
| `TimelineReloadPolicy` API | ✅ | ❌ |
| `WidgetCenter` reload API | ✅ | ✅ |
| Significant location changes | ❌ | ✅ |
| System updates | ❌ | ✅ |

- `TimelineReloadPolicy` API
  - Scheduled background updates `atEnd` `afterDate:` `never`
  - informs the system when you’d like to automatically refresh your widget in the background

- `WidgetCenter` API
  - Event-based triggers `.reloatTimelines(ofKind:)` `.reloadAllTimelines`
  - free when:
    - the reload occurs when the host app is in the foreground
    - the user is in a current session, like Navigation or Now Playing audio

- Significant location changes
  - budget-free update when the system detects a significant location change
  - this free refresh occurs when the user views your widget (not when the location changes)

- System updates
  - when a big change in the system happen:
    - the user changes an Accessibility preference like dynamic text or bold text
    - language or region change
    - iCloud or App Store account change
    - significant time change
    - and more

## TimelineReloadPolicy

- `atEnd`
  - marks your widget eligible to be refreshed when the current timeline last entry becomes relevant
  - recommended if your widget already has content that extends beyond the life of its current timeline
  - not recommended for single-entry timeline, as the system would choose a reload time for you
  - not recommended for data that loses accuracy over time (e.g. weather)
  - Apple apps using `atEnd` policy: Reminders, Calendar, Photos, Tips, and more

- `afterDate`
    - marks your widget eligible to be refreshed after the date you specify
    - recommended for widgets showing unpredictable data (that can change unpredictably/unexpectedly)
    - recommended for widgets showing data whose accuracy or relevance changes periodically
    - Apple apps using `afterDate` policy: Stocks, Weather, News, Mail, and more
    - be cautious of:
      - very frequent reloads (you risk to consume all refresh budget)
      - setting a date where new data will be available 
      - (if backend data driven) to not overload your servers (add jitter to the data, use caching servers)

- `never`
  - it never refreshes automatically
  - recommended for content that: 
    - only changes through user interaction
    - doesn't change 
    - gated on conditional access 

- 
  - Use `WidgetCenter` API to refresh from foreground application or events
  - Apple apps using `never` policy: TV, notes, music, podcast, contacts, and more

## Appearance 

### Full privacy redactions

- Individual views may now be automatically masked in privacy-sensitive environments
- new [`privacySensitive(_:)`][privacySensitive(_:)] view modifier: when the user is not authenticated, the view will automatically be masked/redacted

### default-data-protection entitlement

If your app leverages complete data protection that can’t be accessed while an iOS device is passcode locked (e.g. Health data), you can now tell WidgetKit to:

- replace your active timeline content with your placeholder content when the device is passcode locked for a full redaction of content
- withhold updates while the device is passcode locked

Adopt the entitlement `com.apple.developer.default-data-protection` with value `NSFileProtectionComplete` to opt in into this behavior

## Location

If you'd like to use user location in your widget:

1. add `NSWidgetUsesLocation` with `true` value in your `Info.plist`
2. use `CLLocationManager` from your `TimelineProvider` in your widget extension
  - use the lowest resolution possible (for faster location resolution and less battery usage)
  - use `isAuthorizedForWidgetUpdates` to check whether the widget can get the user location

### Widget location permission

Based on app permission:

- never: ❌
- ask next time ("allow once"): ❌
- while using app: only when the widget’s container app is in the foreground or other situations that would consider the app to be in-use, like being in a navigation session
- while using app or widgets: same as "while using app", plus the widget can receive location up to 15 minutes after it was last viewed
- always: ✅

[privacySensitive(_:)]: https://developer.apple.com/documentation/swiftui/view/privacysensitive(_:)
