# Widgets Code-along, part 2: Alternate timelines

Our code-along continues as we help our widget rewrite the future and travel into an alternate timeline. Continue where you left off from Part 1, or traverse time and space and begin with the Part 2 starter project to jump right into the action. Find out how you can integrate system intelligence into your widgets to help them dynamically change at different points during the day and surface the most relevant information. Explore core timeline concepts, support multiple widget families, and learn how to make your widget configurable.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10035", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Families

Three available families:

![][familiesImage]

When drawing our widget, we can detect which widget family we're drawing the [`.widgetFamily`][wfDoc] environment variable:

```swift
struct WWDCNotesWidgetEntryView : View {
  var entry: Provider.Entry
  @Environment(\.widgetFamily) var family

  var body: some View {
    Text(entry.date, style: .time)
  }
}
```

## Timelines

The [`IntentTimelineProvider`][itpdoc] is the engine of the widget.

When our `TimelineProvider` is asked to provide a [`Timeline`][tl], we also need to define our reload policy:

* `atEnd`: tells `WidgetKit` to request a new timeline only after the date of the last entry has passed
* `after(date: Date)`: tells `WidgetKit` to request a new timeline only after a specified date
* `Never`: tells `WidgetKit` to never request a new timeline, the app will let `WidgetKit` know when a new timeline is available.

When creating a timeline, we can also set an optional `relevance` to each entry, [`TimelineEntryRelevance`][rel], in order to let the system know how important each entry is (compared to other entries from the same widget).

## Configuration

WidgetKit configuration is driven by SiriKit, the core technology for configuration is a custom [`INIntent`][intentsDoc]s.

> For more, refer to session [`Add configuration and intelligence to your widgets`][wwdc20-10194].

To create a new configuration/intent go to `File > New File` and choose `SiriKit Intent Definition File.`.

This will create a `.intentdefinition` file whose target membership must be both the widget target and the main app.

After opening this file create a new intent and:

- set its category to `View`
- check the `is elegible for widgets` option
- add your configuration parameters

Once the intent/configuration setup is complete, we need to go back to our widget definition and make sure that our widget configuration is a [`IntentConfiguration`][iconf], the difference from the default [`StaticConfiguration`][sconf] (beside that one allows configuration and the other doesn't) is that `IntentConfiguration` requires an extra `intent` parameter where we can define set our intent as if it was a class/struct:

```swift
@main
struct WWDCNotesWidget: Widget {
  private let kind: String = "WWDCNotesWidget"

  public var body: some WidgetConfiguration {
    IntentConfiguration(
      kind: kind, 
      intent: WWDCNotesCustomIntent.self, // this is our intent
      provider: Provider(), 
      placeholder: PlaceholderView()
    ) { entry in
      WWDCNotesWidgetEntryView(entry: entry)
    }
    .configurationDisplayName("My Widget")
    .description("This is an example widget.")
  }
}
```

If we use an `IntentConfiguration`, we also need to make sure that we declare a `IntentTimelineProvider` instead of a `TimelineProvider`, the difference with the base `TimelineProvider` is that both the [`snapshot(for:with:completion:)`][snap] and [`timeline(for:with:completion:)`][tlDoc] functions have an extra configuration parameters (that matched our intent type)

```swift
struct Provider: IntentTimelineProvider {
  typealias Intent = WWDCNotesCustomIntent
  public typealias Entry = SimpleEntry
  
  public func snapshot(
    for configuration: WWDCNotesCustomIntent, // extra parameter
    with context: Context, 
    completion: @escaping (SimpleEntry) -> Void
  ) {
      ...
  }

  public func timeline(
    for configuration: WWDCNotesCustomIntent, // extra parameter
    with context: Context, 
    completion: @escaping (Timeline<Entry>) -> Void
  ) {
    ...
  }
}
```

## Deep linking

Widgets do not have animation or custom interactions, but we can deep-link from our widget into our app:

- `.systemSmall` widgets are one large tap area
- `.systemMedium` and `.systemLarge` can use the new SwiftUI [`Link`][linkDoc] API to create tappable zones within the widget.

To add deep links into our app, we use the [`.widgeturl(_:)`][wurlDoc] modifier.

In our app then we apply the new [`.onOpenURL(perform:)`][oourlDoc] to the right view to manage deeplinks.

[wwdc20-10194]: https://www.wwdcnotes.com/notes/wwdc20/10194

[familiesImage]: WWDC20-10035-families

[itpdoc]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider
[reload]: https://developer.apple.com/documentation/widgetkit/timelinereloadpolicy
[tl]: https://developer.apple.com/documentation/widgetkit/timeline
[intentsDoc]: https://developer.apple.com/documentation/sirikit/inintent
[linkDoc]: https://developer.apple.com/documentation/swiftui/link
[wfDoc]: https://developer.apple.com/documentation/swiftui/environmentvalues/widgetfamily
[rel]: https://developer.apple.com/documentation/widgetkit/timelineentryrelevance
[iconf]: https://developer.apple.com/documentation/widgetkit/intentconfiguration
[sconf]: https://developer.apple.com/documentation/widgetkit/staticconfiguration
[snap]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/snapshot(for:with:completion:)
[tlDoc]: https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/timeline(for:with:completion:)
[wurlDoc]: https://developer.apple.com/documentation/swiftui/view/widgeturl(_:)
[oourlDoc]: https://developer.apple.com/documentation/swiftui/view/onopenurl(perform:)