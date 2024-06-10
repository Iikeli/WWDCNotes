# Add rich graphics to your SwiftUI app

Learn how you can bring your graphics to life with SwiftUI. We’ll begin by working with safe areas, including the keyboard safe area, and learn how to design beautiful, edge-to-edge graphics that won’t underlap the on-screen keyboard. We’ll also explore the materials and vibrancy you can use in SwiftUI to create easily customizable backgrounds and controls, and go over graphics APIs like drawingGroup and the all new canvas. With these tools, it’s simpler than ever to design fully interactive and interruptible animations and graphics in SwiftUI.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10021", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Safe area

- By default, SwiftUI positions your content within the safe area, avoiding anything that would obscure or clip your view, like the Home indicator or any bars that are being shown. 
- The safe area is represented as a region that is inset from the outermost full area where a view is shown. 

## Background/Foreground view modifiers

`.background()` view modifier has new overloads with new behaviors:

- previous `.background()` view modifiers were applied with the same bounds to the view they were called on. 
- to make previous `.background()` effects go beyond the safe area, we had to call `.ignoreSafeArea()` on the background content, e.g. `.background { Color.red.ignoresSafeArea() }`
- the new background modifier gives you this behavior automatically by default

- the new overloads can accept:
  - a specific style, like `Color`, `Gradient`, or the new materials (think UIKit's `UIVibrancyEffect`)
  - a shape, to clip the background to a specific shape (when we use this, the background no longer goes beyond the safe area)

[`.foregroundStyle(_:)`][foregroundstyle(_:)] is a new view modifier that applies styles on top of another view. 

- similar to the new background overloads, it accepts styles
- when passing a [`ForegroundStyle`][ForegroundStyle] between `.secondary` and `.quaternary`, the content might be shown with an effect called Vibrancy, which blends the colors behind it. This happens when you explicitly add a background with a material, or when your content is in a system component, like a sidebar, that adds the material for you.
- Any given text can have a single foreground style applied to it, but multiple colors within its ranges (thanks to the new [AttributedString][AttributedString] API).

`.safeAreaInset(edge:content:)` is a new modifier that lets us reduce the content safe area of the view is applied to.

- it kind of work like an overlay, where the overlay placement is now considered part of the unsafe area
- for example, this allows to place views on top of `ScrollView`, and the edges of the scroll view content (not frame) will not be obscured by those views defined within `.safeAreaInset(edge:content:)`

## Canvas

Previously we had the [`drawingGroup(opaque:colorMode:)`][drawingGroup(opaque:colorMode:)] view modifier:

- drawingGroup tells SwiftUI to combine all of the views it contains in a single layer to draw
- drawingGroup works well for graphical elements, but shouldn't be used with UI controls, like text fields and lists

New `Canvas` view

- addresses some `drawingGroup(opaque:colorMode:)` shortcoming such as bookkeeping and storage required for each view
- similar to UIKit/AppKit `drawRect`

Example:

```swift
Canvas { context, size in 
  var image = context.resolve(Image(systemName: "sparkle"))
  image.shading = .color(.blue)
  let imageSize = image.size 
  context.blendMode = .screen

  for i in 0..<10 {
    let frame = CGRect(
      x: 0.5 * size.width + Double(i) * imageSize.width, y: 0.5 * size. height, 
      width: imageSize.width, height: imageSize.height
    ) 
    var innerContext = context
    innerContext.opacity = 0.5 
    innerContext.fill(Ellipse().path (in: frame), with: .color (.cyan))
    context.draw(image, in: frame) 
  }
}
```

## TimelineView

- new view letting you control exactly how something changes over time
- configurable via schedules like timers and animations

[drawingGroup(opaque:colorMode:)]: https://developer.apple.com/documentation/swiftui/view/drawinggroup(opaque:colormode:)
[foregroundstyle(_:)]: https://developer.apple.com/documentation/swiftui/disclosuregroup/foregroundstyle(_:)
[ForegroundStyle]: https://developer.apple.com/documentation/swiftui/foregroundstyle
[AttributedString]: https://developer.apple.com/documentation/foundation/attributedstring
