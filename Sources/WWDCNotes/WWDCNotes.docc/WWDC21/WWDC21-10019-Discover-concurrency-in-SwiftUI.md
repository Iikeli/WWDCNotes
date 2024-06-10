# Discover concurrency in SwiftUI

Discover how you can use Swift’s concurrency features to build even better SwiftUI apps. We’ll show you how concurrent workflows interact with your ObservableObjects, and explore how you can use them directly in your SwiftUI views and models. Find out how to use await to make your app run smoothly on the SwiftUI runloop, and learn how to fetch remote images quickly with the AsyncImage API. And we'll take you through the process of enabling additional asynchronous flows in your custom views.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10019", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## ObservableObject + @MainActor

Imagine a view with the following class as a dependency:

```swift
class Photos: ObservableObject { 
  @Published var items: [SpacePhoto] = [] 

  func updateItems() { 
    let fetched = /* fetch new items */ 
    items = fetched 
  }
}
```

When we assign to `items` (second row in `updateItems()`):

1. an `objectWillChange` event will trigger
2. then the new data will be stored in `items`'s storage

When SwiftUI receives an `objectWillChange` event:

1. it takes a snapshot of the object (**before** its storage is updated). 
2. once the storage is updated, SwiftUI compares the previous snapshot with the current value
3. if the values are different, SwiftUI will update the view (and all other views depending on this `Photos` object)

This flow works great as long as we are on the main thread/loop. If we assign to `items` in another thread, both SwiftUI snapshots might be taken before the `items` storage is actually updated, meaning that the snapshot comparison will find unchanged values, thus not updating our views.

New in Swift 5.5, we can declare our `ObservableObject` class with swift's `@MainActor` attribute and, instead of asynchronous callbacks, use the new `await` syntax to makes sure all operations are executed in the main thread. 

## New SwiftUI features

- [`task(_:)`][task(_:)] is a new view modifier that lets you run a task for the view lifetime
  - it starts when the view appears (similar to `onAppear`)
  - it gets cancelled when the view disappears (similar to `onDisappear`)

- use [`AsyncImage`][AsyncImage] to asynchronously load and display an image

```swift
AsyncImage(url: photo.url) { image in
  image
    .resizable()
    .aspectRatio(contentMode: .fill)
} placeholder: {
  ProgressView()
}
.frame(minWidth: 0, minHeight: 400)
```

- use [`refreshable(action:)`][refreshable(action:)] to add pull to refresh capabilities in your views
  - as the `action` parameter expects an `async` closure, the refresh indicator stays visible for the duration of the awaited operation

[task(_:)]: https://developer.apple.com/documentation/swiftui/emptyview/task(_:)
[AsyncImage]: https://developer.apple.com/documentation/swiftui/asyncimage
[refreshable(action:)]: https://developer.apple.com/documentation/swiftui/view/refreshable(action:)
