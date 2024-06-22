# Building Custom Views with SwiftUI

Learn how to build custom views and controls in SwiftUI with advanced composition, layout, graphics, and animation. See a demo of a high performance, animatable control and watch it made step by step in code. Gain a deeper understanding of the layout system of SwiftUI.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/237", purpose: link, label: "Watch Video (40 min)")

   @Contributors {
      @GitHubUser(zntfdr)
      @GitHubUser(ATahhan)
   }
}



- `View` always has the size of its body
- The `rootView` has the dimensions of the device minus the safe area insets.
- The top layer of any view with a body is always what we call **layout neutral**. Because its view bounds are defined by the bounds of its body, regardless of what’s _above_.

## Layout Process

Three steps:

1. The root view offers our view a proposed size (the safe layout area)
2. The view answers with the size it would like
3. The root view, know with its child view size knowledge, put the child somewhere (by default at the center of its size, both vertically and horizontally)

In SwiftUI there's no way to force a size on the view's child: the parent has to respect that choice. 

In short:

1. Parent proposes a size for child
2. Child chooses its own size
3. Parent places child in parent’s coordinate space
4. (Bonus) SwiftUI rounds coordinates to the nearest pixel

## Views sizes

Since every view controls its own size, when we build a view, we get to decide how and when it resizes. 

The bounds of a `Text` view never stretch beyond the height and width of its displayed lines.

When we need to fight for space, especially in stack views, we can raise the layout priority by setting the view [`layoutPriority`][layoutPriorityDoc] (`0` by default).

## Custom Alignment

Need of an alignment between views of different stacks?

We can achieve so by creating an extension of [`VerticalAlignment`][verticalAlignemntDoc] (or any other alignment), and then use that as our [`.alignmentGuide`][guideDoc] in the interested views.

[layoutPriorityDoc]: https://developer.apple.com/documentation/swiftui/view/3278584-layoutpriority
[verticalAlignemntDoc]: https://developer.apple.com/documentation/swiftui/verticalalignment
[guideDoc]: https://developer.apple.com/documentation/swiftui/view/3278504-alignmentguide

## Graphics

- SwiftUI has some built in shapes ([`Circle`][circleDoc] for example) that conform to the [`Shape`][shapeDoc] protocol and have their own modifiers, since all shapes are effectively rendered as `View`s, modifiers of shapes can be applied to views and vice versa.

- [`Styling`][stylingDoc] is used to convert shapes into views, available predefined shapes are:
  - [`Rectangle`][rectangleDoc]
  - [`RoundedRectangle`][roundedRectangleDoc]
  - [`Circle`][circleDoc]
  - [`Capsule`][capsuleDoc]
  - [`Ellipse`][ellipseDoc]
  - [`Gradient`s][gradientDoc]

- We can create custom shapes by conforming to the [`Shape`][shapeDoc] protocol and implementing the function [`path(in:)`][pathFunctionDoc] that draws a path representing the desired shape.

- By implementing the [`animatableData`][animatableDataDoc] property in our custom shapes, we can tell SwiftUI how this shape can be interpolated for animations.

- When a view appears/disappears, the default transition animation is to fade-in/fade-out. We can customize these transitions by creating a custom [`ViewModifier`][viewModifierDoc] and specifying how the view should change between the *end states* of the animation, then including that [`ViewModifier`][viewModifierDoc] in an [`asymmetric`][asymmetricDoc] transition.

- SwiftUI [`ViewModifier`][viewModifierDoc]s are `View`s defined as a function of some other view. This can be clearly seen when creating a custom `ViewModifier` where we will have to implement a `body` *function*, which takes a generic `Content` as a parameter (which is constrained to conform to `View`) and returns `some View`.

- By default, SwiftUI renders all views natively as either a `UIView` or an `NSView` (based on the platform it's rendering for), and that's typically what we want for normal controls like `Button`s and `TextField`s. However, if we need better rendering performance for many views, then we can flatten our views inside a [`drawingGroup`][drawingGroupDoc], which is a special way of rendering graphics, where our view hierarchy is flatten into a single `UIView` or `NSView`, which renders all our views using Metal.

[circleDoc]: https://developer.apple.com/documentation/swiftui/circle
[shapeDoc]: https://developer.apple.com/documentation/swiftui/shape
[stylingDoc]: https://developer.apple.com/documentation/swiftui/view/styling
[gradientDoc]: https://developer.apple.com/documentation/swiftui/gradient
[capsuleDoc]: https://developer.apple.com/documentation/swiftui/capsule
[ellipseDoc]: https://developer.apple.com/documentation/swiftui/ellipse
[rectangleDoc]: https://developer.apple.com/documentation/swiftui/rectangle
[roundedRectangleDoc]: https://developer.apple.com/documentation/swiftui/roundedrectangle
[pathFunctionDoc]: https://developer.apple.com/documentation/swiftui/shape/3274626-path
[animatableDataDoc]: https://developer.apple.com/documentation/swiftui/animatable/3046497-animatabledata
[viewModifierDoc]: https://developer.apple.com/documentation/swiftui/viewmodifier
[asymmetricDoc]: https://developer.apple.com/documentation/swiftui/anytransition/3076193-asymmetric
[drawingGroupDoc]: https://developer.apple.com/documentation/swiftui/view/3278548-drawinggroup
