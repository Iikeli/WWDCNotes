# SwiftUI Essentials

Take your first deep-dive into building an app with SwiftUI. Learn about Views and how they work. From basic controls to sophisticated containers like lists and navigation stacks, SwiftUI enables the creation of great user interfaces, faster and more easily. See how basic controls like Button are both simple yet versatile. Discover how to compose these pieces into larger, full-featured user interfaces that facilitate building great apps with SwiftUI. Build your SwiftUI skills as you learn the essentials of Apple’s new declarative framework.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/216", purpose: link, label: "Watch Video (58 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Views

- Basic building blocks of user interfaces.
- Define a piece of UI
- Prefer smaller, single-purpose view
- The entire SwiftUI framework is oriented around composition of small pieces: organize the code in the same way
- Each view is a `Struct`,therefore not inheriting anything
- Each view defines a small piece of UI, it doesn’t have to take care of everything
- Views are **not** persistent objects that we update over time using imperative event-based code.
- Views are defined declaratively as a function of their inputs.

## Container Views

- Container views are declared as a composition of other views serving as their content.
- Those Content views are declared within a special kind of closure known as a [@ViewBuilder][vbDoc].

For example, use the [`Form`][FormDoc] container  to make things look like a normal settings screen

## $bindings

- We use the leading dollar sign prefix to indicate that we should pass a binding in our state instead of just passing a read-only value. 
- A binding is a kind of managed reference that allows one view to edit the state of another view.

## Primitive Views

- [`Text`][TextDoc]
- [`Image`][ImageDoc]
- [`Color`][ColorDoc]
- [`Shape`][ShapeDoc]
- [`Spacer`][SpacerDoc]
- [`Divider`][DividerDoc]

## Controls

- Describe purpose, not visuals
- Adaptive and reusable 
- Customizable
- Accessible out of the box

## `@environment` property

- Connected to the environment value. 
- We can use its value just like any other property. 
- The environment is a great encapsulation for pushing data down through the view hierarchy.

[vbDoc]: https://developer.apple.com/documentation/swiftui/viewbuilder
[TextDoc]: https://developer.apple.com/documentation/swiftui/text
[ImageDoc]: https://developer.apple.com/documentation/swiftui/image
[ColorDoc]: https://developer.apple.com/documentation/swiftui/color
[ShapeDoc]: https://developer.apple.com/documentation/swiftui/shape
[SpacerDoc]: https://developer.apple.com/documentation/swiftui/spacer
[DividerDoc]: https://developer.apple.com/documentation/swiftui/divider
[FormDoc]: https://developer.apple.com/documentation/swiftui/form
