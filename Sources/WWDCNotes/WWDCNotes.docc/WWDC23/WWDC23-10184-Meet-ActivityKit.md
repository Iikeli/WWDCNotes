# Meet ActivityKit

Live Activities are a glanceable way for someone to keep track of the progress of a task within your app. We’ll teach you how you can create helpful experiences for the Lock Screen, the Dynamic Island, and StandBy. Learn how to update your app’s Live Activities, monitor activity state, and take advantage of WidgetKit and SwiftUI to build richer experiences.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10184", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(RamitSharma991)
   }
}




Live Activities are an immersive and glanceable way to keep track of an event or the progress of a task. They have a discrete start and end and provides real-time updates from background app on runtime or remotely using Push Notifications.

## Live Activity overview

* Works with ***ActivityKit framework***, empowering apps to request, update, and manage their lifecycles.
* Declarative programmatic layout with SwiftUI and WidgetKit.
* Explicit user action to begin a Live Activity.
* A Live Activity can be requested when your app is in the foreground. 
* Live Activities are user-moderated similar to Notifications.
* Someone can easily dismiss or turn them off for your app altogether.
* Must support Lock Screen and the Dynamic Island. 
* Update remotely using push notifications.
* Dynamic Island displays Live Activities throughout the system when the app is in the background. 
* When one Live Activity is active, it’s rendered using its `variable-width`, compact presentation.
* The Dynamic Island displays up to two live activities at a time.
* One of these Live Activities appears attached to the TrueDepth camera, while the other renders in its own detached view.
* Both of these Live Activities use their `minimal` presentation.
* Long press a Live Activity to display its `expanded` presentation, giving even more glanceable information.
* In the expanded presentation, views can deep link to different areas within the app. 

### New experiences for Live Activities in iOS 17

* In addition to the Lock screen and Dynamic Island, Live Activities appear in StandBy.
* Now iPad supports Live Activities. 
* Interactive Live Activities 
   * Leverage widget enhancements
   * Add buttons or toggles


## Lifecycle of Live Activities

The Lifecycle of a Live Activity contains four main steps: 

* Request

```swift
// Request Live Activity with initial content state
let adventure = AdventureAttributes(hero: hero)

let initialState = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "Adventure has begun!"
)
let content = ActivityContent(state: initialState, staleDate: nil, relevanceScore: 0.0)

let activity = try Activity.request(
    attributes: adventure,
    content: content,
    pushType: nil
)
```
* Update 

```swift
// Update Live activity with new content
let heroName = activity.attributes.hero.name               
let contentState = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "\(heroName) has taken a critical hit!"
)

var alertConfig = AlertConfiguration(
    title: "\(heroName) has taken a critical hit!",
    body: "Open the app and use a potion to heal \(heroName)",
    sound: .default
)  
activity.update(
    ActivityContent<AdventureAttributes.ContentState>(
        state: contentState,
        staleDate: nil
    ),
    alertConfiguration: alertConfig
)
```
* Observe activity state

```swift
// Observe activity state asynchronously
func observeActivity(activity: Activity<AdventureAttributes>) {
    Task {
        for await activityState in activity.activityStateUpdates {
            if activityState == .dismissed {
                self.cleanUpDismissedActivity()
            }
        }
    }
}

// Observe activity state synchronously
let activityState = activity.activityState
if activityState == .dismissed {
    self.cleanUpDismissedActivity()
}
```
* End

```swift
// Dismiss Live activity with final content.
let hero = activity.attributes.hero

let finalContent = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "Adventure over! \(hero.name) has defeated the boss! Congrats!"
)

let dismissalPolicy: ActivityUIDismissalPolicy = .default

activity.end(
    ActivityContent(state: finalContent, staleDate: nil),
    dismissalPolicy: dismissalPolicy)
}
```

## Building Live Activity UI

* Show most essential content
* Simple design
* Show additional details in the application

### Dynamic Island presentations.

 * There are three presentations: ***compact, minimal and expanded.***
 * When an app’s Live Activity is the only one running on the system, it’ll be displayed using the compact presentation.
 * The compact presentation has two areas, ***leading and trailing.***
 * They appear together to form a cohesive presentation in the Dynamic Island.
 * Choose essential content to show in the leading and trailing space.
 * Users should be able to identify the specific activity by looking at the content.
 * Create a Dynamic Island view builder to represent each of those presentations. 
 * When more than one app starts a Live Activity, the system chooses which Live Activities are visible and displays both of them using the minimal presentation for each: 
    * one minimal presentation appears attached to the Dynamic Island 
    * the other appears detached.
 * Your minimal view should only have the most critical information, as you have very limited space to work with.
 * When users touch and hold a Live Activity in a compact or minimal presentation, the system displays the content in an expanded presentation.
 * For the expanded presentation, the system divides the expanded presentation into different areas. 
 * The first closure of the DynamicIsland view builder represents the expanded content. 
 * Within that closure, each section content can be defined with the expanded region passing the specific position.

## Related Sessions

- [Bring widgets to life - WWDC23](https://developer.apple.com/wwdc23/10028)
- [Update Live Activities with Push Notifications - WWDC23](https://developer.apple.com/wwdc23/10185)
- [Design Dynamic Live Activities - WWDC23](https://developer.apple.com/wwdc23/10194)
