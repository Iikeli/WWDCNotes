# Meet WidgetKit

Meet WidgetKit: the best way to bring your app’s most useful information directly to the home screen. We'll show you what makes a great widget and take a look at WidgetKit's features and functionality. Learn how to get started creating a widget, and find out how WidgetKit leverages the power of SwiftUI to provide a stateless experience. Discover how to harness your existing proactive technologies to make sure your widget surfaces relevant material. And create a Timeline that ensures your content is always fresh.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10028", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(abadikaka)
      @GitHubUser(zntfdr)
   }
}



## Where we can find the new widgets

- iPhone and iPad Home screens
- iPhone and iPad  Today view
- macOS Notification Center

## What Makes A Great Widget

* Glanceable: great widgets display the right amount of content
* Relevant: great widgets display relevant content at the right place and time, this is very important for Smart Stack, too
* Personalized: great widgets allow configurations and support as many sizes as possible, widget configurations are done via intents
* Widget are not mini apps: think of the as your app content projection onto the homescreen

### How [`WidgetKit`][wkDoc] Works
![][widget_works]

* All widgets are built in SwiftUI
* `WidgetKit` extensions are background extensions that return a series of view hierarchies (a.k.a. views) in a [timeline][tl].
* Our widget extension donates this timeline (with our views) to the Home screen which will present them at the correct time according to the timeline.
* This way there's no app launch, load etc: the widgets are immediately glanceable
* Timelines can be refreshed from main app or via scheduled update from the extensions

### Defining a Widget

Main concepts:

* Kind (a custom `String`): identifies the different widgets that we provide (for example we can have a `detail` widget display only one object, and a `list` widget displaying multiple objects)

* Configuration: Either [`StaticConfiguration`][sconf] (no user configuration) or [`IntentConfiguration`][iconf] (allows user customization)

| ![][static_configuration] | ![][intent_configuration] |

* [`WidgetFamily`][wf]: a.k.a. the templates/sizes a widget support: it can be any combination of [small][syssmall], [medium][sysmed], or [large][syslarge]

* Placeholder: A temporary view WidgetKit uses to render the widget for the first time, generic representation of the widget (without any user data)

![][placeholderImage]

### Creating a Glanceable Experience

* Stateless UI
* No scrolling
* No videos or animated images
* Easy tap interactions with deep link into our app

### Views, Timelines, and Reloads

Three types of UI we should provide:

* Placeholder
* [Snapshot][snap]: when is where the system needs to quickly display a single entry, our extension should return it as quickly as possible
* Timelines: basically multiple snapshots combined with a date that tells the system at what time that view should be shown

#### Reloads

- Reloads happen when the system wake up the widget to ask for a new timeline for each widget displayed on the device.
- Reloads make sure that our widget content is always up-to-date for our user

### [TimelineProvider][tp] Definition

```swift
public protocol TimelineProvider {
    associatedType Entry: TimelineEntry
    typealias Context = TimelineProviderContext

    func snapshot(with context: Self.Context, 
                    completion: @escaping (Self.Entry) -> ())

    func timeline(with context: Self.Context, 
                    completion: @escaping (Timeline<Self.Entry>) -> ())

}
```

* `TimelineEntry`: mainly a date.
* `Context`: environment information for which the system is asking us for entries.
* snapshot function: the system is asking us for a single entry (to be returned as soon as possible)
* Timeline function: the system is asking us for series of entries and attached reload policy (a.k.a. when we want the system to ask for a new timeline)

#### [Timeline Reload policy][reload]

When our extension [`TimelineProvider`][tp] is asked to provide a [`Timeline`][tl], we also need to define our reload policy:

* `atEnd`: tells `WidgetKit` to request a new timeline only after the date of the last entry has passed
* `after(date: Date)`: tells `WidgetKit` to request a new timeline only after a specified date
* `Never`: tells `WidgetKit` to never request a new timeline, the app will let `WidgetKit` know when a new timeline is available

#### Reloading via app

* We can ask the system to reload a specific widget or all widget kinds.
* We can use `URLSession` to kick off a task and use batch request as well background session to reload our widgets. 

### Personalization and Intelligence Aspects

* Driven by two major contexts: 
  - Intents: used as a mechanism to allow users to configure our widget
  - Relevance: which allows us to inform the intelligence in the widget stack.

* Intents are powered by the [Intents framework][intentsDoc], the same used with Siri and Shortcuts
* The widget relevance is particularly useful when the user has multiple widgets into a smart stack: the stack will be sorted based on each entry [`TimelineEntryRelevance`][rel], duration, and more

[wkDoc]: https://developer.apple.com/documentation/widgetkit
[wf]: https://developer.apple.com/documentation/widgetkit/widgetfamily
[snap]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/snapshot(for:with:completion:)
[tp]: https://developer.apple.com/documentation/widgetkit/timelineprovider
[reload]: https://developer.apple.com/documentation/widgetkit/timelinereloadpolicy
[tl]: https://developer.apple.com/documentation/widgetkit/timeline
[rel]: https://developer.apple.com/documentation/widgetkit/timelineentryrelevance
[sconf]: https://developer.apple.com/documentation/widgetkit/staticconfiguration
[iconf]: https://developer.apple.com/documentation/widgetkit/intentconfiguration
[syssmall]: https://developer.apple.com/documentation/widgetkit/widgetfamily/systemsmall
[sysmed]: https://developer.apple.com/documentation/widgetkit/widgetfamily/systemmedium
[syslarge]: https://developer.apple.com/documentation/widgetkit/widgetfamily/systemlarge
[intentsDoc]: https://developer.apple.com/documentation/sirikit

[widget_works]: WWDC20-10028-widget_works
[static_configuration]: WWDC20-10028-static_configuration
[intent_configuration]: WWDC20-10028-intent_configuration
[placeholderImage]: WWDC20-10028-placeholder
