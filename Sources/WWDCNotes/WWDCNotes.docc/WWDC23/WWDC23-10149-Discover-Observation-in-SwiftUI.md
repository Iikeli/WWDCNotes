# Discover Observation in SwiftUI

Simplify your SwiftUI data models with Observation. We’ll share how the Observable macro can help you simplify models and improve your app’s performance. Get to know Observation, learn the fundamentals of the macro, and find out how to migrate from ObservableObject to Observable.


@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10149", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(lordmooch)
   }
}



- **Presenter:** Philippe Hauser, Software Engineer
- **Super power:** Observant. Sees everything.
- **Length:** 12m, Supports Copy Code

# Observation

- Magical new feature in Swift.
- Define models with standard Swift syntax - UI will respond to changes to the model.
- Geared towards SwiftUI - “seamless and intuitive”.
- Easy to update code from `@ObservableObject` **property wrapper** to the `@Observable` **macro**.

## What is observation?

- Tracks changes to properties.
- Normal Swift types transformed with **macro magic**.

Consider your current model…

```Swift
class FoodTruckModel {

    var orders: [Order] = []
    
    var donuts = Donut.all
    
}
```

To make SwiftUI respond to changes in that model…

```Swift
@Observable class FoodTruckModel {

    var orders: [Order] = []
    
    var donuts = Donut.all
    
}
```

`@Observable` is a **macro**.

- It transforms your type into something that is able to be observed.
- You don’t need any Property Wrappers* to make this just work - no more `@Published` annotations required.
- SwiftUI knows that your code accesses specific properties in the body call.
- When body is executed, SwiftUI tracks access to properties of `@Observable` types.
- Now SwiftUI can detect when properties on those specific instances will change.
- Important: if your property isn’t accessed, SwiftUI won’t track it and won’t invalidate the view if the property changes.
- Computed properties will be tracked if they refer to a stored property.

In the model above, if your view uses `donuts` then SwiftUI will track changes to it. If your view does not use `orders`, SwiftUI won’t track it - so you can make changes to `orders` without causing a view redraw.

Expect performance improvements as SwiftUI won’t redraw if a non-tracked property changes - only changes to tracked properties will cause a UI redraw.

## Property Wrappers

SwiftUI property wrappers are simplified to an “all the best things comes in threes” approach: `@State`, `@Environment`, `@Bindable`

**`@State`**

- If a View needs its own state use `@State`.
- Managed by the lifetime of the View it’s contained in.

**`@Environment`**

- Propagate values as globally accessible throughout an app.

- `@Observable` types work great with this.

**`@Bindable`**

- Newest member of the Property Wrappers.
- Lightweight - allows bindings to be created for the type.
- Uses $ syntax as before - `$myType.myProperty`.
- Bindings are read/write - we read from it, but we can write back to the Property too.

### How do I decide which to use?

- If the state is part of the View itself - use `@State`
- If the model is required globally - use `@Environment`
- If we only need bindings - use `@Bindable`
- If No is the answer to all three questions - use the model as a property of your view: `var model: MyModel`
- SwiftUI can observe arrays of your model types. Your model types can also contain other `@Observable` model types.
- General rule - if a used property changes, SwiftUI updates the view.
- We can break this rule with computed properties - if they do not reference any stored property, SwiftUI won’t update the view.
- This can be un-broken though - 2 extra steps required: tell Observation when the property is **accessed** and when the property **changes**.
- Usually not required - most computed properties use some stored property anyway.
- Observation just works for stored properties and computed properties that reference stored properties.
- For computed properties that don’t reference stored properties, we have manual control to make Observation work.
- We can easily replace `@ObservableObject` with `@Observable`.
- Good chance of seeing a performance boost too!

**Before…**

```Swift
public class FoodTruckModel: ObservableObject {
	@Published public var truck = Truck()
	@Published public var orders: [Order] = []
	@Published public var donuts: Donut.all
…
}
```

**After…**

```Swift
@Observable public class FoodTruckModel {
	public var truck = Truck()
	public var orders: [Order] = []
	public var donuts: Donut.all
…
}
```

And in SwiftUI

**Before…**

```Swift
struct AccountView: View {
	
   @ObservedObject var model: FoodTruckModel

   @EnvironmentObject private var accountStore: AccountStore
   @Environment(\.authorizationController) private var authorizationController
…
}
```

**After…**

```Swift
struct AccountView: View {

   var model: FoodTruckModel

   @Environment(AccountStore.self) private var accountStore
   @Environment(AuthorizationController.self) private var authorizationController
…
}
```

- Mostly you will just change annotations.
- Or simplify your view down to the 3 primary Property Wrappers: `@State`, `@Environment`, `@Bindable`.
- Fewer options to consider - Views should be easier to reason about.

## Takeaways

- `@Observable` is a **macro** for automating property observation.
- Can be overridden manually in specific cases if required.
- New projects should use `@Observable`.
- Simplifies models and views.
- Performance boost.


* No Property Wrappers were hurt in the making of `@Observable`
