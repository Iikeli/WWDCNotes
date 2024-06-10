# Customize and resize sheets in UIKit

Discover how you can create a layered and customized sheet experience in UIKit. We’ll explore how you can build a non-modal experience in your app to allow interaction with content both in a sheet and behind the sheet at the same time. We’ll also take you through sheet size customization, revealing or hiding grabber controls, and adapting between popovers and customized sheets in your app.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10063", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Session sample code](https://developer.apple.com/documentation/uikit/uiviewcontroller/customize_and_resize_sheets_in_uikit)

- (From iOS 15) new customization options for sheets
- New support for a medium detent (and more)
  - allows to create a vertically resizable sheet that only covers half the screen
  - the dimming view can be removed, allowing to build a non-modal UI where the user can interact with content behind the sheet while the sheet is presented

## `UISheetPresentationController`

- A sheet is an instance of a new `UIPresentationController` subclass called `UISheetPresentationController`
- all customization options are exposed as properties on this class
- you get an instance of `UISheetPresentationController` by reading the `sheetPresentationController` property on a view controller before you present it
- This property will be non-nil as long the view controller's `modalPresentationStyle` is form sheet or page sheet (default)

```swift
if let sheet = viewController.sheetPresentationController {
  // Customize the sheet
}
present(viewController, animated: true)
```

## Detents

- A detent is a height where a sheet naturally rests 
- defined as a fraction of the fully expanded sheet frame 
- this is the fully expanded frame on an iPhone and iPad: 

![][detents]

- two system-defined detents: 
  - `.medium()`, which is about half of a sheet's full height
  - `.large()`, which is the height of a fully expanded sheet

- specify which detents you want a sheet to support via the `UISheetPresentationController`'s `detents` property:

```swift
if let sheet = viewController.sheetPresentationController {
  sheet.detents = [.large()] // This is the default value
}
present(viewController, animated: true)
```

## Scroll behavior

- If we have a scroll view within the sheet, scrolling on that view  will also expand the sheet
- we can opt out of this behavior, meaning that the user can only explicitly resize the sheet by dragging from the bar, by setting an extra property:

```swift
if let sheet = viewController.sheetPresentationController {
  // ...
  sheet.prefersScrollingExpandsWhenScrolledToEdge = false // 👈🏻
}
present(viewController, animated: true)
```

## Programmatically selecting a detent

We can change the current selected detent by setting the `selectedDetentIdentifier` property:

```swift
if let sheet = viewController.sheetPresentationController {
  sheet.selectedDetentIdentifier = .medium
}
```

By default it has no animation, to add one, wrap it around a `sheet.animateChanges` block:

```swift
if let sheet = viewController.sheetPresentationController {
  sheet.animateChanges {
    sheet.selectedDetentIdentifier = .medium
  }
}
```

## Dimming View

- by default all detents are dimmed
- we can change that by setting from which detent the sheet will show the dimming view

```swift
if let sheet = viewController.sheetPresentationController {
  // ...
  sheet.smallestUndimmedDetentIdentifier = .medium
}
```

This property also allows the user to interact with both the content in the sheet and with the content outside of the sheet.

## Keyboard avoidance

- medium height sheets support automatic keyboard avoidance: 
  - the sheet grows automatically to account for the keyboard
  - when the keyboard dismisses, the sheet automatically collapses back down

## Landscape

- new alternate landscape appearance where sheets are only attached to the screen at their bottom edge (no longer full screen on landscape)

![][landscape-1]

```swift
if let sheet = viewController.sheetPresentationController {
  // ...
  sheet.prefersEdgeAttachedInCompactHeight = true
}
```

- this will always give you a sheet that is as wide as the safe area
- if you'd like a sheet whose width follows the presented view controller `preferredContentSize`:

![][landscape]

```swift
if let sheet = viewController.sheetPresentationController {
  // ...
  sheet.prefersEdgeAttachedInCompactHeight = true
  sheet.widthFollowsPreferredContentSizeWhenEdgeAttached = true
}
```

## More Appearance customization

- We can display a default grabber:

```swift
if let sheet = viewController.sheetPresentationController {
  // ...
  sheet.prefersGrabberVisible = true
}
```

- We can customize the corner radius of the sheet:

```swift
if let sheet = viewController.sheetPresentationController {
  // ...
  sheet.preferredCornerRadius = 24.0
}
```

- Note that the system will keep stacked corners looking consistent: if the sheet with different radius expands to full height, the root sheet (and other stacked ones) will have a corner radius to match

## Adaption from popover

- on regular sizes Apple suggests to show a popover instead of a sheet, this popover adapts to a sheet in compact sizes:

```swift
viewController.modalPresentationStyle = .popover // popover by default
if let popover = picker.popoverPresentationController { // reach for popoverPresentationController instead of sheetPresentationController
  popover.barButtonItem = sender // the source of the popover

  let sheet = popover.adaptiveSheetPresentationController // sheet adoption
  // 👇🏻 normal sheet configuration
  sheet.detents = [.medium(), .large()]
  sheet.prefersScrollingExpandsWhenScrolledToEdge = false
  sheet.smallestUndimmedDetentIdentifier = .medium
}
present(viewController, animated: true)
```

[detents]: WWDC21-10063-fullSheet
[landscape]: WWDC21-10063-landscape
[landscape-1]: WWDC21-10063-landscape-1