# What's new in watchOS 8

watchOS 8 brings all-new opportunities to keep people up to date on their watch face. With new APIs for the Always-On Retina display and updating complications from Bluetooth devices and background delivery of HealthKit data, it's never been easier to keep your app up to date. Learn about region-based user notifications to leverage location in your app. Explore all the new enhancements to SwiftUI and watchOS that will get you excited to build your next Watch app.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10002", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Always-on display dimmed state
- Supported on series 5 and 6
- in watchOS 7, the always-on state showed your app’s UI blurred with the system time overlaid
- when you rebuild your app with the watchOS 8 SDK, your app’s UI will now be shown in a dimmed state instead, and is immediately interactive if someone taps the UI
- You can get notified of of this dimmed state via a new SwiftUI environment property  `isLuminanceReduced`.
- You can respond to this state by highlighting information that’s especially important, and hiding information that should stay private

- In the watch simulator, you click the lock button in the simulator window to simulate a wrist-down event (triggering the dimmed state)

Guidance:

- the transition from active state to the always-on state should feel seamless
- don’t drastically alter the UI or reorganize information
- dim non-essential information and elements, to give more prominence to the piece of information you want to stay highlighted and visible
- if your UI has large elements that are filled with color or imagery, you may want to reduce those elements to be represented with a stroke or dimmed color
- redact or remove sensitive information
- reset animations
- use SwiftUI's `TimelineView` to tell watchOS 8 that your inactive app needs to update its UI

UI Updates:
- Apps with an ongoing, active session (e.g. workout or audio session) can update your UI up to once per second
- other apps up to once per minute
- assume your app is visible longer than two minutes

## HealthKit data

- background delivery of HealthKit data to Watch apps
- new results come up to once per hours
- if you have an active complication, up to four times per hour
- all received results count against background app refresh budget

Update frequency:

- immediate for critical data types (fall events, low blood oxygen saturation, heart rate events)
- hourly or longer for other events

## Bluetooth

- from watchOS 4, Bluetooth devices can connect directly to Apple Watch and make use of Core Bluetooth.
- from watchOS 8, Bluetooth devices can connect during background app refresh opportunities that your app’s complications get when they’re on the active watch face
  - this means that your app’s complications can stay up to date with your Bluetooth device and display updated information throughout the day.
  - Background app refresh gives your app’s complications that are on the active watch face up to four opportunities per hour to connect and update.
  - these opportunities will count against your app’s overall background app refresh budget

Guidance:
- connect and process your data within a very short period of time
- new expiration handler on `WKRefreshBackgroundTask`, letting you know when you're about to run out of time

## Location

- new in watchOS 8, region-based user notifications
  - works similarly to iOS:
    - deliver pre-created local notifications
    - "when in use" location permission required
    - limit the number of regions to only include important POIs near someone or locations they’ve shown explicit intent for

- new Location button, for one-time location authorization without going through authorization prompts each time it’s tapped

- always-on altimeter API, no need location access

## More enhancements

- respiratory rate API
- assistive touch
- large accessibility text size
- (Xcode 12.5+) Unit testing and UI testing
- Large titles
- text input improvements:
  - Scribble or Dictation preference is preserved per app
  - quick access to changing between input types while entering text
- SwiftUI's `.searchable` API
- Swipe actions on `List`
- Buttons improvements:
  - new roles parameter to let the system know how to present and handle specific button types (e.g. destructive buttons)
  - new `controlProminence` view modifier, which will give those buttons an additional haptic when tapped.
- SwiftUI `Canvas` for rich programmatic drawing
