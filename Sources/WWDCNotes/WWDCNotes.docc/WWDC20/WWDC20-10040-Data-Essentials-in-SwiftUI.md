# Data Essentials in SwiftUI

Data is a complex part of any app, but SwiftUI makes it easy to ensure a smooth, data-driven experience from prototyping to production. Discover @State and @Binding, two powerful tools that can preserve and seamlessly update your Source of Truth. We'll also show you how ObservableObject lets you connect your views to your data model. Learn about some tricky challenges and cool new ways to solve them — directly from the experts!

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10040", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Questions to ask when creating a new View

- What data does this view need? 
- How will the view manipulate that data? Is it read only?
- Where will the data come from? (To determine the source of truth)
- Who owns the data? (where the source of truth should come from)

## Why do we need to use property wrappers to store our data?

Our views only exist temporarily: after SwiftUI completes their rendering, the structs instances go away.
When we add a property wrapper such as `@State` to our `View`, SwiftUI takes over its managent, making sure it's there even when our `View`s are re-instantiated over and over.

## When to use what, for view-only sources of truth

### Swift Properties

- Use properties for data that is not mutated by the view (a.k.a. data that the view only reads, but not mutates).

### `@State`

- Use `State` for transient data owned by the view.
- By using `@State`, we're saying that the source of truth belongs to this View (a.k.a. it's local to this view). 

### `@Binding`

- Use `@Binding` for reading and mutating data owned by an ancestor. 
- `@Binding` creates a data dependency between our view and the ancestor owning that source of truth.
- It's ok to build a new binding from an existing binding. 

## `ObservableObject`

- We can think of `ObservableObject` as our interface to the real, UI-independent data/models. It represents SwiftUI's `View`s data depenency.
- `ObservableObject` represents the part of our model/data that exposes data to our view, but it is not necessarily the whole data/model.
- Classes conforming to `ObservableObject` automatically gain a publisher, however we can write our own, for example we can use a publisher for a timer, or use a KVO publisher to observe an existing model.
- When our type conforms to `ObservableObject`, we're creating a new Source of Truth and teaching SwiftUI how to react to its changes.

### `@Published`

- Use `@Published` on the properties of our types conforming to `@ObservableObject` when we want SwiftUI to be notified of their changes.

## When to use what, for `ObservableObject`

### `ObservedObject`

- Use `ObservedObject` when the source of truth is separate from its UI.
- SwiftUI does not get ownership of the `ObservedObject` instance we're providing, it's our responsibility to manage its lifecycle.

### `@StateObject`

- Similar to `ObservedObject`.
- SwiftUI will instantiate the `@StateObject` associated value just before running our view `body` for the first time. 
- SwiftUI will keep the instance alive for the whole lifecycle of the view.

### `@EnvironmentObject`

- Use `@EnvironmentObject` when we need to pass data between views that have other views in-between that don't need that data.

## Behind the scenes of SwiftUI Property Wrappers

- Whenever we use `@ObserveObject` in our views, SwiftUI will subscribe to the `objectWillChange` publisher of that specific `ObservableObject`. 
- Whenever the `ObservableObject` will change, all the views _subscribed_ to it will be updated.

## Performance Tips

- Ensure that our `View`s initialization is cheap.
- Make `body` a pure function, no side effects.
- Avoid making assumptions on when and how often `body` is called.

## Event Sources

- [`onReceive`][onReceive]
- [`onChange`][onChange]
- [`onOpenURL`][onOpenURL]
- [`onContinueUserActivity`][onContinueUserActivity]

The closure associtate with these events is launched in the main thread, dispatch to less priority queues to avoid slow updates 

## Source of Truth Lifetime Extended Lifetime

In iOS 14 SwiftUI can automatically manage restoring the state of our app even after our process has been killed. this it done via two new property wrappers:

- [`@SceneStorage`][ss]
- [`@AppStorage`][as]

Both are lightweight stores that we use in conjunction with our model. 

### [`@SceneStorage`][ss]

- Per scene storage
- Accessible to `View`s
- Used to restore the scene
- It's basically a `Scene`'s version of `@State`

### [`@AppStorage`][as]

- Uses UserDefauts behind the scenes
- Per app storage
- Accessible to `View`s and `App`
- Use it to store small sets of data, such as the app settings

[onChange]: https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/onchange(of:perform:)
[onContinueUserActivity]: https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/oncontinueuseractivity(_:perform:)
[onOpenURL]: https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/onopenurl(perform:)
[onReceive]: https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/onreceive(_:perform:)
[ss]: https://developer.apple.com/documentation/swiftui/scenestorage
[as]: https://developer.apple.com/documentation/swiftui/appstorage