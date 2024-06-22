# SwiftUI on watchOS

SwiftUI allows a whole new world of possibilities when developing watchOS apps and notifications. From custom animations to providing an intuitive feel with Digital Crown haptics, SwiftUI helps you build exciting and immersive experiences for Apple Watch. See how easy it is to create custom elements with animations, embed gesture-driven animations within notifications, and learn about the enhanced debugging support to make watchOS app development faster than ever.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/219", purpose: link, label: "Watch Video (30 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



## watchOS 6

- Independent watchOS apps (decoupled from iOS apps)
- Extended run-time sessions
- Building experiences: Complications, Notifications, Siri, etc.
- Prioritize quick interactions


## Full Power of SwiftUI

- **Declarative syntax** Whole new UI framework, new fetures and APIs
- **Integration** Watchkit controllers with SwiftUI Views 
  - `InterfaceController` inherits `WKHostingController`

- **Lists** WatchOS flash cards app
- Keep model and List in sync using @ObservedObject
- Use `Command + Click` to bring up the inspector and use different contextual options while coding
- Use `.listStyle(.carousel)` to get the carousel effect while scrolling the list
- Swipe to delete, drag to reorder
- Use `.onMove` and `.onDelete` blocks to manage the movement and deletion of items in the list
- **Interactive Notifications** Timely and contextual info
  - Short look (Info from payload + App icon) Immediately upon wrist raise

- Long look (Scrolling interface with custom body and action buttons)
- `NotificationController` inherits `WKUserNotificationHostingController`
  - `didReceive` method allows us to extract info from notification
  - `body` property is re-evaluated after `didReceive` is called

- **Digital Crown** Series 4 watch can make use of haptic crown (e.g. workout app, custom timer)
  - Building following custom interfaces requires `.digitalCrowRotation` and `.focusable` modifier
  - Free scrolling interface (no concrete spots between elements)
  - binding (source of truth)
  - from
  - through

- Picking between discrete elements
  - binding (source of truth)
  - from
  - through
  - by (stride along which haptic feedback is provided)

- Moving around circles (not limited to either end of the sequence) 
  - binding (source of truth)
  - from
  - through
  - by (stride along which haptic feedback is provided)
  - sensitivity (how much rotation need to be applied to move from one element to the next)
  - isContinuous (don't stop at either limit of sequence)
