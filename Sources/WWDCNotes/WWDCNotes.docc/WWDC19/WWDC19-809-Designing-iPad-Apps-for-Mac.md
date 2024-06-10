# Designing iPad Apps for Mac

Discover how you can create a great Mac experience with your iPad app. Learn about essential techniques for adapting your iPad app's layout and architecture for Mac, considerations for type and color, and how you can take advantage of macOS interfaces such as the menu bar, sidebar and window toolbar.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/809", purpose: link, label: "Watch Video (30 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- Turning an iPad app to a macOS app doesn’t take much effort
- Apps should be ported to mac only when it mac sense (no AR or Navigation apps for example)
- We get lots of things for free: split views, file browser, contextual menus, copy/paste
- No all UI designed for touch interfaces translate well to mac, buttons should be much smaller.
- Don’t take for granted that all the touch gestures are available for mac (all of them are if the user uses a trackpad tho).
- UI put on the bottom of the screen for reachability doesn’t make sense on the mac, every pixel is reachable the same in a mac window.
iOS text is sized down to 77% its original size on macOS. (e.g. default size 17 points becomes 13 points)
- Some macOS machines still use @1x 😱
- Tabs and other elements should be ported to the equivalent on mac (see more in link below)
- Lots of customized colors should not be used to macOS (e.g. a tabBar with a bright custom color as background), macOS should be more neutral (because it’s possible that the user has multiple windows open, even not from the same app, so we should let the user focus on the content as much as possible)
- We must do some work to support things like:
  - Toolbars
  - appIcon (the iOS version is not macOS-like, we should have a macOS version)
  - Contextual menus (be succinct in the actions names)
  - Menu bars
  - Touch bars (yes we get this even for iPad apps that run on MacOS)
  - macOS also has a gesture that iOS doesn’t have (at least by default): hover events! We should user this event if necessary/useful

Apple has a very nice overview on what steps to take in order to bring an iPadOS app to macOS in the iOS HIG [here](https://developer.apple.com/design/human-interface-guidelines/ios/overview/ipad-apps-for-mac/).
