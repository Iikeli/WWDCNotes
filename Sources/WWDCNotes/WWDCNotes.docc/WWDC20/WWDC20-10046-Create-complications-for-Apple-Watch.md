# Create complications for Apple Watch

When you add complications to a Watch app, people can access glanceable and up to date information directly from their watch face. We’ll show you how to create and build complications from the ground up and introduce you to Multiple Complications. Learn how to construct timelines, use families and templates, and discover best practices on crafting a thorough complication experience.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10046", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Timelines

- Representation of your complication's data over time
- Enables ClockKit to query your app once and get all information needed
- Extend or invalidate as necessary to ask ClockKit to re-query your app

- Your complication will show an entry until the date of the next one

## Complication building blocks

- There are dozens of [watchOS families][families]
- Ideally you will want to support as many complication families as you can
- For each family there are multiple [templates][families]
- All templates inherit from [`CLKComplicationTemplate`][CLKComplicationTemplate]

## Providing data

- When providing a timeline, we're giving WatchKit a list of [`CLKComplicationTimelineEntry`][CLKComplicationTimelineEntry] instances.
- These will populate your complications
- Each entry represents what your complications should look like at a certain point in time. 
- Each entry has two properties:
  - Date, which is the date that this entry should be visible
  - Complication template, which is the template containing the data you want to display for this entry

- Your main interaction with ClockKit is through an object you create that conforms to [`CLKComplicationDatasource`][CLKComplicationDatasource]. 
- There's only one required method in this protocol:

```swift
class ComplicationController: NSObject, CLKComplicationDataSource {
    func getCurrentTimelineEntry(
        for complication: CLKComplication, 
        withHandler handler: @escaping (CLKComplicationTimelineEntry?) -> Void) {
        // Call the handler with the current timeline entry
        handler(createTimelineEntry(forComplication: complication, date: Date()))
    }
}
```

- This function is used to get the current entry only, if we can/want to provide future entries as well, we will need to implement the following methods as well:

```swift
extension ComplicationController {

    // Specifies how far in the future you can provide entries 
    func getTimelineEndDate(
        for complication: CLKComplication, 
        withHandler handler: @escaping (Date?) -> Void) {
        handler(timeline(for: complication)?.endDate)
    }

    // lets you provide as many entries as is appropriate up to the limit after the given date
    func getTimelineEntries(
        for complication: CLKComplication, 
        after date: Date, 
        limit: Int, 
        withHandler handler: @escaping ([CLKComplicationTimelineEntry]?) -> Void) {
       handler(timeline(for: complication)?.entries(after: date, limit: limit))
    }
}
```

### Reloading complications

- Call [`reloadTimeline(for:)`][reloadTimeline(for:)] on the `.sharedInstance()` [`CLKComplicationServer`][CLKComplicationServer] to invalidate the current timeline (and trigger an update session to reload it)
- Call [`extendTimeline(for:)`][extendTimeline(for:)] on the `.sharedInstance()` [`CLKComplicationServer`][CLKComplicationServer] to let ClockKit know we can now provide more entries

### Data Providers

- Provided by ClockKit to adapt the display of the same data/values in different templates or families of complications. 
- [`CLKDateTextProvider`][CLKDateTextProvider] will take care of displaying a date for you

```swift
let longDate: Date = DateComponents(year: 2020, month: 9, day: 23).date ?? Date()
let units: NSCalendar.Unit = [.weekday, .month, .day]
let textProvider = CLKDateTextProvider(date: longDate, units: units)
```

- [`CLKRelativeDateTextProvider`][CLKRelativeDateTextProvider] will take care of displaying relative time/intervals

```swift
let timerStart: Date = …
let units: NSCalendar.Unit = [.hour, .minute, .second]
let textProvider = CLKRelativeDateTextProvider(date: timerStart, style: .timer, units: units)
```

- and [many others][dataProv] including image and gauge providers

## Multiple complications

- New in WatchOS 7
- Declared via our [`CLKComplicationDatasource`][CLKComplicationDatasource] implementation via [`getComplicationDescriptors(handler:)`][getComplicationDescriptors(handler:)]
- We will use a [`CLKComplicationDescriptor`][CLKComplicationDescriptor] to define each complication
- If you want to update the complications your app offer, call [`reloadComplicationDescriptors()`][reloadComplicationDescriptors()]  on the `.sharedInstance()` [`CLKComplicationServer`][CLKComplicationServer] 
- Note that if you remove support for a complication that is currently on a user watch face, WatchKit will continue to ask you for timeline entries for that complication

## Getting information back to your app

- Tapping a complication launches your app
- Based on the `CLKComplicationDescriptor` description define above, the app will be launched with:
  - A `userActivity` 
  - A `userInfo` dictionary

- Either way, WatchKit will pass more data in the `userInfo` dictionary (passed also within the `userActivity`) such as the [`CLKLaunchedTimelineEntryDateKey`][CLKLaunchedTimelineEntryDateKey] and [`CLKLaunchedComplicationIdentifierKey`][CLKLaunchedComplicationIdentifierKey]

[CLKComplicationTemplate]: https://developer.apple.com/documentation/clockkit/clkcomplicationtemplate
[CLKComplicationTimelineEntry]: https://developer.apple.com/documentation/clockkit/clkcomplicationtimelineentry
[CLKComplicationDatasource]: https://developer.apple.com/documentation/clockkit/clkcomplicationdatasource
[getComplicationDescriptors(handler:)]: https://developer.apple.com/documentation/clockkit/clkcomplicationdatasource/3555131-getcomplicationdescriptors
[CLKComplicationDescriptor]: https://developer.apple.com/documentation/clockkit/clkcomplicationdescriptor
[reloadComplicationDescriptors()]: https://developer.apple.com/documentation/clockkit/clkcomplicationserver/3555139-reloadcomplicationdescriptors
[CLKLaunchedComplicationIdentifierKey]: https://developer.apple.com/documentation/clockkit/clklaunchedcomplicationidentifierkey
[CLKLaunchedTimelineEntryDateKey]: https://developer.apple.com/documentation/clockkit/clklaunchedtimelineentrydatekey
[CLKDateTextProvider]: https://developer.apple.com/documentation/clockkit/clkdatetextprovider
[CLKRelativeDateTextProvider]: https://developer.apple.com/documentation/clockkit/clkrelativedatetextprovider
[dataProv]: https://developer.apple.com/documentation/clockkit/data_providers
[reloadTimeline(for:)]: https://developer.apple.com/documentation/clockkit/clkcomplicationserver/1627891-reloadtimeline
[extendTimeline(for:)]: https://developer.apple.com/documentation/clockkit/clkcomplicationserver/1627895-extendtimeline
[CLKComplicationServer]: https://developer.apple.com/documentation/clockkit/clkcomplicationserver
[families]: https://developer.apple.com/design/human-interface-guidelines/watchos/overview/complications/