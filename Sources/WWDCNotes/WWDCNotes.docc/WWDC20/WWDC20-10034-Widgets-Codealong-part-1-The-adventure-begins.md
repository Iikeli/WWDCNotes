# Widgets Code-along, part 1: The adventure begins

Take your app on a most wondrous adventure to the home and Today screens of iPhone, iPad, and Mac. Grab the starter project and code along with us! We will guide you through the process of creating a widget for your app from start to finish so that you can provide people with beautiful views and glanceable information in an easily-accessible place. Discover how to create a widget project, learn fundamental concepts for widgets and their structure, configure the widget and its provider, and start exploring timeline concepts.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10034", purpose: link, label: "Watch Video (9 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



This session is a Code Along session, the session project can be found [here][projdwl].

## What is a Widget?

A widget is a SwiftUI view that updates over time, how and when it updates is covered in this and the following sessions.

## How to add a widget to an existing app

1. Create a new Widget target: `File > New > Target > Widget Extension`

This will create a new target with a new project folder associated with it. 

In this folder we have one `.swift` file, named after our widget, which declares:

- a [`IntentTimelineProvider`][itpdoc] 
- a [`Widget`][widgetDoc], which contains your widget declaration
- a [`TimelineEntry`][timelineEntryDoc]
- a placeholder `View`, which is a view WidgetKit uses to render the widget for the first time
` an entry `View`, which is the default view used by the widget

```swift
import WidgetKit
import SwiftUI
import Intents

struct Provider: IntentTimelineProvider {
  public func snapshot(for configuration: ConfigurationIntent, with context: Context, completion: @escaping (SimpleEntry) -> ()) {
    let entry = SimpleEntry(date: Date(), configuration: configuration)
    completion(entry)
  }

  public func timeline(for configuration: ConfigurationIntent, with context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
    var entries: [SimpleEntry] = []

    // Generate a timeline consisting of five entries an hour apart, starting from the current date.
    let currentDate = Date()
    for hourOffset in 0 ..< 5 {
      let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
      let entry = SimpleEntry(date: entryDate, configuration: configuration)
      entries.append(entry)
    }

    let timeline = Timeline(entries: entries, policy: .atEnd)
    completion(timeline)
  }
}

struct SimpleEntry: TimelineEntry {
  public let date: Date
  public let configuration: ConfigurationIntent
}

struct PlaceholderView : View {
  var body: some View {
    Text("Placeholder View")
  }
}

struct WWDCNotesWidgetEntryView : View {
  var entry: Provider.Entry

  var body: some View {
    Text(entry.date, style: .time)
  }
}

@main
struct WWDCNotesWidget: Widget {
  private let kind: String = "WWDCNotesWidget"

  public var body: some WidgetConfiguration {
    IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider(), placeholder: PlaceholderView()) { entry in
      WWDCNotesWidgetEntryView(entry: entry)
    }
    .configurationDisplayName("My Widget")
    .description("This is an example widget.")
  }
}

struct WWDCNotesWidget_Previews: PreviewProvider {
  static var previews: some View {
    WWDCNotesWidgetEntryView(entry: SimpleEntry(date: Date(), configuration: ConfigurationIntent()))
      .previewContext(WidgetPreviewContext(family: .systemSmall))
  }
}
```

2. in the `Widget` declaration, update the default configuration with your widget title and description (those are two modifiers, [`.configurationDisplayName`][cdnDoc] and [`.description`][descDoc])

3. declare which widget family/sizes you'd like to support by adding a [`.supportedfamilies(_:)`][sfDoc] modifier, by default all three families are supported.

4. the dynamic data of the widget should come from your `TimelineEntry`: any required data should be defined there.

5. `TimelineEntry` instances come from our `TimelineProvider`: we will need to update our `TimelineProvider` definition by adding the necessary entry data in the [`snapshot(for:with:completion:)`][snap] and [`timeline(for:with:completion:)`][tlDoc] functions

## Tips

- Use [`.previewContent(WidgetPreviewContext(family: .system..))`][widgetPreviewDoc] on SwiftUI previews to display the widget layout.

- use the `.isPlaceholder(true)` modifier on the preview view (note: `.isPlaceholder` is not available as of Xcode 12b2)

[projdwl]: https://developer.apple.com/documentation/widgetkit/building_widgets_using_widgetkit_and_swiftui
[itpdoc]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider
[widgetDoc]: https://developer.apple.com/documentation/swiftui/widget
[timelineEntryDoc]: https://developer.apple.com/documentation/widgetkit/timelineentry
[widgetPreviewDoc]: https://developer.apple.com/documentation/widgetkit/widgetpreviewcontext
[cdnDoc]: https://developer.apple.com/documentation/widgetkit/intentconfiguration/configurationdisplayname(_:)-3ubj0
[descDoc]: https://developer.apple.com/documentation/widgetkit/intentconfiguration/description(_:)-1yars
[snap]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/snapshot(for:with:completion:)
[tlDoc]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/timeline(for:with:completion:)
[sfDoc]: https://developer.apple.com/documentation/widgetkit/staticconfiguration/supportedfamilies(_:)