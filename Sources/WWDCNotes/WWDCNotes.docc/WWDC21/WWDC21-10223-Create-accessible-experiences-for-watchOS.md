# Create accessible experiences for watchOS

Discover how you can build a top-notch accessibility experience for watchOS when you support features like larger text sizes, VoiceOver, and AssistiveTouch. We’ll take you through adding visual and motor accessibility support to a SwiftUI app built for watchOS, including best practices around API integration, experience, and more.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10223", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Accessibility on watchOS

Assistive technologies:

- VoiceOver - allows people with visual impairments full use of their Apple Watch by navigating a screen using a series of gestures and taps while content is read back to them
- AssistiveTouch (new with watchOS 8) - allows those with motor impairments to use their Apple Watch without the need to touch the screen at all

Display accommodations:

- Reduce Motion
- Bold Text
- Accessibility large text (new with watchOS 8)

## Visual accessibility

- Use [`Font.TextStyle`][textstyle]s to size your fonts - by using a text style, the system will automatically adjust the font size with the system text size settings
- Wrap text, don't truncate - avoid restricting texts with [`lineLimit(_:)`][linelimit(_:)]
- Prefer vertical layouts for larger text styles - use `sizeCategory` environment value to change layout based on the user preferred Dynamic Type:

```swift
struct PlantContainerView: View {
  @Environment(\.sizeCategory) var sizeCategory // 👈🏻
  @Binding var plant: Plant
  
  var body: some View {
    if sizeCategory < .extraExtraLarge {
      PlantViewHorizontal(plant: $plant)
    } else {
      PlantViewVertical(plant: $plant)
    }
  }
}
```

- Use [`accessibilityElement(children:)`][accessibilityElement(children:)] to create a new higher-level accessible element
  - we can tell this modifier to ignore, combine, or contain the accessibility aspects of children of this view
- Use [`accessibilityAdjustableAction(_:)`][accessibilityAdjustableAction] to surface actions the user can take in this element
- Use [`accessibilityLabel(_:)`][accessibilityLabel] - this label is read when navigating to this element
- Use [`accessibilityValue(_:)`][accessibilityValue(_:)] - this value is read out each time it changes

```swift
struct PlantTaskFrequency: View {
  let task: PlantTask
  @Binding var plant: Plant
  let increment: () -> Void
  let decrement: () -> Void
  
  var value: Int {
    switch task {
    case .water:
      return plant.wateringFrequency
    case .fertilize:
      return plant.fertilizingFrequency
    default:
      return 0
    }
  }
  
  var body: some View {
    Section(header: Text("\(task.name) frequency in days"), content: {
      CustomCounter(value: value, increment: increment, decrement: decrement)
        .accessibilityElement()
        .accessibilityAdjustableAction { direction in
          switch direction {
          case .increment:
            increment()
          case .decrement:
            decrement()
          default:
            break
          }
        }
        .accessibilityLabel("\(task.name) frequency")
        .accessibilityValue("\(value) days")
    })
  }
}
```

### Complication tips

- if your complication text contains abbreviations, make sure to add accessibility labels with the non-abbreviated versions
- use [`accessibilityLabel(_:)`][accessibilityLabel] on images and symbols

## AssistiveTouch (Motor accessibility)

- Full use of Apple Watch without touch
- Controlled by hand gestures or hand motions
- Access to functionality on screen content

Two main features:

- the cursor
- the action menu

When you turn on AssistiveTouch, you see a cursor appear on the screen. The cursor will focus on each element on the screen one at a time, in order from top left to bottom right. 

### Gestures

Hand gestures:

- Clench 🤜 ➡️ Tap
- Double-clench 🤜🤜 ➡️ Action Menu
- Pinch 🤏 ➡️ Next element
- Double-pinch 🤏🤏 ➡️ Previous element

Hand motion (alternative to hand gestures, leverages wrist motion):  
By tilting their wrist, people are able to move the onscreen pointer and interact with the UI elements. Similar to AssistiveTouch on iOS, with Dwell Control you can perform an action by resting the pointer over an element for a set amount of time.

<video autoplay muted loop style="max-width: 100%;">
  <source src="WWDC21-10223-motion">
</video>

## Improve AssistiveTouch usage in your app

Focusable elements

- Built-in control elements - only interactive elements that respond to user interaction are focusable (e.g., `Button`, `Toggle` and `NavigationLink` are focusable by default, when not disabled)
- Actionable elements - all elements with some gesture (e.g., `onTapGesture`, [`accessibilityaction(_:_:)`][accessibilityaction(_:_:)])
- Increase the frame size of small interactive elements by using [`contentShape(_:eoFill:)`][contentShape(_:eoFill:)]

```swift
struct DrinkView: View {
  var currentDrink:DrinkInfo
  
  var body: some View {
    HStack(alignment: .firstTextBaseline) {
      DrinkInfoView(drink:currentDrink)
      
      Spacer()
      
      NavigationLink(destination: EditView()) {
        Image(systemName: "ellipsis")
          .symbolVariant(.circle)
      }
      .contentShape(Circle().scale(1.5)) // 👈🏻
    }
  }
}
```

[textstyle]: https://developer.apple.com/documentation/swiftui/font/textstyle
[linelimit(_:)]: https://developer.apple.com/documentation/swiftui/menu/linelimit(_:)
[accessibilityElement(children:)]: https://developer.apple.com/documentation/swiftui/form/accessibilityelement(children:)
[accessibilityAdjustableAction]: https://developer.apple.com/documentation/familycontrols/familyactivitypicker/accessibilityadjustableaction(_:)
[accessibilityLabel]: https://developer.apple.com/documentation/swiftui/view/accessibilitylabel(_:)-1d7jv
[accessibilityValue(_:)]: https://developer.apple.com/documentation/swiftui/view/accessibilityvalue(_:)-z9mo
[accessibilityaction(_:_:)]: https://developer.apple.com/documentation/swiftui/view/accessibilityaction(_:_:)
[contentShape(_:eoFill:)]: https://developer.apple.com/documentation/swiftui/form/contentshape(_:eofill:)