# Explore enhancements to App Intents

Bring your widgets to life with App Intents! Explore the latest updates and learn how you can take advantage of dynamic options and user interactivity to build better experiences for your App Shortcuts. We’ll share how you can integrate with Apple Pay, structure your code more efficiently, and take your Shortcuts app integration to the next level.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10103", purpose: link, label: "Watch Video (29 min)")

   @Contributors {
      @GitHubUser(Cecile-Lebleu)
   }
}



Speaker: Roman Efimov, Shortcuts Engineering

## Widgets

New options to connect App Intents with Widgets through interactivity and configuration.

### Widget configuration

The options found on the back of a configurable widget are called Parameters, and they're added with Intents. Previously Intents had to be declared in an Intent Definition file, but now they can be declared directly in the Widget extension code.

- Use the new `AppIntentConfiguration` WidgetConfiguration type, instead of `IntentConfiguration`
- Define a type that conforms to the `WidgetConfigurationIntent` protocol
- Use `@Parameter` to add widget configurations

```swift
// App Intents widget configuration
@main
struct UpNextWidget: Widget {
	let kind: String = "UpNext"
		var body: some WidgetConfiguration {
			AppIntentConfiguration( // NEW, instead of IntentConfiguration()
			kind: kind, intent: UpNextConfiguration.self,
			provider: Provider()
		) { entry in
			UpNextWidgetView(entry: entry)
		}
	}
}

struct UpNextConfiguration: AppIntent, WidgetConfigurationIntent {
	static var title: LocalizedStringResource = "Up Next"
	
	@Parameter(title: "Example")
	var example: Example
}
```

Providing dynamic options can be done right here too, instead of creating a separate Intents extension. Queries and dynamic option providers can be implemented.

```swift
struct ExampleQuery: EntityStringQuery {
	func entities(
		matching string: String
	) async throws -> [Example] { ... }
}
```

*See more in the session "Dive into App Intents" from WWDC22.*

### Migrating widgets from SiriKit to App Intents

- Support latest and previous OS
- Enable continued use of existing widgets
- Remove SiriKit Intent Definition file (do not do this if you plan to support previous OS versions)

Migration is automatic. In the Intent definition file, go to the SiriKit widget configuration Intent, and click "Convert to App Intent...". Make sure to test.

### Interactive widgets

Widgets now support button taps and toggles. Swift UI buttons and toggles now support intents.

```swift
struct SetAlarm: AppIntent {
	static var title: LocalizedStringResource = "Set Alarm"
	
	@Parameter (title: "Bus Stop")
	var busStop: BusStop
	
	// Other parameters...
	
	func perform() async throws -> some IntentResult {
		AlarmManager.shared.addAlarm(forTime: arrivalTime)
		return .result()
	}
}

struct NextBusView: View {
	var body: some View {
		Button(intent: SetAlarm(arrivalTime: arrivalTime)) {
			Text(arrivalTime.asString)
		}
	}
}
```

AppIntents are also available outside of Widgets, in regular SwiftUI apps. App intents can serve as a configuration, so sharing code can reduce redundancy and ensure consistent behavior. WidgetConfigurationIntents can also serve as Shortcuts actions.

*See more in the session "Bring your widget to life" from WWDC23.*
### Dynamic options
Conform to `DynamicOptionsProvider` or the `EntityQuery` protocols to provide the available values of a parameter in the App Intent.

```swift
struct BusStopQuery: EntityStringQuery {
	func entities(
		matching string: String
	) async throws -> [BusStop] {
		BusStop.allStops.filter {
			$0. name .contains(string)
		}
	}
	
	func entities(
		for identifiers: [BusStop.ID]
	) async throws -> [BusStop] {
		BusStop.allStops.filter {
			identifiers.contains($0.id)
		}
	}
}
```

Conditionally show options based on other parameter with `@IntentParameterDependency`.

```swift
struct BusRouteQuery: EntityQuery {
	@IntentParameterDependency<ShowNextBus>(
		\.$busStop
	)
	var showNextBus
	
	func suggestedEntities() async throws -> [Route] {
		guard let showNextBus else { return [] }
		return Route.allRoutes.filter {
			$0.busStops.contains(showNextBus.busStop)
		}
	}
}
```

Limit the size of array parameters for different widget sizes.

```swift
struct ShowFavoriteRoutes: AppIntent, WidgetConfigurationIntent {
	// Pass an int for a fixed array size
	@Parameter(title: "Favorite Routes", size: 3)
	var routes: [Route]
	
	// Or pass an array for multiple widget sizes
	@Parameter(title: "Favorite Routes", size: [
		.systemSmall: 3, .systemLarge: 5
	])
	var routes: [Route]
}
```

Define which parameters are shown, and when, with `ParameterSummary`.  Use `When` to display conditionally based on widget size.

