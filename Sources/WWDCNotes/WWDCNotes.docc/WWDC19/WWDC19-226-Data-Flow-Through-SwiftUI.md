# Data Flow Through SwiftUI

SwiftUI was built from the ground up to let you write beautiful and correct user interfaces free of inconsistencies. Learn how to connect your data as dependencies while keeping the UI fully predictable and error free. Familiarize yourself with SwiftUI’s powerful data flow tools and understand what the best tool is for each situation.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/226", purpose: link, label: "Watch Video (37 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Principles of Data Flow

- Every time we read a piece of data in our view, we're creating a dependency for that view.
- Every time the data changes, the view has to change (read: re-render) to reflect the new value.
- Every piece of data that we're reading in the view hierarchy has a source of truth.
- The source of truth can live in the view hierarchy (if view-related, like a `collapsed` status), or externally (our models).
- We should always have a single source of truth (no duplicate values).

## [`@State`][StateDoc]

- When a property has an associated property wrapper `@State`, SwiftUI knows that it needs to persist the storage across multiple updates of the same view.
- Mark `@State` property as private, to really enforce the idea that state is **owned and managed** by that view specifically.
- `@State` properties are a special case of state variables, as SwiftUI knows when they change. 
- And because SwiftUI knows that the `@State` variables change the view `body` look, SwiftUI knows that the view rendering depend on their state.
- When a `@State` property changes, in the runtime SwiftUI will recompute the body of that view and all of its children. 
- All the changes always flow down through your view hierarchy. 
- We're able to do these re-rendering very efficiently because SwiftUI compares the views and renders again only what is changed.
- **Views are a function of state, not of a sequence of events**.

## [`@Binding`][BindingDoc]

- By using the `Binding` property wrapper, we define an explicit dependency to a source of truth without owning it.

- We don't need to provide an initial value because the binding can be derived from a state.
- Primitives views such as `Toggle`, `TextField`, and `Slider` all expect a binding.
We can read and write a `@binding` property (without owning it!)

## Publishers (External changes)

- In case the event comes from an external source (a.k.a. not within the views), we use `Combine`’s [`Publisher`s][pubDoc]
- It's very important for `View`s to receive events on the main thread.

## `ObservableObject` (External Data)

- When the data is outside our View, swiftUI needs to know how to react on changes of that data
- By making our external data conform to [`ObservableObject`][observableDoc] protocol, we are required to have a publisher, accessible via the synthesized [`objectWillChange`][willChangeDoc] property, that fires every time the data is about to change.
- All we need to do later is to associate an `@ObservedObject` property wrapper in the model and pass the `ObservedObject` to the view in its initialization.
- While SwiftUI `View`s are value types (`Struct`s), any time we're using a reference type, we should be using the `@ObservedObject` property wrapper: this way SwiftUI will know when that data changes and can keep our view hierarchy up to date.

## Environment

- Thanks to the `.environment` modifier, we can add our `ObservableObject`s into the environment.
- By using this [`@EnvironemntObject`][envObj] property wrapper, we can create a dependency to these environment objects. 

## Object Binding vs Environment

- We can build your whole app with `@Binding`, but it can get kind of cumbersome to pass around the model from view to view.
- `EnvironmentObject` is a convenience for passing data around your hierarchy indirectly (without the need to pass the object to each view first).

## `@State` vs `@ObservedObject`

- `@State` is great for data that's view **local**, a value type, managed.
- `@State` is allocated, created and managed by SwiftUI
- `@State` is value type

- `@ObservedObject` is for representing **external data** to SwiftUI, such as a database.
- `@ObservedObject` is storage that **we** manage, which makes it great for the model that we already have.
- `@ObservedObject` is managed by us
- `@ObservedObject` is reference type

[StateDoc]: https://developer.apple.com/documentation/swiftui/state
[BindingDoc]: https://developer.apple.com/documentation/swiftui/binding
[ToggleDoc]: https://developer.apple.com/documentation/swiftui/toggle
[TextFieldDoc]: https://developer.apple.com/documentation/swiftui/textfield
[SliderDoc]: https://developer.apple.com/documentation/swiftui/slider
[PubDoc]: https://developer.apple.com/documentation/combine/publisher
[observableDoc]: https://developer.apple.com/documentation/combine/observableobject
[willChangeDoc]: https://developer.apple.com/documentation/combine/observableobject/3362556-objectwillchange
[envObj]: https://developer.apple.com/documentation/swiftui/environmentobject