# The SwiftUI cookbook for navigation

The recipe for a great app begins with a clear and robust navigation structure. Join the SwiftUI team in our proverbial coding kitchen and learn how you can cook up a great experience for your app. We’ll introduce you to SwiftUI’s navigation stack and split view features, show you how you can link to specific areas of your app, and explore how you can quickly and easily restore navigational state.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10054", purpose: link, label: "Watch Video (26 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Key moments
[00:57 - Setup/overview](https://developer.apple.com/videos/play/wwdc2022/10054/?time=57)    
[01:25 - Comparison of pre-iOS16 navigation APIs with new](https://developer.apple.com/videos/play/wwdc2022/10054/?time=)    
[05:24 - Recipes for navigation](https://developer.apple.com/videos/play/wwdc2022/10054/?time=324)    
[06:06 - Pushable Stack recipe, single stacks of view](https://developer.apple.com/videos/play/wwdc2022/10054/?time=366)    
[10:43 - Multiple Columns recipe, columns showing progressively more info](https://developer.apple.com/videos/play/wwdc2022/10054/?time=643)    
[14:12 - Multiple Columns With Stack recipe, Navigating between related information](https://developer.apple.com/videos/play/wwdc2022/10054/?time=852)    
[18:04 - Persistent State](https://developer.apple.com/videos/play/wwdc2022/10054/?time=1084)    
[23:34 - Navigation tips/summary](https://developer.apple.com/videos/play/wwdc2022/10054/?time=1414)    

> [Sample App](https://developer.apple.com/documentation/swiftui/bringing_robust_navigation_structure_to_your_swiftui_app)

## Deprecation notice

All previous navigation APIs are deprecated:

- `NavigationView` is replaced by two new _containers_, [`NavigationStack`][NavigationStack] and [`NavigationSplitView`][NavigationSplitView] (more on this later)
- `NavigationLink` still exists, has a completely different API and behavior, and is no longer mandatory to use

## New navigation APIs

### Containers

- with the new containers, we have a single binding for managing their stack state/path
- this single binding represents all the values pushed onto the stack, think of it as an array of screens
  - The new `NavigationLink` APIs append values to this binding
  - you can directly mutate this binding yourself (just add/remove elements from this state/path array)
  - You can programmatically push/pop multiple screens at once
  - You can programmatically deep link
  - You can pop to the root view by removing all the items

Two new containers:

1. [`NavigationStack`][NavigationStack] - for a single push-pop stack
2. [`NavigationSplitView`][NavigationSplitView] - for multi-column apps
  - automatically adapts into a single-column stack on iPhone, into Slide Over on iPad, Apple Watch and Apple TV
  - provides configuration options that let you:
    - customize column widths (see [`navigationSplitViewColumnWidth(_:)`][navigationSplitViewColumnWidth(_:)] and similar modifiers)
    - control sidebar presentation and show/hide columns (see [`NavigationSplitViewVisibility`][navigationsplitviewvisibility] and associated modifiers)

`NavigationSplitView` has two initializers:

- To create a two-column navigation split view, use [`init(sidebar:detail:)`][init(sidebar:detail:)]
- To create a three-column view, use the [`init(sidebar:content:detail:)`][init(sidebar:content:detail:)]

```swift
NavigationStack(path: $path) {
  NavigationLink("trigger", value: value)
}

NavigationSplitView {
  List(model.employees, selection: $employeeIds) { employee in
    Text(employee.name)	
  }
} detail: {
  EmployeeDetails(for: employeeIds)
}
```

### NavigationLink

- appends elements onto the stack it appears in
- no longer accepts a `destination` parameter (we use the new [`navigationDestination(for:destination:)`][navigationDestination(for:destination:)] modifier instead)
- is no longer needed, programmatically append things to the `$path` state yourself instead

### [`navigationDestination(for:destination:)`][navigationDestination(for:destination:)] modifier

- declares the type of the presented data that it's responsible for
- takes in a view builder that describes what view to push onto the stack when a instance of that data is presented

```swift
NavigationStack {
  List {
    NavigationLink("Mint", value: Color.mint)
    NavigationLink("Pink", value: Color.pink)
    NavigationLink("Teal", value: Color.teal)
  }
  .navigationDestination(for: Color.self) { color in
    ColorDetail(color: color) // The view to be pushed
  }
  .navigationTitle("Colors")
}
```

> Explanation: when a navigation link is tapped, a new `Color` value is appended to the `NavigationStack` internal `$path` state, `NavigationStack` then asks the `navigationDestination(for:destination:)` modifier associated with `Color` for the view to be pushed.

- Every navigation stack keeps track of a path that represents all the data that the stack is showing
- When the stack shows its root view, the path is empty
- the stack also keeps track of all the navigation destinations declared inside it, or inside any view pushed onto the stack
- if you'd like to use this path yourself (e.g., for programmatic navigation), use the new type-erasing [`NavigationPath`][NavigationPath] collection where you can push values of different types

Example with explicit path:

```swift
struct ContentView: View {
  /// The stack state.
  @State private var path: [Int] = []

  var body: some View {
    // This navigation doesn't use NavigationLink! 🎉
    NavigationStack(path: $path) {
      NumberView(0, onGoToNextNumber: { path = [1] })
        // 👇🏻 This modifier returns the view to be pushed for $path values of type Int
        .navigationDestination(for: Int.self) { number in 
          NumberView(number, onGoToNextNumber: { path.append(number + 1) })
        }
    }
  }
}

/// A view that displays a number and has a button triggering an action injected by 
/// its parent view.
struct NumberView: View {
  var number: Int
  var onGoToNextNumber: () -> Void

  init(_ number: Int, onGoToNextNumber: @escaping () -> Void) {
    self.number = number
    self.onGoToNextNumber = onGoToNextNumber
  }

  var body: some View {
    Text("\(number)").font(.headline)
    Button("Go to next number", action: onGoToNextNumber)
  }
}
```

## Persistent state

1. encapsulate your navigation state into a `Codable` model
2. use `SceneStorage` to save and restore that state
3. restore data via `.task`:

```swift
// Use SceneStorage to save and restore
@StateObject private var navModel = NavigationModel()
@SceneStorage("navigation") private var data: Data?

var body: some View {
  NavigationSplitView { ... }
  .task {
  	// restore state if present
  	if let data = data {
  		navModel.jsonData = data
    }

    // start an asynchronous for loop that will iterate 
    // whenever my navigation model changes. The body of 
    // this loop will run on each change, so I can use 
    // that to save my navigation state back to my scene 
    // storage data.
    for await _ in navModel.objectWillChangeSequence {
    	data = navModel.jsonData
    }
  }
}
```

## Tips

- Check Apple's [Migrating to New Navigation Types][migrating-to-new-navigation-types] article
- `List`, `NavigationSplitView`, and `NavigationStack` were made to mix together
- put `navigationDestination(for:destination:)` modifiers within easy reach
- start with `NavigationSplitView` when it makes sense

[NavigationStack]: https://developer.apple.com/documentation/swiftui/navigationstack
[NavigationSplitView]: https://developer.apple.com/documentation/swiftui/navigationsplitview
[init(sidebar:detail:)]: https://developer.apple.com/documentation/swiftui/navigationsplitview/init(sidebar:detail:)
[init(sidebar:content:detail:)]: https://developer.apple.com/documentation/swiftui/navigationsplitview/init(sidebar:content:detail:)
[navigationDestination(for:destination:)]: https://developer.apple.com/documentation/swiftui/presentedwindowcontent/navigationdestination(for:destination:)
[NavigationPath]: https://developer.apple.com/documentation/swiftui/navigationpath
[migrating-to-new-navigation-types]: https://developer.apple.com/documentation/swiftui/migrating-to-new-navigation-types
[navigationSplitViewColumnWidth(_:)]: https://developer.apple.com/documentation/swiftui/navigationsplitview
[navigationsplitviewvisibility]: https://developer.apple.com/documentation/swiftui/navigationsplitviewvisibility

## Related

[SwiftUI on iPad: Organize your interface](https://developer.apple.com/videos/play/wwdc2022/10058/)    
[SwiftUI on iPad: Add toolbars, titles, and more](https://developer.apple.com/videos/play/wwdc2022/110343/)    
