# Build for the iPadOS pointer

Help people who use iPad with a Magic Keyboard, mouse, trackpad or other input device get the most out of your app. We’ll show you how to add customizations to the pointer on iPad using pointer interaction APIs, create pointer effects for your buttons and custom views, and change the pointer shape in specific areas of your app to highlight them.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10093", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Updating your app for pointer support

### Controls

- many system components have pointer support built-in: 
  - controls like `UIBarButtonItem`, `UISegmentedControl`, `UIMenuController`, and more.
  - Scroll views respond to scrolling with two fingers and mouse wheels
  - Scroll views respond to pinching to zoom on the trackpad. 
  - Collection and TableView support two finger panning for swipe actions. 
  - `UITextView` and other components that use `UITextInteraction` support a suite quick text selection and editing gestures, behaving similarly to the Mac. 
  - `UIDragInteraction` allows you to drag quickly via a click and drag instead of requiring a long press as it does with touch. 
  - `UIContextMenuInteraction` lets you invoke its menu in a new compact appearance via a secondary click.

- `UIBarButtonItem` have pointer support enabled by default
- `UIButton` offer API that allow you to enable and customize their effects

### Interactions

[`UIPointerInteraction`][UIPointerInteraction] lets your custom UI react to and interact with the pointer: you can choose from one of a collection of system-vended effects to apply to your views, or you can change the pointer's shape within an area of your app. 

### Gestures

[`UIHoverGestureRecognizer`][UIHoverGestureRecognizer] lets you customize to the pointer's motion directly. For more information, see session [`Handle trackpad and mouse input`][20-10094]

### UIButton

- set `isPointerInteractionEnabled` property to true to enable pointer interactions
- set `pointerStyleProvider` to customize the effect

```swift
typealias PointerStyleProvider = (UIButton, UIPointerEffect, UIPointerShape) -> UIPointerStyle?
```

In this `PointerStyleProvider` closure, the system offers you a proposed effect and shape that have been determined based on the appearance, size, and contents of the button. Here, you can customize either of these or replace them entirely and construct a new style.

[`UIPointerStyle`][UIPointerStyle] fits in two categories:

- **Content Effect**: causes the pointer to morph into a view in the app and apply some visual treatment to it. This is described via a [`UIPointerEffect`][UIPointerEffect], which describes the visual treatment applied to the view, and [`UIPointerShape`][UIPointerShape] which describes the shape to which the pointer will change
- **Shape customization**: causes the pointer to morph into the provided shape and is constrained along the specified axes within the current region. This is described via a [`UIPointerShape`][UIPointerShape] and a [`UIAxis`][UIAxis] mask

## Demo

Apple has provided a demo with all the pointer effects [here][demo].

[20-10094]: https://developer.apple.com/videos/play/wwdc2020/10094

[UIPointerStyle]: https://developer.apple.com/documentation/uikit/uipointerstyle
[UIAxis]: https://developer.apple.com/documentation/uikit/uiaxis
[UIPointerShape]: https://developer.apple.com/documentation/uikit/uipointershape
[UIPointerEffect]: https://developer.apple.com/documentation/uikit/uipointereffect
[demo]: https://developer.apple.com/documentation/uikit/pointer_interactions/enhancing_your_ipad_app_with_pointer_interactions
[UIPointerInteraction]: https://developer.apple.com/documentation/uikit/uipointerinteraction
[UIHoverGestureRecognizer]: https://developer.apple.com/documentation/uikit/uihovergesturerecognizer