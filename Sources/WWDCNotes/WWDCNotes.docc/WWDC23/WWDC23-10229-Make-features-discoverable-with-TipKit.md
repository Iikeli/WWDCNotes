# Make features discoverable with TipKit

Teach people how to use your app with TipKit! Learn how you can create effective educational moments through tips. We’ll share how you can build eligibility rules to reach the ideal audience, control tip frequency, and strategies for testing to ensure successful interactions.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10229", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



- TipKit is a new framework that makes it easy to shop tips in your app (for feature discoverability)
- Designed with education in mind to teach about brand new / hidden feature or faster way to accomplish a task
- Available on iPhone, iPad, Mac, Apple Watch, and Apple TV (not in Vision Pro ?)

## Create a tip

- Conform custom type to `Tip` which has `title: Text` and `message: Text` requirements, `Text` view can be customized
- Recommended usage when tips are: actionable, instructional, and easy to remember

![][Recommended]

[Recommended]: ../../../images/notes/wwdc23/10224/Recommended.png

- Not recommended for: Promotional messages, Error messages, pure information that is not actionable, too complex text

![][NotRecommended]

[NotRecommended]: ../../../images/notes/wwdc23/10224/NotRecommended.png

- Call `TipsCenter.shared.configure` on app start (e.g. `init()` of `App`)
- `Tip` protocol optionally has an `asset: Image` to show an icon alongside tip
- `Tip` protocol also optionally has `actions: [Action]` where you can return `Tip.Action(id:title:)` instances for buttons (like "Learn More" or "Open Settings")
- 2 types of tip views:
	- `.popoverMiniTip(tip: ...)`: Appears over app's UI, no need to change app screen – on tvOS this is the only option
	- Inline views: Adjusts the app's UI to temporarily fit around it

## Eligibility rules

- 2 types of rules to decide when to show a tip:
	- Parameter-based rules: State and Bool comparisons, persistent, best suited for Swift value types
	- Event-based rules: User actions that must be performed before showing tip
- For parameter-based rule, use `@Parameter` on a static stored propertty like `static var isLoggedIn: Bool = false` of your custom `Tip` conforming type, and:
- Specify `#Rule(Self.$isLoggedIn) { $0 == true }` in a `rules: Predicate<RuleInput...>` computed property within your type
- For event-based rule, specify a static stored property like `static let appStarts: Event = Event<AppStartDonation>(id: "app-start")`, and:
- Specify `#Rule(Self.appStarts) { $0.count >= 3 }` in a `rules: Predicate<RuleInput...>` computed property within your type
- Don't forget to "donate" the event when it happens, like calling `.onAppear { CustomTip.appStarts.donate() }`

Example with both parameter-based and event-based rules combined:

![][CombinedRules]

[CombinedRules]: ../../../images/notes/wwdc23/10224/CombinedRules.png

- You can not only use `.count` on `Event` but also `.donations` which have a `date` property for more complex logic, like "3 times within last week"
- You can even specify your custom `DonationValue` types with additional details like `ID`s of your models or more

## Display and dismissal

- Display frequency of tips can be set so they appear at an ideal cadence, we don't want to show them all at once
- To show one tip per day, on app start use: `TipsCenter.shared.configure { DisplayFrequency(.daily) }` (or `.hourly`, custom `TimeInterval`, or `.immediate`)
- A specific `Tip` conformance can override frequency via computed property `options: [Option] { [.ignoresDisplayFrequency(true)] }`
- Call `customTip.invalidate(reason: .userPerformedAction)` to dismiss a tip that is presented or to never show a tip for an action the user has already performed
- Additionally, you can provide a max display count via `options: [Option] { [.maxDisplayCount(5)] }` computed property on a `Tip` type
- TipKit can sync tip status via iCloud to sync multi-device on same app (didn't explain how or if customizable)


## Test tips

- To inspect all tips without needing to satisfy the eligibility rules, perform `TipsCenter.showAllTips()` for testing
- You can also show certain tips via `TipsCenter.showTips([tip1, tip2, tip3])` by passing their IDs, or hide them via `.hideTips(...)`
- To hide all tips to focus on a new feature for example, call `TipsCenter.hideAllTips()`
- Use `TipsCenter.resetDatastore()` to reset persisted eligibility data to get same experience on each run during development
- All the above options can also be passed via launch arguments in Scheme Editor in Xcode:

![][LaunchArguments]

[LaunchArguments]: ../../../images/notes/wwdc23/10224/LaunchArguments.png
