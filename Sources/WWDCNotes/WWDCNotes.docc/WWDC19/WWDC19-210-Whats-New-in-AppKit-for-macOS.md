# What’s New in AppKit for macOS

Learn about the latest APIs in AppKit and associated frameworks. Get an overview of the enhancements coming in macOS Catalina to help you save time, take advantage of the latest hardware, and add polish to your application.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/210", purpose: link, label: "Watch Video (37 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



## New frameworks

AppKit is no longer the only framework to build Mac apps with - you also have an option now to use UIKit via Catalyst and the new SwiftUI framework (although you will probably still need to use at least a little bit of AppKit in both cases).


## Colors

- new system colors: [`NSColor.systemTeal`](https://developer.apple.com/documentation/appkit/nscolor/3174921-systemteal) and [`systemIndigo`](https://developer.apple.com/documentation/appkit/nscolor/3174920-systemindigo) (dynamic colors, specific color values depend on light/dark appearance)
- [`NSColor`](https://developer.apple.com/documentation/appkit/nscolor) uses tagged pointers - this means that color data is now stored in the pointer itself and allocating NSColor objects becomes very cheap
- [`NSColorSampler`](https://developer.apple.com/documentation/appkit/nscolorsampler) - a magnifier tool for picking a color from somewhere on the screen
- new [`NSColor`](https://developer.apple.com/documentation/appkit/nscolor) initializer that lets you return different colors based on appearance:

```swift
let color = NSColor(name: "userWidgetColor") { appearance in
  switch appearance.bestMatch(from: [.aqua, .darkAqua]) {
  case .darkAqua:
    return darkUserWidgetColor
  case .aqua, .default:
    return lightUserWidgetColor
  }
}
```


## Screens

- [`NSScreen.localizedName`](https://developer.apple.com/documentation/appkit/nsscreen/3228043-localizedname) now returns e.g. “Thunderbolt Display”

### Extended dynamic range:

Computer screens are very bright these days and most of the time aren't used at full brightness. This means that at lower brightness levels we can use the monitor's capability to show brighter pixels than what 100% white means at that brightness level, and provide an extended dynamic range, i.e. color values of more than 1.0.

This feature has been available in macOS since 10.11:

  - [`CAMetalLayer.wantsExtendedDynamicRangeContent`](https://developer.apple.com/documentation/quartzcore/cametallayer/1478161-wantsextendeddynamicrangecontent) - enables dynamic range in this layer
  - [`NSScreen.maximumExtendedDynamicRangeColorComponentValue`](https://developer.apple.com/documentation/appkit/nsscreen/1388362-maximumextendeddynamicrangecolor) - tells you the maximum color component value (e.g. 1.3), if some content on the screen is using EDR

New APIs in 10.15:

  - [`NSScreen.maximumPotentialExtendedDynamicRangeColorComponentValue`](https://developer.apple.com/documentation/appkit/nsscreen/3180381-maximumpotentialextendeddynamicr) - tells you the maximum component value even if it’s not rendering in EDR mode at the moment, so that you can make some decisions in advance before you enable it
  - [`NSScreen.maximumReferenceExtendedDynamicRangeColorComponentValue`](https://developer.apple.com/documentation/appkit/nsscreen/3180382-maximumreferenceextendeddynamicr) - maximum usable value on reference screens like the new Pro Display XDR

### Finding the current screen in Metal:

Previously, in order to find the [`MTLDevice`](https://developer.apple.com/documentation/metal/mtldevice) for the current screen in Metal you had to do something like this:

```swift
let preferredDevice = CGDirectDisplayCopyCurrentMetalDevice(
    self.window?.screen?.deviceDescription["NSScreenNumber"]
)
```

In 10.15, there's a new property [`CAMetalLayer.preferredDevice`](https://developer.apple.com/documentation/quartzcore/cametallayer/3175021-preferreddevice) that returns the MTLDevice for the current screen

- also [`MTKView.preferredDevice`](https://developer.apple.com/documentation/metalkit/mtkview/3181987-preferreddevice)


## Text & Fonts

[`NSTextView.usesAdaptiveColorMappingForDarkAppearance`](https://developer.apple.com/documentation/appkit/nstextview/3237223-usesadaptivecolormappingfordarka)

- automatically updates colors for light/dark appearance
- enable this if it's more important for rich text to be readable regardless of appearance than to show the document in an original unchanged form

Almost all of the `NSText*` related classes support secure coding now.

### Spell checking:

- [`NSSpellChecker`](https://developer.apple.com/documentation/appkit/nsspellchecker) - old API for managing spell checking (since OS X 10.0!)
- [`NSTextCheckingController`](https://developer.apple.com/documentation/appkit/nstextcheckingcontroller) - new API built as a successor of NSSpellChecker
  - used in UIKit, WebKit and AppKit
  - you can add support for it to any view by implementing the [`NSTextCheckingClient`](https://developer.apple.com/documentation/appkit/nstextcheckingclient) protocol
  - also does grammar checks, data detection, autocorrection

### Font descriptors:

[`NSFontDescriptor.withDesign(.monospaced / .rounded / .serif)`](https://developer.apple.com/documentation/appkit/nsfontdescriptor/3152380-withdesign)

- lets you choose a different variant of the same font, if available

### Scaling fonts between macOS & iOS:

If your app opens rich text documents created on another platform, you will notice that fonts at the same point size will look different, because Mac and iOS devices use very different screen densities. There are now new [`NSAttributedString`](https://developer.apple.com/documentation/foundation/nsattributedstring) APIs that let you scale fonts in documents coming from the other platform to a more appropriate size:

- [`NSAttributedString.DocumentAttributeKey.sourceTextScaling`](https://developer.apple.com/documentation/foundation/nsattributedstring/documentattributekey/3294212-sourcetextscaling) and [`textScaling`](https://developer.apple.com/documentation/foundation/nsattributedstring/documentattributekey/3294215-textscaling)
- [`NSAttributedString.DocumentReadingOptionKey.sourceTextScaling`](https://developer.apple.com/documentation/foundation/nsattributedstring/documentreadingoptionkey/3294213-sourcetextscaling) and [`targetTextScaling`](https://developer.apple.com/documentation/foundation/nsattributedstring/documentreadingoptionkey/3294214-targettextscaling)

### Text hyphenation:

Previously, you could configure hyphenation for each paragraph using [`NSParagraphStyle`](https://developer.apple.com/documentation/uikit/nsparagraphstyle). Now, this can be enabled for the whole container using [`NSLayoutManager.usesDefaultHyphenation`](https://developer.apple.com/documentation/uikit/nslayoutmanager/3180380-usesdefaulthyphenation).


## Toolbars

[`NSToolbarItem.isBordered`](https://developer.apple.com/documentation/appkit/nstoolbaritem/3237224-isbordered) - an easier way to make standard toolbar buttons:

- previously, you had to manually create an [`NSButton`](https://developer.apple.com/documentation/appkit/nsbutton), configure it appropriately and add it as a custom view to the toolbar item
- now, you can just create an `NSToolbarItem` with an image and set `isBordered = true`
- needs to be done in code though, no setting on the storyboard yet
- this also allows you to make use of NSToolbarItem's built-in functionality like automatic enabling/disabling

[`NSToolbarItem.title`](https://developer.apple.com/documentation/appkit/nstoolbaritem/3237225-title):

- allows you to make toolbar items that are buttons with a text label instead of an icon
- note: this is different than the item label, which appears below the button

[`NSToolbarItemGroup`](https://developer.apple.com/documentation/appkit/nstoolbaritemgroup):

- new convenience initializers that create a group with given item labels/icons in one line
- can display items as a segmented control which collapses into a popup or pulldown menu when there isn't enough space (use the new initializers for this)

[`NSMenuToolbarItem`](https://developer.apple.com/documentation/appkit/nsmenutoolbaritem):

- a button toolbar item that shows a pulldown menu
- the menu can be any [`NSMenu`](https://developer.apple.com/documentation/appkit/nsmenu), with separators, nested menus etc.


## Touch Bar

[`NSTouchBar.isAutomaticCustomizeTouchBarMenuItemEnabled`](https://developer.apple.com/documentation/appkit/nstouchbar/3228044-isautomaticcustomizetouchbarmenu)

- lets you enable/disable the "Customize Touch Bar…" entry in the Edit menu
- [`NSApplication`](https://developer.apple.com/documentation/appkit/nsapplication) already has such property, but this is useful if you want to avoid referencing NSApplication in the Touch Bar related code, or if you don't have access to it (e.g. Catalyst apps don't)

[`NSStepperTouchBarItem`](https://developer.apple.com/documentation/appkit/nssteppertouchbaritem):

- a new item type for selecing an item from a list, with left/right arrows

[`NSSliderTouchBarItem`](https://developer.apple.com/documentation/appkit/nsslidertouchbaritem):

- adds [`minimumSliderWidth`](https://developer.apple.com/documentation/appkit/nsslidertouchbaritem/3237222-minimumsliderwidth), [`maximumSliderWidth`](https://developer.apple.com/documentation/appkit/nsslidertouchbaritem/3237221-maximumsliderwidth) properties for defining min/max width
- previously you could achieve this using AutoLayout constraints, but it was less convenient


## Other controls

[`NSSwitch`](https://developer.apple.com/documentation/appkit/nsswitch):

- a new NSControl that looks like [`UISwitch`](https://developer.apple.com/documentation/uikit/uiswitch) on iOS
- not a replacement for checkbox
- avoid using it for small things and in large numbers, just one for some general mode switch, a "master toggle", like the Time Machine on/off switch

[`NSCollectionView`](https://developer.apple.com/documentation/appkit/nscollectionview):

- added compositional layout and diffable data source, same as on iOS


## Storyboards

Storyboards now let you use custom VC initializers for injecting dependencies:

```swift
@IBSegueAction func showFoo(_ coder: NSCoder) -> NSViewController {
    return MyViewController(params...)
}
```

This lets you use initializers in view controller classes that require passing any data the view controller needs, while still using storyboard segues to connect view controllers on the storyboard. 


## AutoLayout

Controls like text labels, buttons with titles etc. calculate their intrinsic content size that lets the AutoLayout system scale the whole window layout depending on the text contents of those controls. However, in some scenarios (e.g. inside an [`NSGridView`](https://developer.apple.com/documentation/appkit/nsgridview)) you want all controls to have their size set externally, and the control's intrinsic size is ignored. The calculations to determine the intrinsic size are still performed though in such case.

Now you can manually disable them (as an optimization) by setting these to false if you know the result will not be used in the layout calculations:

- [`NSView.isHorizontalContentSizeConstraintActive`](https://developer.apple.com/documentation/appkit/nsview/3353053-ishorizontalcontentsizeconstrain)
- [`NSView.isVerticalContentSizeConstraintActive`](https://developer.apple.com/documentation/appkit/nsview/3353054-isverticalcontentsizeconstrainta)


## NSResponder

There was a possible source of bugs before if you captured an instance of a UI-related class in a block in such a way that it could be later deallocated on a background thread:

```swift
let label = NSTextField(labelWithString: "...")

dispatch_async(dispatch_get_global_queue(0, 0)) {
    // ...

    dispatch_async(dispatch_get_main_queue()) {
        label.value = "done"
    }
}
```

If the outer block is released after the inner block (which you have no control over), then at that point the label object is deallocated on the background thread, and this could cause various hard to debug issues. This is now solved in 10.15 - the SDK guarantees that UI objects will be deallocated on the main thread in such scenario.


## Privacy & security

- Recording the screen will now require asking the user for permission
  - this doesn't apply to the new NSColorSampler control mentioned earlier, which runs out of process
- Open and save panels are also now always out-of-process, even for non-sandboxed apps
  - you may run into problems if you were somehow subclassing or customizing them in a non-standard way


## NSWorkspace

Added new asynchronous versions of existing methods for opening one or more URLs or applications. You can configure some aspects of opening using [`NSWorkspace.OpenConfiguration`](https://developer.apple.com/documentation/appkit/nsworkspace/openconfiguration), e.g.:

- if the user can control the process of opening a URL/app
- if the opened app or document is added to the "Recents" menu
- if the launched app is hidden on launch


## iPad & Sidecar

With the Sidecar feature, an iPad can function as an additional screen for the Mac, and also as a drawing tablet.

Tablet drawing events are sent as normal mouse events, with [`.tabletPoint`](https://developer.apple.com/documentation/appkit/nsevent/eventsubtype/1642479-tabletpoint) subtype. They also provide pressure data (although unlike on iOS, you can't register to get retroactive updates for previous pressure).

Switching mode on the Apple Pencil by double-tapping can be handled by another event with event type [`.changeMode`](https://developer.apple.com/documentation/appkit/nsevent/eventtype/changemode), also an [`NSResponder`](https://developer.apple.com/documentation/appkit/nsresponder) in the responder chain can handle that event through a new [`changeMode(withEvent:)`](https://developer.apple.com/documentation/appkit/nsresponder/3237219-changemode) method.

You can also listen to that event outside of the responder chain by using the "local event monitor" API:

```swift
let monitorId = NSEvent.addLocalMonitorForEvents(matching: .changeMode) { (event) -> NSEvent? in
    switchTool()
    return event
}
```


## Foundation additions

### View geometry:

New data types for specifying view geometry:

- [`NSDirectionalRectEdge`](https://developer.apple.com/documentation/uikit/nsdirectionalrectedge)
- [`NSDirectionalEdgeInsets`](https://developer.apple.com/documentation/uikit/nsdirectionaledgeinsets)
- [`NSRectAlignment`](https://developer.apple.com/documentation/uikit/nsrectalignment)

These use the terms leading & trailing on the horizontal axis instead of e.g. "minX" or "maxX", so they work better with right-to-left languages.

### Formatters:

[`NSRelativeDateFormatter`](https://developer.apple.com/documentation/foundation/relativedatetimeformatter) - allows you to format dates as e.g. "1 month ago" or "last week"

[`NSListFormatter`](https://developer.apple.com/documentation/foundation/listformatter) - formats lists of things, adding commas properly (uses nested formatters from [`.itemFormatter`](https://developer.apple.com/documentation/foundation/listformatter/3130988-itemformatter) to format specific items)


## Changes in extensions

- non-UI file provider action extension
- Network Extensions, DriverKit and Endpoint Security replace old kernel extensions
