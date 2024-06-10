# Localize your SwiftUI app

Learn how to localize your SwiftUI app and make it available to a global audience. Explore how you can localize strings in SwiftUI, including those with styles and formatting. We'll demonstrate how you can save time by having SwiftUI automatically handle tasks such as layout and keyboard shortcuts, and take you through the localization workflow in Xcode 13.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10220", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## `Text` 

- Format types are automatically inferred in Xcode 13
- `Text` and `LocalizedStringKey` now work together with the compiler to better extract localizable content from code source (e.g. with multiline text)

## Styling

- ability to style localizable strings using markdown
- different localizations can use different formatting (e.g. `"hello _world_" = "Hola **Mundo**";`)

## Formatting

- we can specify the format in a declarative manner directly inline with where the value is being shown
- more performant APIs

Old way:

```swift
let calories = Measurement<UnitEnergy>(value: nutritionFact.kilocalories, unit: .kilocalories)

static let measurementFormatter: MeasurementFormatter = {
  let formatter = MeasurementFormatter()
  formatter.unitStyle = .long
  formatter.unitOptions = .providedUnit
  return formatter
}()

Text(Self.measurementFormatter.string(from: calories))
Text("Energy: \(calories, formatter: Self.measurementFormatter)")
```

New way:

```swift 
let calories = Measurement<UnitEnergy>(value: nutritionFact.kilocalories, unit: .kilocalories)

Text(calories.formatted(.measurement(width: .wide, usage: .food)))
Text("Energy: \(calories, format: .measurement(width: .wide, usage: .food))")
```

## Shortcuts

- New in macOS and iPadOS, any keyboard shortcuts you define in your SwiftUI app will now be automatically adjusted so that they can be typed on the user's currently active keyboard layout
  - this is thanks to a new remapping feature of macOS Monterey and iPadOS 15
  - no work to be done on the developer side

## Exporting for localization

- New Xcode project build settings `Use Compiler to Extract Swift Strings`
  - Once active, when before exporting strings for localization, Xcode will build all project targets and use the compiler type information to extract LocalizedStringKeys from your SwiftUI code
- New in Xcode 13 `.xcloc` a.k.a. Xcode Localization Catalogs can be opened directly in Xcode with the new Localization Catalog Editor

## Summary

- Extract `LocalizeStringKey`s
- Turn on `Use Compiler to Extract Swift Strings` project build setting
- Internationalize your code with formatting
- Style your localized strings with Markdown
- Use `Text()` to add comments for translation context
