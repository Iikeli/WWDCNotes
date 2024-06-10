# Unleash the UIKit trait system

Discover powerful enhancements to the trait system in UIKit. Learn how you can define custom traits to add your own data to UITraitCollection, modify the data propagated to view controllers and views with trait override APIs, and adopt APIs to improve flexibility and performance. We’ll also show you how to bridge UIKit traits with SwiftUI environment keys to seamlessly access data from both UIKit and SwiftUI components in your app.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10057", purpose: link, label: "Watch Video (29 min)")

   @Contributors {
      @GitHubUser(srujanc)
      @GitHubUser(rusik)
   }
}



# UIKit Trait System

UIKit provides many built-in system traits, such as user interface style, horizontal size class, and preferred content size category. In iOS 17, we can define our own custom traits as well. This unlocks a powerful new way to provide data to the app's view controllers and views. The main way to work with traits in UIKit is using [trait collections](https://developer.apple.com/documentation/uikit/uitraitcollection?language=objc).


### UITraitCollection

A trait collection contains traits and their associated values. The `traitCollection` property of the [`UITraitEnvironment`](https://developer.apple.com/documentation/uikit/uitraitenvironment?language=objc) protocol contains traits that describe the state of various elements of the iOS user interface, such as size class, display scale, and layout direction. Together, these traits compose the UIKit trait environment.


There are some new APIs in iOS 17 that make it easier to work with trait collections.

### Trait environments and hierarchy

![Trait environments and hierarchy][trait-hierarchy]  

[trait-hierarchy]: WWDC23-10057-trait-hierarchy

Example of how trait hierarchy worked prior to iOS 17. Flow of traits in the view hierarchy stopped at each view owned by a view controller. This behavior could be surprising.

![Trait hierarchy before iOS 17][traits-ios16]  

[traits-ios16]: WWDC23-10057-traits-ios16

Updated trait hierarchy on iOS 17. View controllers inherit their trait collection from their view’s superview, instead of directly from their parent view controller. This creates a simple linear flow of traits through view controllers and views. 

![Trait hierarchy after iOS 17][traits-ios17]  

[traits-ios17]: WWDC23-10057-traits-ios17

#### View controller trait updates

* Traits not up-to-date in `viewwillAppear(_:)`
* Use `viewIsAppearing(_:)` instead
	* View controller and view traits up-to-date
	* View added to hierarchy, has accurate geometry

Learn more about `viewIsAppearing(_:)` in [What's new in UIKit](https://developer.apple.com/videos/play/wwdc2023/10055/?time=128)

#### View trait updates

* Views only update traits when in the hierarchy
* Views update traits before layout
* `layoutSubviews()` is the best place to use traits

### Working with trait collections

```swift
// Build a new trait collection instance from scratch
let myTraits = UITraitCollection { mutableTraits in
    mutableTraits.userInterfaceIdiom = .phone
    mutableTraits.horizontalSizeClass = .regular
}
```

From the above code, there is a mutableTraints variable in side the closure which conforms to a new protocol called [`UIMutableTraits`](https://developer.apple.com/documentation/uikit/uimutabletraits). When the closure finishes executing, the initializer returns an immutable `UITraitCollection` instance that contains all of the trait values I set inside the closure. 

There’s also a new `modifyingTraits` method that allows you to create a new instance by modifying values from the original trait collection inside the closure.

```swift
// Get a new instance by modifying traits of an existing one
let otherTraits = myTraits.modifyingTraits { mutableTraits in
    mutableTraits.horizontalSizeClass = .compact
    mutableTraits.userInterfaceStyle = .dark
}
```

### Implementing a simple custom trait

Conform to `UITraitDefinition` protocol with one required static property `defaultValue`. This is default value for the trait when no  value has been set. Each trait definition has an associated value type, which is inferred from the `defaultValue`. 

```swift
struct ContainedInSettingsTrait: UITraitDefinition {
    static let defaultValue = false
｝

let traitCollection = UITraitCollection { mutableTraits in
    mutableTraits[ContainedInSettingsTrait.self] = true
}

let value = traitCollection[ContainedInSettingsTrait.self]
// true
```

Adding property syntax with simple extension

```swift
extension UITraitCollection {
    var isContainedInSettings: Bool { self[ContainedInSettingsTrait.self] }
｝

extension UMutableTraits {
    var isContainedInSettings: Bool {
        get { self[ContainedInSettingsTrait.self] }
        set { self[ContainedInSettingsTrait.self] = newValue }
    ｝
｝

let traitCollection = UITraitCollection { mutableTraits in
    mutableTraits.isContainedInSettings = true
}

let value = traitCollection.isContainedInSettings
// true
```


### UIMutableTraits

The `UIMutableTraits` protocol provides read-write access to get and set trait values on an underlying container. UIKit uses this protocol to facilitate working with instances of `UITraitCollection`, which are immutable and read-only. The `UITraitCollection` initializer `init(mutations:)` uses an instance of `UIMutableTraits`, which enables you to set a batch of trait values in one method call. `UITraitOverrides` conforms to `UIMutableTraits`, making it easy to set trait overrides on trait environments such as views and view controllers.


```swift
// ThemeTrait conforms to UITraitDefinition, and has a defaultValue type of Theme
extension UIMutableTraits {
    var theme: Theme {
        get { self[ThemeTrait.self] }
        set { self[ThemeTrait.self] = newValue }
    }
}

// Apply an override for the custom theme trait.
view.traitOverrides.theme = .monochrome
```

Trait overrides are the mechanism you use to modify data within the trait hierarchy. In iOS 17, it’s easier than ever to apply trait overrides. There’s a new `traitOverrides` property on each of the trait environment classes, including window scenes, windows, views, view controllers, and presentation controllers.

> Trait overrides applied to the parent affect the parent’s own trait collection. And then the values from the parent’s trait collection are inherited to the child. Finally, the child's trait overrides are applied to the values it inherited to produce its own trait collection.

```swift
/// Managing trait overrides

func toggleThemeOverride(_ overrideTheme: MyAppTheme) {
    if view.traitOverrides.contains(MyAppThemeTrait.self) {
        // There's an existing theme override; remove it
        view.traitOverrides.remove(MyAppThemeTrait.self)
    } else {
        // There's no existing theme override; apply one
        view.traitOverrides.myAppTheme = overrideTheme
    }
}
```

**Note:** `traitCollectionDidChange` is deprecated in iOS 17. When you implement `traitCollectionDidChange`, the system doesn’t know which traits you actually care about, so it has to call that method every time that any trait changes value. However, most classes only use a handful of traits and don’t care about changes to any others. This is why `traitCollectionDidChange` doesn’t scale as you add more and more custom traits. 


In place of `traitCollectionDidChange` we have new traits registration methods in iOS 17.0 with a closure-method based.

> Call `registerForTraitChanges` and pass an array of traits to register for as well as the target and action method to call on changes. The target parameter is optional. If you omit it, the target will be the same object that `registerForTraitChanges` is called on.


```swift
// Register for horizontal size class changes on self
registerForTraitChanges(
    [UITraitHorizontalSizeClass.self],
    action: #selector(UIView.setNeedsLayout)
)

// Register for changes to multiple traits on another view
let anotherView: MyView
anotherView.registerForTraitChanges(
    [UITraitHorizontalSizeClass.self, ContainedInSettingsTrait.self],
    target: self,
    action: #selector(handleTraitChange(view:previousTraitCollection:))
)

@objc func handleTraitChange(view: MyView, previousTraitCollection: UITraitCollection) {
    // Handle the trait change for this view...
}
```

The first parameter is always the object whose traits are changing. Use this parameter to get the new `traitCollection`. The second parameter will always be the previous trait collection for that object before the change. In addition to registering for individual traits, you can also register using new semantic sets of system traits.

### SwiftUI bridging

* Bridge UIKit traits and SwiftUI environment keys
* Data propagates across UIKit and SwiftUI boundaries
* Works in both directions

Implementing a bridged UIKit trait and SwiftUI environment key

```swift
// Custom UIKit trait
struct MyAppThemeTrait: UITraitDefinition {...}

// Custom SwiftUI environment key
struct MyAppThemeKey: EnvironmentKey {...}

// Bridge SwiftUI environment key with UIKit trait
extension MyAppThemeKey: UITraitBridgedEnvironmentKey {
    static func read(from traitCollection: UITraitCollection) -> MyAppTheme {
       traitCollection.myAppTheme
    }

    static func write(to mutableTraits: inout UIMutableTraits, value: MyAppTheme) {
        mutableTraits.myAppTheme = value
    }
}
```

Using a bridged UIKit trait and SwiftUI environment key

```swift
// UIKit trait override applied to the window scene
windowScene.traitOverrides.myAppTheme = monochrome

// Cell in a UICollectionView configured to display a SwiftUI view
cell.contentConfiguration = UIHostingConfiguration {
    CellView()
}

// SwiftUI view displayed in the cell, which reads the bridged value from the environment
struct CellView: View {
    @Environment (\.myAppTheme) var theme: MyAppTheme
    var body: some View {
        Text("Settings")
            .foregroundStyle(theme == monochrome ? .gray : .blue)
    }
}

// SwiftUI environment value applied to a UIViewControllerRepresentable
struct SettingsView: View {
    var body: some View {
        SettingsControllerRepresentable()
            .environment(\.myAppTheme, .standard)
    }
}

// UIKit view controller contained in the SettingsControllerRepresentable
class SettingsViewController: UIViewController {
    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        title = settingsTitle(for: traitCollection.myAppTheme)
    }
}
```
