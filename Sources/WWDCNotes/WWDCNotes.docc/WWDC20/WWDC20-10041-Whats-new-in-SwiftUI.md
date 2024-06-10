# What's new in SwiftUI

SwiftUI can help you build better and more powerful apps for iPhone, iPad, Mac, Apple Watch, and Apple TV.  Learn more about the latest refinements to SwiftUI, including interface improvements like outlines, grids, and toolbars. Take advantage of SwiftUI’s enhanced support across Apple frameworks to enable features like Sign In with Apple. Discover new visual effects, as well as new controls and styles. And find out how the new app and scene APIs enable you to create apps entirely in SwiftUI, as well as custom complications and all new widgets.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10041", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## 100% SwiftUI Apps
We can now build an entire app completely in SwiftUI.

This is a fully functional app:

```swift
@main
struct HelloWorld: App {
    var body: some Scene {
        WindowGroup {
            Text("Hello, world!").padding()
        }
    }
}
```

- We can do so thanks to the new [`App`][appDoc] protocol, which requires a `body` of type [`some Scene`][sceneDoc].
- [`WindowGroup`][winDoc] lets us create scenes easily based on the platform, and more.
- [`DocumentGroup`][docDoc] lets us open, edit, and save document-based scenes.

## LaunchScreen
- New `Launch Screen` `Info.plist` key
- Lets us declare standard UI components such as images, navigation bar, tab bar, background colors as the app launch screen. 
- Replaces the `LaunchScreen.storyboard`.

## Widgets
- Built exclusively with SwiftUI
- Widgets are just like Views, however instead of conforming to `View`, they conform to the [`Widget`][widgetDoc] protocol, and the `body` returns a [`some WidgetConfiguration`][wiConf] instance. 

## Apple Watch
- We can use SwiftUI to build custom complications

## Lists
- Stacks behave more like `List` by prepending them with with Lazy (e.g. [`LazyVStack`][lazyvstack])
- New grid component via [`LazyHGrid`][lazyHGrid], [`LazyVGrid`][lazyVGrid], and [`GridItem`][GridItem]

## Toolbar and Controls
- New [`.toolbar`][tbDoc] modifier and [`ToolBarItem`][tbitem], those are examples of **semantic placements**, we describe to SwiftUI the role that these toolbar items have, and SwiftUI will automatically place them properly.
- We can also specify the placement in the `ToolBarItem` explicitly in its initializer

- [`Label`][labelDoc]
    - new View that combines a text and an image (in reality `Label` accepts two `AnyView` instances)
    - `Label`s adapt neatly based on the context and dynamic type size, sometimes the text might not be shown (for example on navigation bars)

- [`ProgressView`][proViewDoc]
    - can display determinant and indeterminate progress
    - linear and circular style
    - customizable

- [`Gauge`][gaugeDoc]
    - used to display a value relative to a range (min/max)

- Other important modifiers
    - `.help`
    - `.keyboardShortcut`

## Effects and Styling
- [`matchedGeometryEffect`][mgeDoc]
- [`ContainerRelativeShape`][crsDoc]
- We can add a new `AppAccentColor` color assets in our bundle to have our app automatically adapt that accent color in all our UI.
- We can also customize some parts of our UI with a different color than the app accent color by using the relative tint modifier such as [`listItemTint`][lit]

## System Integration
- The new [`Link`][lv] view lets open a webpage or deep link to another app, this is the equivalent of creating a button with a `UIApplication.shared.open` action
- drag and drop support
- [`SignInWithAppleButton`][signIn]
- More integrations:
    - AVKit
    - MapKit
    - SceneKit
    - SpriteKit
    - QuickLook
    - HomeKit
    - ClockKit
    - StoreKit
    - WatchKit

[appDoc]:  https://developer.apple.com/documentation/swiftui/app
[sceneDoc]: https://developer.apple.com/documentation/swiftui/scene
[winDoc]: https://developer.apple.com/documentation/swiftui/windowgroup
[docDoc]: https://developer.apple.com/documentation/swiftui/documentgroup
[widgetDoc]: https://developer.apple.com/documentation/swiftui/widget
[wiConf]: https://developer.apple.com/documentation/swiftui/widgetconfiguration 
[tbDoc]: https://developer.apple.com/documentation/swiftui/view/toolbar(content:)  
[tbitem]: https://developer.apple.com/documentation/swiftui/toolbaritem 
[labelDoc]: https://developer.apple.com/documentation/swiftui/label
[proViewDoc]: https://developer.apple.com/documentation/swiftui/progressview 
[gaugeDoc]: https://developer.apple.com/documentation/swiftui/gauge 
[mgeDoc]: https://developer.apple.com/documentation/swiftui/view/matchedgeometryeffect(id:in:properties:anchor:issource:) 
[crsDoc]: https://developer.apple.com/documentation/swiftui/containerrelativeshape 
[lit]: https://developer.apple.com/documentation/swiftui/view/listitemtint(_:)-12mbh 
[lv]: https://developer.apple.com/documentation/swiftui/link 
[signIn]: https://developer.apple.com/documentation/swiftui/signinwithapplebutton 
[lazyvstack]: https://developer.apple.com/documentation/swiftui/lazyvstack 
[lazyHGrid]: https://developer.apple.com/documentation/swiftui/lazyhgrid
[lazyVGrid]: https://developer.apple.com/documentation/swiftui/lazyvgrid
[GridItem]: https://developer.apple.com/documentation/swiftui/griditem
