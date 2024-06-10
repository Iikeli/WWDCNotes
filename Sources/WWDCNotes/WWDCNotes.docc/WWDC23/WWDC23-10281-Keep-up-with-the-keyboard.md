# Keep up with the keyboard

Each year, the keyboard evolves to support an increasing range of languages, sizes, and features. Discover how you can design your app to keep up with the keyboard, regardless of how it appears on a device. We’ll show you how to create frictionless text entry and share important architectural changes to help you understand how the keyboard works within the system.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10281", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Spencer Lewson, Keyboard Engineer

## Out of process keyboard

Taking the keyboard out of the app process:

- improves privacy and security
- improves memory management as there is now only one keyboard
- allows for new exciting functionality in the future

> Prior to iOS 17, the keyboard's views and logic were executing within your apps process. But new on iPhone on iOS 17, the keyboard has been moved to its own process, running almost completely outside your app.

> For the majority of apps, these changes are completely transparent and require no adoption from you. Though, aspects of this new asynchronous approach now exist throughout the entire keyboard and can introduce some slight differences in timing.
> 
> So if your app is especially sensitive to the timing of text entry, selection changes, or any other text related operations, you should keep this new architecture in mind.

## Design for the keyboard

With Stage Manager, now you can always just move up your views, the height of the keyboard.

### Keyboard Layout Guide

Align to the `keyboardLayoutGuide` to automatically adjust for the keyboard:

```swift
view.keyboardLayoutGuide.topAnchor.constraint(equalTo: textView.bottomAnchor).isActive = true
```

#### Default behaviors

- Follows the keyboard when onscreen and docked
- Uses bottom safe area when offscreen or undocked
- Follows *dismiss* gesture when it intersects

*See more in the session "Your Guide to Keyboard Layout" from WWDC21.*

#### Customizing the guide

- `followsUndockedKeyboard` (defaults to false): Set to `true` to always follow the keyboard when undocked from the bottom.
- **NEW** `usesBottomSafeArea` (defaults to true): Set to `false` to avoid using the bottom safe area when undocked or offscreen.
- **NEW** `keyboardDismissPadding`: Used to add padding above the keyboard for the *scroll to dismiss* gesture.

Example of using `usesBottomSafeArea` to create keyboard and text view aligned with safe area:

```swift
view.keyboardLayoutGuide.usesBottomSafeArea = false
textField.topAnchor.constraint(equalToSystemSpacingBelow: backdrop.topAnchor, multiplier: 1.0).isActive = true
view.keyboardLayoutGuide.topAnchor.constraint(greaterThanOrEqualToSystemSpacingBelow: textField.bottomAnchor, multiplier: 1.0).isActive = true
view.keyboardLayoutGuide.topAnchor.constraint(equalTo: backdrop.bottomAnchor).isActive = true
view.safeAreaLayoutGuide.bottomAnchor.constraint(greaterThanOrEqualTo: textField.bottomAnchor).isActive = true
```

Example of using `keyboardDismissPadding` to start the *keyboard dismiss gesture* when the touch intersects with the textfield:

```swift
var dismissPadding = aboveKeyboardView.bounds.size.height
view.keyboardLayoutGuide.keyboardDismissPadding = dismissPadding
```

### SwiftUI

Most of the stuff around keyboards are automatically handled by SwiftUI.

### Notifications

> Before SwiftUI and keyboard layout guide, the only way to integrate your keyboard into your application was to listen for a set of keyboard notifications– willShow, didShow, willHide, didHide– and then manually adjust your layout based on the frame and animation information in the notification.

- Can still be used
- With Stage Manager it doesn't always work 100 % of the time
- With the new out of process keyboard, the timings can be different and notifications can be delayed

```swift
func handleWillShowOrHideKeyboardNotification(notification: NSNotification) {
    // Retrieve the UIScreen object from the notification (Added iOS 16.1)
    guard let screen = notification.object as? UIScreen else { return }

    // Determine if the notification’s screen corresponds to your view’s screen
    guard(screen.isEqual(view.window?.screen)) else { return }

    // Calculate intersection with keyboard
    let endFrameKey = UIResponder.keyboardFrameEndUserInfoKey

    // Get the ending screen position of the keyboard
    guard let keyboardFrameEnd = userInfo[endFrameKey] as? CGRect else { return }

    let fromCoordinateSpace: UICoordinateSpace = screen.coordinateSpace
    let toCoordinateSpace: UICoordinateSpace = view

    // Convert from the screen coordinate space to your local coordinate space
    let convertedKeyboardFrameEnd = fromCoordinateSpace.convert(keyboardFrameEnd, to: toCoordinateSpace)

    // Calculate offset for view adjustment
    var bottomOffset = view.safeAreaInsets.bottom
            
    // Get the intersection between the keyboard's frame and the view's bounds
    let viewIntersection = view.bounds.intersection(convertedKeyboardFrameEnd)
            
    // Check whether the keyboard intersects your view before adjusting your offset.
    if !viewIntersection.isEmpty {
        // Set the offset to the height of the intersection
        bottomOffset = viewIntersection.size.height
    }

    // Use the new offset to adjust your UI
    movingBottomConstraint.constant = bottomOffset
  
    // Adjust view layouts and animate using information in notification
    ...
}
```

## New text entry APIs

### Inline predictions (new in iOS 17)

- English langugage keyboard will suggest text inline
- Generated using contextual information in the text field

*See more in the session "What's new in text and text interactions".*

```swift
@MainActor public protocol UITextInputTraits : NSObjectProtocol {
    // Controls whether inline text prediction is enabled or disabled during typing
    @available(iOS, introduced: 17.0)
    optional var inlinePredictionType: UITextInlinePredictionType { get set }
}

public enum UITextInlinePredictionType : Int, @unchecked Sendable {
    case `default` = 0
    case no = 1
    case yes = 2
}

let textView = UITextView(frame: frame)
textView.inlinePredictionType = .yes
```
