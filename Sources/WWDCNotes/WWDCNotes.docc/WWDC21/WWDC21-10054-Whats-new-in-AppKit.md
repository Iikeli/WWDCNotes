# What's new in AppKit

Explore the latest advancements in Mac app development with AppKit. We’ll show how you can enhance your app’s design with new control features and SF Symbols 3, build powerful text experiences using TextKit 2, and harness the latest Swift features in your app.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10054", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



## Design & control updates

There are design updates for some system controls:

- popovers appear with an animation
- sliders smoothly glide into position when clicked
- smaller things like increased spacing between table sections or slightly wider toolbar buttons

#### Control tinting

Individual controls like buttons, segmented controls or sliders can have a custom tint. You can set it using these properties:

- [`NSButton.bezelColor`](https://developer.apple.com/documentation/appkit/nsbutton/2561000-bezelcolor)
- [`NSSegmentedControl.selectedSegmentBezelColor`](https://developer.apple.com/documentation/appkit/nssegmentedcontrol/2561002-selectedsegmentbezelcolor)
- [`NSSlider.trackFillColor`](https://developer.apple.com/documentation/appkit/nsslider/2560999-trackfillcolor)

These properties have been introduced in macOS Sierra for Touch Bar controls; in macOS Monterey they’re functional also for normal app controls.

This is useful for specific controls that need to have some kind of semantically meaningful color - e.g. a green “Accept Call” and a red “End Call” buttons in a video call app.

Avoid confusion with the default button if there is one in the same view, since it will also be colorful. Also, make sure to indicate purpose with more than just the color (using a clear label or an icon), since some of your users may not be able to distinguish buttons by color.

#### Push buttons

Push buttons no longer highlight using the accent color on click - they behave just like e.g. segmented controls in macOS Big Sur. This means you shouldn't make assumptions about how the highlight state looks (e.g. drawing white text over a button that should be blue when pressed, but will now be light gray).

Instead, check the interior background style `NSButtonCell.interiorBackgroundStyle`:

- `.normal` = colorless state
- `.emphasized` = colorful state

The old “regular square button” aka “bevel button” has now been refreshed as “Flexible push style button” and can be used as a variable height push button. It supports the same kind of configuration as a regular push button, so it can serve as a default button and can be tinted. Its corner radius and padding now match other button styles, and it can contain larger icons or multi-line text.

The vast majority of buttons should still use the standard fixed height push button - the variable height button is meant for special cases.


## Localizing keyboard shortcuts

Some keyboard shortcuts should be localized for different keyboard layouts, because in some layouts they may be hard or impossible to type, or it may make sense to adapt them for right-to-left languages. E.g. `Cmd + \` is not possible to type on the Japanese keyboard, which doesn’t have a backslash key.

AppKit can now handle this for you. In macOS Monterey, the system automatically remaps such shortcuts to different ones that are more natural on the given keyboard layout. Also, shortcuts like `Cmd + [` and `Cmd + ]` to go back and forward will be swapped in right-to-left languages (this applies to brackets, braces, parentheses and arrow keys).

You can opt out of this behavior using:

- [`NSMenuItem.allowsAutomaticKeyEquivalentMirroring`](https://developer.apple.com/documentation/appkit/nsmenuitem/3787555-allowsautomatickeyequivalentmirr) - for directional keys like brackets
- [`NSMenuItem.allowsAutomaticKeyEquivalentLocalization`](https://developer.apple.com/documentation/appkit/nsmenuitem/3787554-allowsautomatickeyequivalentloca) - turns off all key localization, including mirroring

If you really don’t want to use this feature at all, you can also disable it completely in your app by implementing the `NSApplicationDelegate` method: [`applicationShouldAutomaticallyLocalizeKeyEquivalents(_:)`](https://developer.apple.com/documentation/appkit/nsapplicationdelegate/3787553-applicationshouldautomaticallylo/).


## Update to SF Symbols

There is a new version available - SF Symbols 3:

- it expands capabilities of the SF Symbols app
- some symbols now have multiple layers that can be individually colored
- there is an updated format for custom symbols - it allows you to annotate distinct layers within an image

Big Sur had two coloring modes for symbols:

- traditional monochrome template style, drawing the whole symbol using one accent color
- a multicolor style that uses multiple colors that are predefined in the symbol itself

SF Symbols 3 in macOS Monterey adds two new rendering modes:

- "hierarchical" - uses a single tint color, but draws different layers of the image in an emphasized or deemphasized way (lighter or darker than the base color)
- "palette" - lets you assign each layer any custom color independently

APIs for the new rendering modes:

```swift
NSImage.SymbolConfiguration(hierarchicalColor: .red)
NSImage.SymbolConfiguration(paletteColors: […])
NSImage.SymbolConfiguration.preferringMulticolor()
```

#### Symbol variants

There are also new APIs for mapping between symbol variants, e.g. outline heart symbol ⭤ filled heart symbol, or variants with circles etc. This is useful e.g. when you’re building a picker control that uses outline icons for unselected states and filled variants of the same icons for the selected item.

To convert between variants, call e.g.:

```swift
baseImage.image(with: .fill)
```

There are constants for each kind of symbol variant, and you can combine multiple variants together (e.g. circle + fill).

See “Design and build SF Symbols” for more info.


## TextKit 2

macOS Monterey brings a huge update to the text system. TextKit is a great text engine with a long track record, used across all Apple systems. However, TextKit is a _linear_ text layout engine, which means it typesets a block of text from the beginning to the end. There are a lot of use cases where a _non-linear_ layout engine is more useful.

TextKit 2 always uses a non-linear layout system. This means it can perform layout on a more granular level, which allows it to avoid some unnecessary work. For example, when you’re looking at a middle fragment of a long document, a linear layout system needs to process all text from the beginning up to the given fragment in order to render it; a non-linear system can start at the nearest start of a paragraph.

The non-linear layout system also makes it easier to mix text with non-text elements, and improves performance for large documents.

TextKit 2 provides a lot of customization points, which allow you to extend its behavior. The new version coexists with TextKit 1, you can choose which engine to use for each view.

TextKit 2 has actually already been used in some system apps and controls in Big Sur.

See “[Meet TextKit 2](https://developer.apple.com/videos/play/wwdc2021/10061/)” for more info.


## New Swift features

Swift 5.5 introduces some important new features for managing concurrency: async/await and actors.

In AppKit, many asynchronous methods that return value through a completion handler now have variants that work with async/await:

```swift
@IBAction func pickColor(_ sender: Any?) {
  async {
    guard let color = await NSColorSampler().sample() else { return }
    textField.textColor = color
  }
}
```

The actor model is a great fit for a UI framework like AppKit where most APIs should be called on a single main thread. The macOS SDK now has a `@MainActor` property wrapper that marks all types that have to be accessed from the main thread. Classes such as `NSView`, `NSView/WindowController`, `NSApplication`, `NSCell`, `NSDocument` etc. are now marked with `@MainActor`.

Code running in the main actor can freely call methods on other main actor types. However, code that isn’t running on the main thread needs to use async/await to run code on a `@MainActor` type. This is enforced at the compiler level, which lets you avoid common errors that happen when mixing concurrency with UI code.

See “[Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)” and “[Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133)” for more info.

#### AttributedString

Swift 5.5 also adds a new value type [`AttributedString`](https://developer.apple.com/documentation/foundation/attributedstring/). It has type-safe attributes and a more swifty API for reading & writing attributes. You can easily convert between `AttributedString` and `NSAttributedString`.

See “[What’s new in Foundation](https://developer.apple.com/videos/play/wwdc2017/212/)” for more info.

#### Updating NSViews

There is a new Swift property wrapper which should reduce boilerplate around view properties.

Let’s say we have a custom view class like this:

```swift
class BadgeView: NSView {
  var fillColor: NSColor
  var shadow: NSShadow
  var scaling: NSImageScaling
  …
}
```

These properties will usually need to have a didSet which updates properties like [`needsDisplay`](https://developer.apple.com/documentation/appkit/nsview/1483360-needsdisplay/) or [`needsLayout`](https://developer.apple.com/documentation/appkit/nsview/1526912-needslayout) when they’re modified:

```swift
var fillColor: NSColor {
  didSet { needsDisplay = true }
}
```

The new `@Invalidating` attribute in [`NSView`](https://developer.apple.com/documentation/appkit/nsview) lets you easily specify which other view properties should be updated when the given property is modified:

```swift
@Invalidating(.display) var fillColor: NSColor
```

Properties that can be invalidated include: display, layout, constraints, intrinsic content size, restorable state. The marked property needs to be [`Equatable`](https://developer.apple.com/documentation/swift/equatable/), since AppKit checks if the value was actually changed before triggering a view update.

You can extend the invalidation system by conforming to `NSViewInvalidating` protocol.


## Shortcuts

iOS Shortcuts are now available on the Mac.

Shortcuts appear in all the places where you can access services today - if your app supports services, it will also support Shortcuts. AppKit decides which shortcuts are available at the given place by checking the responder chain - it asks each responder whether it can provide or receive the type of data used by each shortcut. The types of data are represented by [`NSPasteboard.PasteboardType`](https://developer.apple.com/documentation/appkit/nspasteboard/pasteboardtype) (usually a UTI).

To support shortcuts in a given responder object, implement the method:

```swift
func validRequestor(forSendType sendType: NSPasteboard.PasteboardType?,
                              returnType: NSPasteboard.PasteboardType?) -> Any
```

In that method return an instance of a type implementing [`NSServicesMenuRequestor`](https://developer.apple.com/documentation/appkit/nsservicesmenurequestor/) (usually the same object):

```swift
protocol NSServicesMenuRequestor {
  func writeSelection(to pasteboard: NSPasteboard,
                              types: [NSPasteboard.PasteboardType]) -> Bool

  func readSelection(from pasteboard: NSPasteboard) -> Bool
}
```


## Siri Intents

You can now use Siri Intents in a Mac app by adding an Intents Extension. You can also return an intents handler from the application delegate:

```swift
protocol NSApplicationDelegate {
  optional func application(_ application: NSApplication,
                        handlerFor intent: INIntent) -> Any?
}
```

The returned object should conform to an appropriate intent handler protocol, depending on the intent type.
