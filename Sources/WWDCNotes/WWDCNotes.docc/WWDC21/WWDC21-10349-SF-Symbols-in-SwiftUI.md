# SF Symbols in SwiftUI

Discover how you can incorporate SF Symbols into your SwiftUI app. We’ll explore basic techniques for presenting symbols, customizing their size, and showing different variants. We’ll also take you through the latest updates to symbol colorization and help you pick the right tool for your app’s needs.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10349", purpose: link, label: "Watch Video (10 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## How to use them

- via `Image`s, e.g. `Image(systemName: "heart")`
- via `Label`s, e.g. `Label("Heart", systemImage: "heart")`
  - a `Label` is a general description of this text + image pairing and will adapt its behavior to the context where it's shown: sometimes the image will be shown first, sometimes second, sometimes the text is hidden
- via `Text` string interpolation:

```swift
Text("""
    Thalia, Paul, and
    3 others \(Image(systemName: "chevron.forward"))
""")
``` 

## Accessibility

- `Label` is great for accessibility, as the `Label`'s `text` will be read
- for `Image`s:
  - sometimes SwiftUI provides a label based on a system symbol's content
  - you can use `accessibilityLabel("description")` to provide that information
  - for custom symbols, you can provide a localization string for the symbol name (or use the `accessibilityLabel` modifier above)

## Style

- By default SwiftUI uses the Monochrome rendering, which default to black or white in light mode or dark mode. 
- You can set the `foregroundStyle(_:)` to a specific color, or to more semantic values, like the current tint or secondary style: 

```swift
Label("Heart", systemImage: "heart")

Label("Heart", systemImage: "heart")
    .foregroundStyle(.red)

Label("Heart", systemImage: "heart")
    .foregroundStyle(.tint)

Label("Heart", systemImage: "heart")
    .foregroundStyle(.secondary)
```

- You can change both text and font size
  - if you use text style, like `.body` or `.caption` then the text and symbol will scale with dynamic type
  - if you choose a fixed size, then they stay constant

```swift
Label("Heart", systemImage: "heart")
    .font(.body)

Label("Heart", systemImage: "heart")
    .font(.caption)

Label("Heart", systemImage: "heart")
    .font(.system(size: 10))
```

- you can change just he `Label` image by using the `imageScale(_:)` view modifier:

```swift
Label("Heart", systemImage: "heart")
    .imageScale(.large)

Label("Heart", systemImage: "heart")
    .imageScale(.medium)

Label("Heart", systemImage: "heart")
    .imageScale(.small)
```

- variants (NEW) - if we use the base symbol, `Label` will automatically pick the right variant, like `outlined` or `fill`, based on the context (e.g. a `TabView`)

- new `symbolVariant(_:)` view modifier to set the variant to use

```swift
List {
    Label("Ace of Hearts", systemImage: "suit.heart")
    Label("Ace of Spades", systemImage: "suit.spade")
    Label("Ace of Diamonds", systemImage: "suit.diamond")
    Label("Ace of Clubs", systemImage: "suit.club")
    Label("Queen of Hearts", image: "queen.heart")
}
.symbolVariant(.fill)
```

- rendering modes
  - monochrome - to have a constant tint on the whole symbol
  - multicolor - to show colors for what each symbol represents
    - If a symbol doesn't have a multicolor representation, it will fall back to the monochrome rendering mode
  - hierarchical - uses the current foreground style to apply a single color to the symbol, but also adds multiple levels of opacity, to emphasize the key elements of the symbol
  - palette - allows maximum control over the coloring of the layers of a symbol

- new `symbolRenderingMode(_:)` to pick which rendering our symbols should use
- use [`foregroundStyle(_:)`][foregroundStyle(_:)] to use the palette rendering
  - you can specify up to three styles to control each level of the symbol (one for each level of the hierarchy)
  - when you declare only two styles, e.g. `.foregroundStyle(.white, .black)`, the second color will be applied to both secondary and tertiary levels
  - you can apply any `ShapeStyle` with `foregroundStyle(_:)`, not just color: e.g. `.regularMaterial` to blur the background behind a symbol, `.secondary` to get a vibrant effect in front of blurs ...)

```swift
Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.white, .yellow, .red)

Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.white, .red)

Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.white, .secondary)

Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.red, .regularMaterial)
```

- Use the SF Symbols app to preview your SF Symbols customization

[foregroundStyle(_:)]: https://developer.apple.com/documentation/swiftui/view/foregroundstyle(_:)
