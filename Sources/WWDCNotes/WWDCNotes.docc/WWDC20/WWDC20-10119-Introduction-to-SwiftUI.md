# Introduction to SwiftUI

Explore the world of declarative-style programming: Discover how to build a fully-functioning SwiftUI app from scratch as we explain the benefits of writing declarative code and how SwiftUI and Xcode can combine forces to help you build great apps, faster.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10119", purpose: link, label: "Watch Video (54 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



This session is an introduction on how to develop apps using SwiftUI. 

> If you're totally new to SwiftUI, I suggest you to watch the talk to get acquaint with the preview canvas and other important interactions that really don't translate well in a written note.

Some takeaway for everyone:

- `⌘ + click` on a SwiftUI component (either in code or in the preview) to have options such as:
  - `Embed in HStack`
  - `Show SwiftUI Inspector` (use `⌃ + ⌥ + click` for jumping to this option)
  - and more

> Alternatively, place your cursor on the element and use the `⌘ + ⇧ + A` shortcut to invoke the actions menu, note that SwiftUI-related actions show up only when the preview canvas is displayed.

- `⌘ + ⌥ + P` to resume a preview

- Towards the end of the video it is shown how to use the new [`.toolbar`][toolbarDoc] modifier (iOS 14+, macOS 11+, watchOS 7+), which adds items to either a toolbar or navigation bar (depending on the context).

- We can now change the environment (like color scheme, size category, ...) of a preview directly via its new toolbar:

![][previewImage]

[toolbarDoc]: https://developer.apple.com/documentation/swiftui/view/toolbar(content:)

[previewImage]: WWDC20-10119-preview
