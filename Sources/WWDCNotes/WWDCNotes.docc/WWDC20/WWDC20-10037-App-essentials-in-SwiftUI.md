# App essentials in SwiftUI

Thanks to the new App protocol, SwiftUI now supports building entire apps! See how Apps, Scenes, and Views fit together. Learn how easy it is to implement the features people expect from a best-in-class product while saving time and reducing complexity. Easily add expected functionality to your interface using the new commands modifier, and explore the ins and outs of the new WindowGroup API.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10037", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## `View`s, [`Scene`][scene]s, and [`App`][app]s

- Beside `View`s, from this year SwiftUI can declare `Scene`s and `App`s

- `View`s are the basic building blocks, rendering everything on screen. `View`s can be composed with other `View`s.

- `View`s form the content of a `Scene`, making them platform-independent

- `Scene`s can also be composed with other `Scene`s (think of tabs on macOS)

- `Scene`s form the content of an `App`

### Example

```Swift
@main
struct MyApp: App {
    @StateObject private var store = Store()

    var body: some Scene {
        WindowGroup {
            MyView(store: store)
        }
    }
}
```

- In thie example we have an `App`, `MyApp`, which contains a `Scene`, [`WindowGroup`][wg], which contains a view `MyView`.

- Above the struct declaration we have a new Swift 5.3 attribute [`@main`][main], which declares this `struct` as the entry point of our programs execution.

- Declaring `App`s is very similar to declaring `View`s: both comform to a protocol (`App` vs `View`), and both need to define a `body` (of type `some Scene` vs `some View`).

- Like `View`s, also Apps can declare data depenencies, and in this example `MyApp` declares to be owner of the `store` object by using the `StateObject` property wrapper,

## [`WindowGroup`][wg]

- Scene that allows us to define the primary interface of our app.

- `WindowGroup` is a scene that allows our app to automatically create multiple scenes containing `MyView` when needed (this works for having multple windows on both iPad and mac).

- Like in UIKit, an app can provide a shared model for each scene to use, but the state of the views in those scenes will be independent.

- `WindowGroup` also manages tabs for us on macOS.

- [`@SceneStorage`][scenestore] is a new property wrapper that manages scenes restoration.

## More New features

- [`DocumentGroup`][docg]: New scene that help us with document based apps, more on this in `Build Document-Based Apps in SwiftUI` session.

- [`Settings`][sett] is a macOS-only scene used for declaring apps preferences

- The [`Commands`][cmd] API (along with the [`.commands`][cmdm] modifier) let us describe commands to be shown in the main menu on macOS and key commands on iOS.

[app]: https://developer.apple.com/documentation/swiftui/app
[scene]: https://developer.apple.com/documentation/swiftui/scene
[main]: https://docs.swift.org/swift-book/ReferenceManual/Attributes.html#ID626
[wg]: https://developer.apple.com/documentation/swiftui/windowgroup
[scenestore]: https://developer.apple.com/documentation/swiftui/scenestorage
[docg]: https://developer.apple.com/documentation/swiftui/documentgroup
[sett]: https://developer.apple.com/documentation/swiftui/settings
[cmd]: https://developer.apple.com/documentation/swiftui/commands
[cmdm]: https://developer.apple.com/documentation/swiftui/scene/commands(content:)