```swift
struct ShowFavoriteRoutes: AppIntent, WidgetConfigurationIntent {
	@Parameter(title: "Favorite routes", size: 3)
	var routes: [Route]
	
	@Parameter(title: "Include weather info")
	var includeWeatherInfo: Bool?
	
	static var parameterSummary: some ParameterSummary {
		When(widgetFamily: .equalTo, .systemLarge) {
			Summary("Show favorite \(\.$routes)") {
				\.$includeWeatherInfo
			}
		} otherwise: {
			Summary("Show favorite \(\.$routes)")
		}
	}
}
```

In this case, array `routes`and toggle `includeWeatherInfo` are shown, in that order, on a large widget, and only `routes` is shown on small widgets.

### Continue user activity
Show relevant information when the user taps on the widget.

- Call the `widgetConfigurationIntent` on the user activity to get the configuration Intent.
- Use that configuration data to display relevant information in the app.

```swift
WindowGroup {
	ContentView()
		.onContinueUserActivity("NextBus") { userActivity in
			let configuration: Configuration? =
				userActivity.widgetConfigurationIntent()
			
			// Navigate to the corresponding view
			navigate(
				toView: .busStopView,
				busStop: configuration?.busStop,
				route: configuration?.route
			)
		}
}
```

Use the `RelevantContext` APIs to suggest when to display the widget in a Smart Stack. The new `RelevantIntentManager` and `RelevantIntent` are more Swift-friendly and work seamlessly with App Intents.

```swift
let relevantIntents = gameTimes.map {
	RelevantIntent(SportsWidgetIntent(), "SportsWidget", .date(from: $0.start, to: $0.end))
}
RelevantIntentManager.shared.updateRelevantIntents(relevantIntents)
```

*See more about Relevance in "Build widgets for the Smart Stack on Apple Watch" from WWDC23.*
## Developer experience

### Framework support
In iOS 17 and Xcode 15, frameworks can now expose App Intents. This reduces code duplication. The `AppIntentsPackage` APIs can recursively import dependencies. By conforming types to the AppIntentsPackage protocol, both your app and frameworks can re-export metadata from other frameworks.

The example shown connects different frameworks in various snippets. Please watch from 15:45 to 17:00 for more.

`AppShortcutsProvider` and App Shortcuts can now be created in App Intents extensions, previously they could only be defined in the main app bundle. This helps code stay modular, and helps performance since the app doesn't have to launch in the background every time an App Shortcut runs.

### Static metadata extraction

All these features rely on static metadata extraction, which has been significantly improved in Xcode 15. Errors are shown directly during this process, so problems can be fixed faster.

### Continue execution

- The `ForegroundContinuableIntent` protocol continues the execution of an Intent even if that Intent was previously running in the background.
- Use `needsToContinueInForegroundError` to stop the Intent execution and require action to continue.
- Use `requestToContinueInForeground` to get a result from the person and use it to complete the App Intent's perform.

### Apple Pay

Initiate an Apple Pay transaction directly in the perform method with `PKPaymentRequest` and `PKPaymentAuthorizationController`.

```swift
struct RequestPayment: AppIntent {
	static var title: LocalizedStringResource = "Request Payment"

	func perform() async throws -> some IntentResult {
		let paymentRequest = PKPaymentRequest()
		// Configure your payment request
		let controller = PKPaymentAuthorizationController(
			paymentRequest: paymentRequest
		)
		guard await controller.present() else {
			return .result(dialog: "Unable to process payment")
		}
		return .result(dialog: "Payment Processed")
	}
}
```

## Shortcuts app integration

- App Intents have been used to build Shortcuts actions, for use with Siri and the Shortcuts app; as well as Focus Filters and the Action button on Apple Watch Ultra. In iOS 17, are now integrated with Interactive Live Activities, Widget Configuration and Interactivity, and SwiftUI.
- App Shortcuts now include support for Spotlight Top Hits and Automations.
- With all this integration, it's important to make sure parameter summaries are well written.
- If an App Intent is only for use inside an app or widget, set `isDiscoverable` to `false` to hide it elsewhere.
- For App Intents that run more slowly, make them conform to the `ProgressReportingIntent` protocol. Update the progress by setting `progress.totalUnitCount` and `progress.completedUnitCount`.
- `EntityPropertyQuery` is joined by the new `EnumerableEntityQuery` for integrating Find actions in Shortcuts. To use `EnumerableEntityQuery`, return all possible values for the entity in the `allEntities()` method, and Shortcuts and App Intents generates find actions automatically. Prefer `EnumerableEntityQuery` when the number of entities is small. When dealing with a large number of entities, use `EntityPropertyQuery`, and run the search on behalf of the user.
- `IntentDescription`, which is used to show action information in the Shortcuts UI, now has a property called `resultValueName` so we can adda more descriptive name for the output of the action.

*See more in the session "Spotlight your app with App Shortcuts" from WWDC23.*
