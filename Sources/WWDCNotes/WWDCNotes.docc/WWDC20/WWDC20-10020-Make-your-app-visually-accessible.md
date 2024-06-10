# Make your app visually accessible

When you design with accessibility in mind, you empower everyone to use your app. Discover how to create an adaptive interface for your app that takes a thoughtful approach to color, provides readable text, and accommodates other visual settings to maintain a great experience throughout.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10020", purpose: link, label: "Watch Video (16 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Color and shapes

### Button Shapes

In iOS 14 there's a new accessibility property for users that prefer an additional shape instead of relying just on the control color: [`buttonShapesEnabled`][buttonShapesEnabled] which we can listen to changes via [`buttonShapesEnabledStatusDidChangeNotification`][buttonShapesEnabledStatusDidChangeNotification].

```swift
func observeButtonShapesNotification() {
    // Make buttons more visible by using shapes.
    // If your default design does not include button shapes, 
    // observe this notification to make visual changes.
    NotificationCenter.default.addObserver(
    	self, 
    	selector: #selector(updateButtonShapes), 
    	name: UIAccessibility.buttonShapesEnabledStatusDidChangeNotification, 
    	object: nil
    )
}

@objc func updateButtonShapes() {
    if UIAccessibility.buttonShapesEnabled {
        // Use extra visualizations for buttons.
    } else {
        // Use default design for buttons.
    }
}
```

### Differentiate without color

From iOS 13 there's a new accessibility property for users that ask you to differentiate app components without using colors (status icons, text, etc): [`shouldDifferentiateWithoutColor`][shouldDifferentiateWithoutColor] which we can listen to changes via [`differentiateWithoutColorDidChangeNotification`][differentiateWithoutColorDidChangeNotification].

```swift
func observeDifferentiateWithoutColorNotification() {
    // Use symbols or shapes to convey meaning instead of relying on color alone.
    // If your default design does not differentiate without color, 
    // observe this notification to make visual changes.
    NotificationCenter.default.addObserver(
    	self, 
    	selector: #selector(updateColorAndSymbols), 
    	name: NSNotification.Name(UIAccessibility.differentiateWithoutColorDidChangeNotification), 
    	object: nil
    )
}

@objc func updateColorAndSymbols() {
    if UIAccessibility.shouldDifferentiateWithoutColor {
        // Use symbols or shapes to convey meaning.
    } else {
        // Use default design.
    }
}
```

### Contrast

To check whether the users has enabled the `Increase Contrast` accessibility feature (iOS 8+), use [`isDarkerSystemColorsEnabled`][isDarkerSystemColorsEnabled] and the [`darkerSystemColorsStatusDidChangeNotification`][darkerSystemColorsStatusDidChangeNotification] notification.

In assets catalog you can also provide a variant (for colors and images) for this accessibility, which will be used automatically when the accessibility feature is enabled.

### Smart Invert Color

Smart Invert Colors is a system setting that asserts an inverted UI over any app. You should flag certain views in your app so they don't get inverted, like photos, videos and app icons.

You can so by setting [`accessibilityIgnoresInvertColors`][accessibilityIgnoresInvertColors] on any `UIView` subclass.

## Text readability

When designing your app, keep text size, weight, and layout in mind for clarity and readability:

- design with large text in mind
- avoid truncating text as the content size increases, wrap labels and use all of the available display width
- symbols and glyphs should scale with text

### Bold Text

- Use text weight to create emphasis and improve readability
- Whenever possible, use system font styles

Bold Text setting allows user to focus on the text: monitor this accessibility feature via the [`isBoldTextEnabled`][isBoldTextEnabled] property and observe the [`boldTextStatusDidChangeNotification`][boldTextStatusDidChangeNotification] notification.

```swift
func observeBoldTextNotification() {
    // Update labels to use bold or heavy font styles.
    // If you aren't using system font styles, observe 
    // this notification to make visual changes.
    NotificationCenter.default.addObserver(
    	self, 
    	selector: #selector(updateLabelWeight), 
    	name: UIAccessibility.boldTextStatusDidChangeNotification, 
    	object: nil
    )
}

@objc func updateLabelWeight() {
    if UIAccessibility.isBoldTextEnabled {
        // Use bold or heavy font weight
    } else {
        // Use font weight that is default to your design.
    }
}
```

## Display preferences

There is a collection of display accommodation settings that people can enable if they're sensitive to motion.

### Reduce motion

When this the user requests reduce motion, suppress idle animations, parallax or other motion effects.

Use [`isReduceMotionEnabled`][isReduceMotionEnabled] and [`reduceMotionStatusDidChangeNotification`][reduceMotionStatusDidChangeNotification] for monitoring these preferences:

```swift
func observeReduceMotionNotification() {
    // Observe this notification to reduce or remove the frequency and intensity of motion effects.
    NotificationCenter.default.addObserver(
    	self, 
    	selector: #selector(updateMotionEffects), 
    	name: UIAccessibility.reduceMotionStatusDidChangeNotification, 
    	object: nil
    )
}

@objc func updateMotionEffects() {
    if UIAccessibility.isReduceMotionEnabled {
        // Reduce or remove extraneous motion effects.
    } else {
        // Use default motion effects.
    }
}
```

## Transitions

New in iOS 14 we have a new [`prefersCrossFadeTransitions`][prefersCrossFadeTransitions] (along with its [`prefersCrossFadeTransitionsStatusDidChange`][prefersCrossFadeTransitionsStatusDidChange] notification), this setting is used to suppress navigation view controllers slide transitions and replace them with a cross fade animation.

```swift
func observeCrossFadeTransitionsNotification() {
    // Reduce or remove sliding animations for transitioning views.
    // If you aren't using system-provided navigation, observe this notification to make visual changes.
    NotificationCenter.default.addObserver(
    	self, 
    	selector: #selector(updateTransitionEffects), 
    	name: UIAccessibility.prefersCrossFadeTransitionsStatusDidChange, 
    	object: nil
    )
}

@objc func updateTransitionEffects() {
    if UIAccessibility.prefersCrossFadeTransitions {
        // Replace sliding transitions with cross-fade animations.
    } else {
        // Use default sliding transitions.
    }
}
```

### Reduce Transparency

When this setting is enabled, blur effects should become completely opaque (use system blur effects to get this behavior automatically).

Use [`isReduceTransparencyEnabled`][isReduceTransparencyEnabled] and [`reduceTransparencyStatusDidChangeNotification`][reduceTransparencyStatusDidChangeNotification] for tracking this.

```swift
func observeReduceTransparencyNotification() {
    // Reduce or remove transparency by adjusting these effects to be completely opaque.
    // If you aren't using system-provided visual effects for blurs or vibrancy, 
    // observe this notification to make visual changes.
    NotificationCenter.default.addObserver(
    	self, 
    	selector: #selector(updateTransparencyEffects), 
    	name: UIAccessibility.reduceTransparencyStatusDidChangeNotification, 
    	object: nil
    )
}

@objc func updateTransparencyEffects() {
    if UIAccessibility.isReduceTransparencyEnabled {
        // Make transparency effects opaque.
    } else {
        // Use default transparency.
    }
}
```

[isReduceTransparencyEnabled]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615074-isreducetransparencyenabled
[reduceTransparencyStatusDidChangeNotification]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615125-reducetransparencystatusdidchang
[prefersCrossFadeTransitionsStatusDidChange]: https://developer.apple.com/documentation/uikit/uiaccessibility/3584819-preferscrossfadetransitionsstatu
[prefersCrossFadeTransitions]: https://developer.apple.com/documentation/uikit/uiaccessibility/3584818-preferscrossfadetransitions
[reduceMotionStatusDidChangeNotification]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615204-reducemotionstatusdidchangenotif
[isReduceMotionEnabled]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615133-isreducemotionenabled
[isBoldTextEnabled]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615156-isboldtextenabled
[boldTextStatusDidChangeNotification]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615152-boldtextstatusdidchangenotificat
[accessibilityIgnoresInvertColors]: https://developer.apple.com/documentation/uikit/uiview/2865843-accessibilityignoresinvertcolors
[isDarkerSystemColorsEnabled]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615087-isdarkersystemcolorsenabled
[darkerSystemColorsStatusDidChangeNotification]: https://developer.apple.com/documentation/uikit/uiaccessibility/1615177-darkersystemcolorsstatusdidchang
[shouldDifferentiateWithoutColor]: https://developer.apple.com/documentation/uikit/uiaccessibility/3043553-shoulddifferentiatewithoutcolor
[differentiateWithoutColorDidChangeNotification]: https://developer.apple.com/documentation/uikit/uiaccessibility/3043554-differentiatewithoutcolordidchan
[buttonShapesEnabled]: https://developer.apple.com/documentation/uikit/uiaccessibility/3618943-buttonshapesenabled
[buttonShapesEnabledStatusDidChangeNotification]: https://developer.apple.com/documentation/uikit/uiaccessibility/3618944-buttonshapesenabledstatusdidchan