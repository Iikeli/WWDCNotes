# Introducing Desktop-class Browsing on iPad

iOS 13 brings desktop-class browsing to iPad. With blazing-fast performance, industry-leading security, and modern desktop features, Safari on iPad supports the latest web standards designed and automatically adapts desktop sites and web apps to touch in order to deliver a rich browsing experience. Learn how your site or embedded WebView can take advantage of powerful new features and coding best practices to deliver a best-in class user experience for iPad.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/203", purpose: link, label: "Watch Video (49 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- New Features: **Download Manager**, **Keyboard Shortcuts**, **Full Page Zoom**, ...
- Websites designed to show lots of information, e.g. GitHub, are **zoomed** to make use of iPad's full width
- Hovering with the mouse on desktop now feels natural on an iPad
- Speaker covers the topics `Link following`, `Web Browser`, `Hybrid Apps` and `Authentication` for WKWebView apps
- Prevent horizontal scrolling in WKWebView: Compile with iOS 13 SDK and set `webViewConfiguration.applicationNameForUserAgent = "Version/1.0 MyBrowser/1.0"` (user agent auto-completed by Web Kit)
- Set `preferredContentMode = WKWebpagePreferences.ContentMode.[recommended, mobile, desktop]` 
- Demo: "Shiny Browser" iPad App
  - it is now easier to implement toggles between mobile/desktop websites by leveraging WKWebViewDelegate

- Optimization potential for **web developers**
  - **Pointer Events**
    - Adopt by `if (window.PointerEvent) element.addEventListener("pointermove", updateInteraction); else element.addEventListener("mousemove", updateInteraction);`
    - Cancel default web browser behavior by `event.preventDefault();` and additional the CSS variant `touch-action: none;` (more granualar on disabling events)

- 
  - **Hover**
    - Provide alternative way to access hover content
    - Avoid two tap for common interactions
    - Keep hover snappy

- 
  - **Accelerated Scrolling**
    - works on all frames, not only the main frame
    - Frameworks don't need workarounds to simulate acceleration of sub frames anymore

- 
  - **View Port and Text Sizing**
    - Some webpages are build in fixed width, wider than an iPad
    - Now on iPadOS the paremeter `width=device-width` is ignored for websites laying out wider than iPad width. The site is zoomed to max iPad width.
    - Add content parameter `shrink-to-fit=no` to the viewport meta tag to prevent auto shrinking
    - Implement proper responsive websites to get around the new iPadOS auto adjustments

- 
  - **Visual Viewport API**
    - Visual VP is the area of your webpage visible to the user, e.g. between top and top of the iPadOS keyboard
    - react to changes, e.g. keyboard will show, by `visualViewport.addEventListener("resize", visualSizeChanged);`

- 
  - **Streaming Video**
    - Media Source Extensions now available on iPad

- **Best Practices**
  - Build one responsive website instead of two for mobile/desktop
  - Use feature detection instead of switching on user agent
  - Don't use Flash - support will be dropped in Safari on all platforms
  - Let users decide if they want audio
  - Remember that some desktop browsers don't have mice
  - Use built-in APIs
